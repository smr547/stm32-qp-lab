# Handover for a new ChatGPT tab / collaborator

**Read this file and the [project README](../README.md) first.**

## User goal and collaboration style

We are exploring ARM Cortex-M and Quantum Leaps QP/C++ in depth, initially as a learning laboratory, and ultimately as a potential foundation for a low-power solar-powered LoRa tipping-bucket rain gauge. The user is an experienced software/electronics engineer, prefers careful incremental experimentation and understanding the actual silicon, and is interested in ARM exception handling, opcodes, QP critical sections, AOs/HSMs and low-power operation. They use Linux, Git, C++, and have previously experimented with ESP32/FreeRTOS/SMP critical sections. Give **one or a few precise steps at a time**, request the actual command output, and do not declare an experiment complete without results.

## What is confirmed

- GitHub project: [smr547/stm32-qp-lab](https://github.com/smr547/stm32-qp-lab).
- Board in hand: **NUCLEO-L152RE** (STM32L152RE / Cortex-M3).
- Other available boards: two NUCLEO-F411RE; one STM32F446 Nucleo-64 (exact variant to verify).
- INA228 current-sensing boards have arrived but have not been unpacked/characterised.
- Quantum Leaps publishes [an official L152RE DPP example](https://github.com/QuantumLeaps/qpcpp-examples/tree/main/arm-cm/dpp_nucleo-l152re), with **QV, QK and QXK** subdirectories.
- Initial decision: **unmodified official DPP, QV variant first**; then examine QP port, critical sections, interrupts, and low-power modes.
- Existing ESP32/SX1276 LoRa node and Signal K base station are functioning references; **do not modify them as part of initial DPP bring-up**.

## What is not yet done

- No L152RE compilation, flash, debugger session, DPP run, or current measurement has been recorded here.
- No toolchain versions or upstream QP/example SHAs have been pinned.
- No production MCU selection; no SX1276 connected to this Nucleo yet.
- No reason to bring FreeRTOS into the first experiment.

## Next assistant action

1. Ask the user to connect the NUCLEO-L152RE and report its status; establish available Linux compiler and ST-LINK tools.
2. Inspect upstream QV example build instructions and match QP/C++ dependencies and releases (don't invent build command or pin versions without checking).
3. Guide a reproducible build, ST-LINK flash and observed DPP run.
4. Record versions, commands and observations in [lab-log.md](lab-log.md) when obtained.

**Please don't delegate coding to Codex before this baseline is established.** Codex may help once we understand the original code/build and have an observed known-good result.

## Documentation conventions

- Use Markdown links formatted as **[descriptive label](https://example.com)** for every GitHub URL in repository docs.
- Maintain clear distinction between *observed*, *documented upstream*, *hypothesis*, and *planned*.
- Keep upstream files/licences and origins identifiable if vendor/example code is added.
- Where possible, update the actual GitHub repository through the connected GitHub integration; avoid claiming an edit happened until it is verified.
