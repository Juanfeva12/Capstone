# Space Ground Command Response System — Real-Time Systems Final Capstone

## One sentence

A dual-core spacecraft ground-command response system that compares FreeRTOS direct task notifications with binary semaphores, measures interrupt and task wake latency under controlled load, and demonstrates graceful degradation through a compile-time fault mode.

## Links

* **GitHub Pages:** https://github.com/Juanfeva12/Capstone/
* **Live Wokwi:** https://wokwi.com/projects/468120880275908609
* **Demo video:** 



## Project overview

A button connected to GPIO 18 represents a ground command. The `IRAM\_ATTR` ISR timestamps the event, toggles GPIO 19 for logic-analyzer measurement, gives a binary semaphore, sends a direct task notification, and exits. Two Core-1 bottom-half tasks measure the observed wake latency. The direct-notification task is the real command-handling path; the semaphore task remains as a comparison path.

When `WITH\_LOAD=1`, four periodic tasks from App 2 run on Core 1 at periods of 10, 20, 50, and 100 ms. They create controlled scheduling contention and record their own WCET maxima. A low-priority Core-0 metrics task prints WCET and heartbeat evidence every five seconds.

## Architecture

!\[System architecture](docs/architecture.svg)

* GPIO 18 produces the event.
* The ISR is the short top half.
* Direct notification and a binary semaphore wake separate bottom-half tasks.
* App 2 periodic tasks provide deterministic Core-1 load.
* Core 0 prints WCET and heartbeat evidence.

## Build modes

Normal loaded build:

```c
#define WITH\_LOAD 1
#define FAULT\_MODE 0
```

Idle baseline:

```c
#define WITH\_LOAD 0
#define FAULT\_MODE 0
```

Loaded fault injection:

```c
#define WITH\_LOAD 1
#define FAULT\_MODE 1
```

Fault mode still delivers both signals but intentionally skips `portYIELD\_FROM\_ISR()`. The system remains functional, while immediate rescheduling is no longer explicitly requested.

## Recorded timing evidence

|Configuration|Presses|Notification max|Semaphore max|
|-|-:|-:|-:|
|Idle normal|50|1983 µs|2383 µs|
|Loaded normal|57|2198 µs|2488 µs|
|Loaded fault|48|2204 µs|2367 µs|

The logic-analyzer measurement from the GPIO 18 falling edge to the GPIO 19 ISR pulse was approximately **15.35 µs**.

These values are complete observed ISR-to-task latencies. They include scheduling, the other priority-12 responder, logging, and Wokwi simulation overhead. They are not claimed to be the isolated internal execution cost of each FreeRTOS API.

Full timing documentation: [docs/task-table.md](docs/task-table.md)

## Engineering analysis

### Why is the ISR kept short?

Long processing inside an ISR would block other interrupts and make worst-case latency harder to bound. The ISR only timestamps the event, toggles the timing pin, performs ISR-safe signaling, and exits.

### Why use direct task notification for the real command path?

The relationship is one ISR to one specific responder task. A direct notification uses the task's built-in notification state and normally requires less kernel-object overhead than a separate semaphore.

### Why did load increase the observed maximum?

Load Task A has priority 15, while both bottom-half tasks have priority 12. Task A can therefore delay a newly awakened responder. The loaded normal notification maximum increased by 215 µs compared with the idle normal run.

### Why do the two paths alternate between low and high values?

Both responders have the same priority and are awakened by the same ISR. One runs first, while the other can wait behind its peer and serial logging. The experiment measures the complete response path, not only primitive cost.

### What did fault injection show?

Removing the immediate ISR yield did not lose accepted commands and did not crash the system. In Wokwi, the maximum values remained similar to loaded normal operation. The supported conclusion is graceful continued operation without an explicit immediate context-switch request, not a guaranteed large latency increase.

### Why are the periodic tasks deterministic?

Their buffers are initialized once, operations are fixed, and volatile sinks prevent compiler removal. That provides a repeatable workload for comparing idle and loaded response.

## Hazard analysis

See [docs/hazard-analysis.md](docs/hazard-analysis.md).

## Graceful degradation

With `FAULT\_MODE=1`, ISR-safe signaling remains active but the explicit ISR-exit yield is removed. Startup clearly reports degraded mode. Commands remain functional, and the latency counters continue recording the resulting timing behavior.

## Repository structure

```text
├── docs/
│   ├── index.html
│   ├── architecture.svg
│   ├── task-table.md
│   ├── hazard-analysis.md
│   ├── reflection.md
│   └── assets/
├── firmware/
│   ├── main.c
│   ├── diagram.json
│   ├── wokwi-project.txt
│   └── logic-analyzer-capture.vcd
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Build and run

1. Import the `firmware` files into an ESP-IDF ESP32-S3 Wokwi project.
2. Connect the button between GPIO 18 and GND.
3. Connect logic-analyzer D0 to GPIO 18, D1 to GPIO 19, and analyzer GND to ESP32 GND.
4. Select the required `WITH\_LOAD` and `FAULT\_MODE` values.
5. Run the simulation and press the button.
6. Read latency lines and the five-second WCET summaries in the serial monitor.

## Portfolio focus

This project is tailored toward an **embedded firmware / real-time systems role**. It demonstrates ISR design, top-half/bottom-half separation, ISR-safe FreeRTOS APIs, task priorities, multicore pinning, deterministic load, WCET instrumentation, fault injection, and evidence-based engineering conclusions.

## Final reflection

See [docs/reflection.md](docs/reflection.md).

## AI assistance disclosure

ChatGPT was used to help organize the final repository, review the capstone documentation, and make a minimal addition that prints existing WCET counters. The original App 3 interrupt, notification, semaphore, load-task, and measurement design was preserved. All reported timing values came from the user's Wokwi runs or existing App 3 evidence; unrecorded WCET values were not invented.

## License

MIT License. See [LICENSE](LICENSE).

