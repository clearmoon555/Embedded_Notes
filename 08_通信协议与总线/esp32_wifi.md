esp32的wifi架构

你的应用
   ↓
ESP-IDF Wi-Fi API
   ↓
Wi-Fi Driver
   ↓
无线射频

设置成sta模式，可以连接wifi进行通讯

```
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"
#include "esp_netif.h"

static const char *TAG = "wifi_sta";

/* ========= 1. Wi-Fi / IP 事件处理 ========= */

static void wifi_event_handler(void *arg,
                               esp_event_base_t event_base,
                               int32_t event_id,
                               void *event_data)
{
    if (event_base == WIFI_EVENT) {
        switch (event_id) {

        case WIFI_EVENT_STA_START:
            ESP_LOGI(TAG, "WIFI_EVENT_STA_START");
            esp_wifi_connect();
            break;

        case WIFI_EVENT_STA_DISCONNECTED:
            ESP_LOGI(TAG, "WIFI_EVENT_STA_DISCONNECTED");
            esp_wifi_connect();   // 简单粗暴自动重连
            break;

        default:
            break;
        }
    }

    if (event_base == IP_EVENT &&
        event_id == IP_EVENT_STA_GOT_IP) {

        ip_event_got_ip_t *event = (ip_event_got_ip_t *)event_data;
        ESP_LOGI(TAG, "Got IP: " IPSTR, IP2STR(&event->ip_info.ip));
    }
}

/* ========= 2. Wi-Fi 初始化 ========= */

static void wifi_init_sta(void)
{
    esp_netif_init();
    esp_event_loop_create_default();
    esp_netif_create_default_wifi_sta();

    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    esp_wifi_init(&cfg);

    esp_event_handler_register(WIFI_EVENT,
                               ESP_EVENT_ANY_ID,
                               &wifi_event_handler,
                               NULL);

    esp_event_handler_register(IP_EVENT,
                               IP_EVENT_STA_GOT_IP,
                               &wifi_event_handler,
                               NULL);

    wifi_config_t wifi_config = {
        .sta = {
            .ssid = "YOUR_SSID",
            .password = "YOUR_PASSWORD",
        },
    };

    esp_wifi_set_mode(WIFI_MODE_STA);
    esp_wifi_set_config(WIFI_IF_STA, &wifi_config);
    esp_wifi_start();
}

/* ========= 3. app_main ========= */

void app_main(void)
{
    nvs_flash_init();
    wifi_init_sta();
}

```

发出连接请求返回结果事件并进行对应操作



通过socket使用进行信息交换



mqtt在此基础上进行了扩展和封装



连接上broker，然后订阅同一个topic进行收发