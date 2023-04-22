# LCFR

Visit the [Assignment page](https://thiagosoutogit.github.io/ProgrammingOracle/Pages/Embedded/Assignment-1.html#takes-from-assignment-1-brief) for the complete documentation on the development of the software.


## Task 1

Task 1 (Monitor frequency and RoC): Measures the instantaneous frequency and the rate of change of frequency. The frequency is measured using an external frequency analyzer, and the rate of change is calculated based on time intervals. The network stability is then determined based on the measured frequency and rate of change.

```c
void task1(void *pvParameters) {
    // Variables for frequency measurement and rate of change calculation
    float prev_freq = 0;
    float curr_freq = 0;
    float prev_time = 0;
    float curr_time = 0;
    float measured_inst_freq;
    float measured_roc_freq;

    while (1) {
        printf("Task 1 - Frequency measurement: \n");
        // Measure inst_freq using appropriate sensors or methods
        measured_inst_freq = IORD(FREQUENCY_ANALYSER_BASE, 0);
        printf("%f Hz\n", 16000 / (double)measured_inst_freq);

        // Calculate rate of change of frequency in Hz/s
        curr_time = xTaskGetTickCount() * portTICK_PERIOD_MS / 1000.0; // Convert ms to s
        curr_freq = measured_inst_freq;
        if (curr_time != prev_time) { // Check if the time interval is not zero
            measured_roc_freq = (curr_freq - prev_freq) / (curr_time - prev_time);
        } else {
            measured_roc_freq = 0; // Set rate of change to 0 if time interval is zero
        }
        prev_freq = curr_freq;
        prev_time = curr_time;

        printf("Task 1 - Rate of frequency change: \n");
        printf("%f Hz\n", (double)measured_roc_freq);

        // Update shared resources
        xSemaphoreTake(xMutex, portMAX_DELAY);
        inst_freq = measured_inst_freq;
        roc_freq = measured_roc_freq;
        net_stability = (inst_freq >= THRESHOLD_FREQ) && (abs(roc_freq) <= THRESHOLD_ROC);
        xSemaphoreGive(xMutex);

        // Sleep for a while (adjust the delay as needed)
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```