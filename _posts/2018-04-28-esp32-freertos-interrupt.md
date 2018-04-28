---
published: false
---
---
layout: post
title: "ESP32 FreeRTOS - Interrupt"
date:   2018-04-28 15:35:30 +0700
categories: esp32 freertos
---

```c
#include <stdio.h>
#include <time.h>
#include <inttypes.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_system.h"
#include "driver/gpio.h"


unsigned long IRAM_ATTR millis()
{
    return xTaskGetTickCount() * portTICK_PERIOD_MS;
}

xQueueHandle queue;

static void isr_handler(void *arg) {
    unsigned long t = millis();
    portBASE_TYPE pxHigherPriorityTaskWoken = pdFALSE;

    xQueueSendToBackFromISR(queue, &t, &pxHigherPriorityTaskWoken);
}

void printTask() {
    unsigned long t;

    while (1) {
        xQueueReceive(queue, &t, portMAX_DELAY);

        printf("Time: %ld\n", t);
        fflush(stdout);
    }
}

void testTask(void *params) {
    const TickType_t xFreq = 1000 / portTICK_PERIOD_MS;
    TickType_t xLastWakeTime;

    xLastWakeTime = xTaskGetTickCount();

    while (1) {
        printf("Test task is running..\n");
        vTaskDelayUntil(&xLastWakeTime, xFreq);
    }
}

void app_main(void) {
    queue = xQueueCreate(5, sizeof(unsigned long));
    gpio_config_t io_conf;
    //disable interrupt
    io_conf.intr_type = GPIO_INTR_NEGEDGE;
    //set as output mode
    io_conf.mode = GPIO_MODE_INPUT;
    //bit mask of the pins that you want to set,e.g.GPIO18/19
    io_conf.pin_bit_mask = 1ULL << 12;
    //disable pull-down mode
    io_conf.pull_down_en = 0;
    //disable pull-up mode
    io_conf.pull_up_en = 1;
    //configure GPIO with the given settings
    gpio_config(&io_conf);

    gpio_set_intr_type(12, GPIO_INTR_NEGEDGE);

    xTaskCreate(printTask, "Print task", 2048, NULL, 10, NULL);
    xTaskCreate(testTask, "Testing task", 2048, NULL, 1, NULL);

    gpio_install_isr_service(0);
    //hook isr handler for specific gpio pin
    gpio_isr_handler_add(12, isr_handler, NULL);
}

```
