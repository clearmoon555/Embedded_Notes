```c

#include "freertos/FreeRTOS.h"
#include "freertos/queue.h"
#include "esp_bt.h"
#include "esp_bt_main.h"
#include "esp_gap_ble_api.h"
#include "esp_gatts_api.h"
#include "esp_bt_defs.h"
#include "esp_log.h"
#include <string.h>
#include <stdint.h>
#include <stdbool.h>

#include "ble_gatts.h"

#define TAG "BLE_GATTS"

/* ================= UUID 定义 ================= */
static const uint8_t SERVICE_UUID[16] = {
    0x12,0x34,0x56,0x78,0x90,0xab,0xcd,0xef,
    0xfe,0xdc,0xba,0x09,0x87,0x65,0x43,0x21
};

static const uint8_t CHAR_UUID[16] = {
    0x21,0x43,0x65,0x87,0x09,0xba,0xdc,0xfe,
    0xef,0xcd,0xab,0x90,0x78,0x56,0x34,0x12
};

/* ================= BLE 句柄 ================= */
static uint16_t service_handle = 0;
static uint16_t char_handle = 0;
static uint16_t conn_id = 0;
static bool notify_enabled = false;
static esp_gatt_if_t g_gatts_if = 0;

static QueueHandle_t s_cmd_queue = NULL;

/* ================= GAP 回调 ================= */
static void gap_cb(esp_gap_ble_cb_event_t event, esp_ble_gap_cb_param_t *param)
{
    switch(event) {
        case ESP_GAP_BLE_ADV_DATA_SET_COMPLETE_EVT:
            {
                static esp_ble_adv_params_t adv_params = {
                    .adv_int_min        = 0x20,
                    .adv_int_max        = 0x40,
                    .adv_type           = ADV_TYPE_IND,
                    .own_addr_type      = BLE_ADDR_TYPE_PUBLIC,
                    .channel_map        = ADV_CHNL_ALL,
                    .adv_filter_policy  = ADV_FILTER_ALLOW_SCAN_ANY_CON_ANY,
                };
                esp_ble_gap_start_advertising(&adv_params);
            }
            break;
        case ESP_GAP_BLE_ADV_START_COMPLETE_EVT:
            if(param->adv_start_cmpl.status == ESP_BT_STATUS_SUCCESS)
                ESP_LOGI(TAG, "Advertising started");
            else
                ESP_LOGE(TAG, "Advertising failed");
            break;
        default:
            break;
    }
}


/* ================= GATTS 回调 ================= */
static void gatts_cb(esp_gatts_cb_event_t event, esp_gatt_if_t gatts_if,
                     esp_ble_gatts_cb_param_t *param)
{
    switch (event) {
    case ESP_GATTS_REG_EVT:
        g_gatts_if = gatts_if;

        esp_ble_gap_set_device_name("ESP32_LED");

        esp_ble_adv_data_t adv_data = {
            .set_scan_rsp = false,
            .include_name = true,
            .include_txpower = false,
            .service_uuid_len = ESP_UUID_LEN_128,
            .p_service_uuid = (uint8_t *)SERVICE_UUID,
            .flag = ESP_BLE_ADV_FLAG_GEN_DISC | ESP_BLE_ADV_FLAG_BREDR_NOT_SPT,
        };
        esp_ble_gap_config_adv_data(&adv_data);

        /* 创建服务 */
        {
            esp_gatt_srvc_id_t service_id = {
                .is_primary = true,
                .id.inst_id = 0,
                .id.uuid.len = ESP_UUID_LEN_128,
            };
            memcpy(service_id.id.uuid.uuid.uuid128, SERVICE_UUID, 16);
            esp_ble_gatts_create_service(gatts_if, &service_id, 4);
        }
        break;

    case ESP_GATTS_CREATE_EVT:
        service_handle = param->create.service_handle;
        esp_ble_gatts_start_service(service_handle);

        /* 添加特征 */
        {
            esp_bt_uuid_t char_uuid = { .len = ESP_UUID_LEN_128 };
            memcpy(char_uuid.uuid.uuid128, CHAR_UUID, 16);

            esp_ble_gatts_add_char(service_handle, &char_uuid,
                ESP_GATT_PERM_WRITE,
                ESP_GATT_CHAR_PROP_BIT_WRITE_NR | ESP_GATT_CHAR_PROP_BIT_NOTIFY,
                NULL, NULL);
        }
        break;

    case ESP_GATTS_ADD_CHAR_EVT:
        char_handle = param->add_char.attr_handle;
        break;

    case ESP_GATTS_CONNECT_EVT:
        conn_id = param->connect.conn_id;
        ESP_LOGI(TAG, "Client connected, conn_id=%d", conn_id);
        break;

    case ESP_GATTS_DISCONNECT_EVT:
        notify_enabled = false;
        ESP_LOGI(TAG, "Client disconnected, restarting advertising");
        esp_ble_gap_start_advertising(&(esp_ble_adv_params_t){
            .adv_int_min = 0x20,
            .adv_int_max = 0x40,
            .adv_type = ADV_TYPE_IND,
            .own_addr_type = BLE_ADDR_TYPE_PUBLIC,
            .channel_map = ADV_CHNL_ALL,
            .adv_filter_policy = ADV_FILTER_ALLOW_SCAN_ANY_CON_ANY,
        });
        break;

    case ESP_GATTS_WRITE_EVT:
        if (!param->write.is_prep && param->write.handle == char_handle && param->write.len == 1) {
            ble_led_cmd_t cmd = { .value = param->write.value[0] };
            if (xQueueSend(s_cmd_queue, &cmd, 0) != pdPASS) {
                ESP_LOGW(TAG, "Command queue full, LED command dropped");
            }
        }

        /* 检测客户端订阅通知（CCC写） */
        if (param->write.handle == char_handle + 1 && param->write.len == 2) {
            notify_enabled = (param->write.value[0] & 0x01) ? true : false;
            ESP_LOGI(TAG, "Notify %s", notify_enabled ? "enabled" : "disabled");
        }
        break;

    default:
        break;
    }
}

/* ================= 对外接口 ================= */
void ble_gatts_init(QueueHandle_t cmd_queue)
{
    s_cmd_queue = cmd_queue;

    // 关闭传统蓝牙
    esp_bt_controller_mem_release(ESP_BT_MODE_CLASSIC_BT);

    esp_bt_controller_config_t bt_cfg = BT_CONTROLLER_INIT_CONFIG_DEFAULT();

    // 初始化硬件，打开蓝牙射频
    if (esp_bt_controller_init(&bt_cfg) != ESP_OK) {
        ESP_LOGE(TAG, "BT controller init failed");
        return;
    }
    if (esp_bt_controller_enable(ESP_BT_MODE_BLE) != ESP_OK) {
        ESP_LOGE(TAG, "BT controller enable failed");
        return;
    }

    // 初始化蓝牙协议栈，打开蓝牙协议栈
    if (esp_bluedroid_init() != ESP_OK) {
        ESP_LOGE(TAG, "Bluedroid init failed");
        return;
    }
    if (esp_bluedroid_enable() != ESP_OK) {
        ESP_LOGE(TAG, "Bluedroid enable failed");
        return;
    }

    // 注册gap，gatts回调函数
    esp_ble_gap_register_callback(gap_cb);
    esp_ble_gatts_register_callback(gatts_cb);


    esp_ble_gatts_app_register(0);
}

void ble_gatts_notify_led_state(uint8_t state)
{
    if (!notify_enabled || char_handle == 0) return;

    esp_ble_gatts_send_indicate(
        g_gatts_if,
        conn_id,
        char_handle,
        sizeof(state),
        &state,
        false
    );
}

```

流程

1. 关闭传统蓝牙
2. 初始化硬件，打开蓝牙射频
3. 初始化蓝牙协议栈，打开蓝牙协议栈
4. 注册gap，gatts回调函数
5. 发送创建蓝牙服务请求
6. 收到ESP_GATTS_REG_EVT，配置广播数据，创建服务
7. 收到ESP_GAP_BLE_ADV_DATA_SET_COMPLETE_EVT，设置广播参数，开始广播
8. 收到ESP_GAP_BLE_ADV_START_COMPLETE_EVT，广播成功
9. 收到ESP_GATTS_CREATE_EVT，服务创建成功，创建特征