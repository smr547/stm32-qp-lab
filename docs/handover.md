# Handover for a new ChatGPT tab / collaborator

**Read this file and the [project README](../README.md) first.** See [lab-log.md](lab-log.md) for commands and measurements.

## User goal and collaboration style

Learn ARM Cortex-M and Quantum Leaps QP/C++ deeply on the NUCLEO-L152RE, potentially progressing to a solar-powered LoRa rain gauge. User is an experienced software/electronics engineer who wants **one experiment at a time**, with actual output inspected before moving on. Interested in event queues, state machines, ARM exceptions, critical sections, memory ordering and low power.

## Verified baseline — 2026-10-14

- **Official unmodified DPP/QV example successfully compiled, programmed, verified and observed executing** on NUCLEO-L152RE (STM32L152RE, Cortex-M3). LD2 flashes green, LD3 steady red.
- Upstream `qpcpp` commit `cad6c100`; `qpcpp-examples` `08a097d`; QP `3rd_party` submodule `827ac80bef352748b495e108178ed3576a8f34bf`.
- Ubuntu 24.04.1; OpenOCD 0.12.0; ARM GNU toolchain 13.2.Rel1 / GCC 13.2.1. ST-LINK/V2.1 `0483:374b`.
- Directory `~/projects/qpcpp-examples/arm-cm/dpp_nucleo-l152re/qv/gnu`.
- Dependency: `git -C ~/projects/qpcpp submodule update --init 3rd_party`.
- Build: `make norun QP="$HOME/projects/qpcpp" GNU_ARM="$HOME/arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi"`.
- Flash: `openocd -f interface/stlink.cfg -f target/stm32l1.cfg -c "program dbg/dpp-qv.elf verify reset exit"`.
- Debugger `arm-none-eabi-gdb` currently blocked by missing `libncursesw.so.5`; not required for flash.
- Flash section allocation roughly 8.9 KB; reserved stack 2 KB, plus small data/BSS; see lab log for section-by-section figures.
- LED is set by `BSP::displayPhilStat`: on for EATING, off otherwise; philosopher ID is ignored for LED, but included in QS trace.
- Source tracing: `Philo::hungry_e` posts `HUNGRY_SIG` directly to Table; Table arbitrates forks and publishes `EAT_SIG`; philosopher only enters eating on matching `philoId`.
- QS `PHILO_STAT` trace records exist in BSP source; **no QSpy trace captured yet**.

## Next immediate goal: QSpy before QV scheduler internals

1. Inspect the *existing* BSP QS setup and transport (`bsp.cpp` and `gnu/Makefile`) and host availability of `qspy` / QTools.
2. Confirm physical serial endpoint or other QS transport and settings; do not guess baud rate, USB interface, or whether the current build enables QS.
3. Capture and decode a real `PHILO_STAT` trace record if supported; document exact commands, connections and results.
4. Only after QSpy bring-up, trace QV scheduling, AO event queues, interrupts and ARM details.

## Still outstanding

- No verified QSpy transport/capture; no debugger session; no current measurements; no LoRa attached; no production MCU selected.
- Other boards: two NUCLEO-F411RE and one STM32F446 Nucleo-64 (exact variant to verify); INA228 boards not yet characterised.
- Preserve working ESP32/LoRa rain-gauge system and unmodified official STM32 example.

## Documentation conventions

- Use [descriptive GitHub links](https://github.com/smr547/stm32-qp-lab), distinguish observation from inference, preserve upstream revisions.
- Prefer direct repository edits when authorised, verify resulting commits, and keep experiments small.
