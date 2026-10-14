# STM32 QP Lab

A practical laboratory for learning **ARM Cortex-M**, **Quantum Leaps QP/C++**, active objects (AOs), hierarchical state machines (HSMs), interrupts, and low-power embedded design.

**Initial reference board:** ST **NUCLEO-L152RE** (STM32L152RE, Cortex-M3).  
**First milestone:** build, flash, and understand Quantum Leaps' official **Dining Philosophers Problem (DPP)** example **without modifying its application code**.

**Baseline achieved (2026-10-14):** the unmodified official DPP/QV example was built, programmed, verified and observed running on NUCLEO-L152RE. Next: capture QS/QSpy trace output.

## Why this project?

We are assessing whether a low-power STM32 plus an SPI-connected SX1276 LoRa module can become the foundation of a solar-powered tipping-bucket rain-gauge node. The STM32's well-documented ARM architecture and available QP/C++ Cortex-M ports make it an attractive alternative to developing our application framework on ESP32/SMP FreeRTOS. The larger ambition is a reusable QP/C++ STM32 platform with interchangeable communication services (LoRa first; possibly an external Wi-Fi coprocessor later).

**Immediate focus: learn the official QP example and ARM port before touching the rain-gauge application.**

## Hardware in the workshop

| Board | Quantity | MCU / architecture | Initial role |
| --- | ---: | --- | --- |
| NUCLEO-L152RE | 1 | STM32L152RE / Cortex-M3 | **Primary DPP and low-power experiments** |
| NUCLEO-F411RE | 2 | STM32F411RE / Cortex-M4F | Later porting / comparison |
| STM32F446 Nucleo-64 | 1 | STM32F446 family / Cortex-M4F | Later porting / comparison; verify exact board designation |

INA228 current/power measurement boards have arrived; their shunt values and configuration have **not yet been inspected**.

## Upstream references

- [Quantum Leaps QP/C++](https://github.com/QuantumLeaps/qpcpp)
- [Official QP/C++ example collection](https://github.com/QuantumLeaps/qpcpp-examples)
- [Official `dpp_nucleo-l152re` example](https://github.com/QuantumLeaps/qpcpp-examples/tree/main/arm-cm/dpp_nucleo-l152re) — includes `qv`, `qk`, and `qxk` variants.
- [Quantum Leaps documentation](https://www.state-machine.com/)
- [ST NUCLEO-L152RE board information](https://www.st.com/en/evaluation-tools/nucleo-l152re.html)

The presence of an official example establishes a useful board-specific starting point; it is **not** evidence of a formal safety certification. Verify current upstream build instructions, versions, dependencies, and licensing before copying files.

## Lab plan

1. **Baseline:** inspect the official DPP example, choose **QV** initially, record exact upstream revisions and available compiler/debug tools.
2. **Build and flash:** run *unmodified* official DPP on NUCLEO-L152RE with ST-LINK. Record serial/LED/QS observations as applicable. Do not mark complete until observed on real hardware.
3. **Understand the architecture:** trace QP startup, AO queues, event dispatch, SysTick, ISR posting, critical sections, NVIC priorities, and Cortex-M exception entry/return.
4. **Measure power:** inspect the Nucleo power schematic and IDD measurement provisions before connecting instrumentation. Characterise run/idle/Sleep, then Stop/Standby while verifying clock recovery and SRAM/state retention.
5. **Wake on a tip:** create a repeatable GPIO wake test and measure latency and current, including closely spaced input events.
6. **LoRa:** drive an SX1276 module over SPI, preserve existing radio packet compatibility, and explore interrupt-driven radio operation in a LoRa AO.
7. **Broaden the platform:** examine other STM32 boards, portable drivers, and possibly a UART/SPI Wi-Fi coprocessor.

The **production MCU is not selected**. NUCLEO-L152RE is an experimental platform, not a production-board commitment.

## Lab documents

- [Getting started with DPP](docs/dpp-first-run.md) — first-session checklist and evidence to capture.
- [Architecture and research questions](docs/architecture.md) — QP, STM32, sleep modes, and radio interfaces.
- [Handover to the next conversation](docs/handover.md) — project context, completed work, and the very next action.
- [Lab log](docs/lab-log.md) — dated observations, commands, results, and failures.

## Working principles

- Prefer **one reproducible experiment at a time** with measured observations rather than assumptions.
- Keep original upstream code identifiable and record the upstream commit/tag before modifications; comply with its license.
- Prefer an **initial QV-only** bring-up. Other QP kernels and FreeRTOS integration can be evaluated separately.
- Avoid conflating *MCU* current with *whole Nucleo board* current, or peak current with integrated energy.
- Preserve the known-working ESP32/LoRa rain-gauge baseline while investigating STM32.
- Keep GitHub links in Markdown documents as clickable **[descriptive text](URL)** links.

## Status

**DPP/QV built and flashed successfully; LD2 observed flashing green.** See [lab log](docs/lab-log.md) for exact versions, commands and observed behaviour. **QSpy trace, debugger operation, power measurements and LoRa hardware remain unverified.**
