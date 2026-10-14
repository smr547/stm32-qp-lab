# First experiment: official DPP on NUCLEO-L152RE

## Goal

Build, program, and observe Quantum Leaps' **unmodified** [DPP example for NUCLEO-L152RE](https://github.com/QuantumLeaps/qpcpp-examples/tree/main/arm-cm/dpp_nucleo-l152re).

Choose the [QV directory](https://github.com/QuantumLeaps/qpcpp-examples/tree/main/arm-cm/dpp_nucleo-l152re/qv) as the initial cooperative-kernel path. The official example also offers QK and QXK variants; do not inadvertently mix their build settings or ports.

## Before building

- [ ] Inspect board markings; confirm exact NUCLEO-L152RE target.
- [ ] Confirm ST-LINK USB cable, debug connection and target power.
- [ ] Record host operating system, `arm-none-eabi-gcc --version`, `make --version`, flashing/debug utilities, and tool paths.
- [ ] Check upstream [QP/C++ repository](https://github.com/QuantumLeaps/qpcpp) and [examples repository](https://github.com/QuantumLeaps/qpcpp-examples) for version compatibility, submodules/dependencies, build directions, and licences.
- [ ] Pin exact upstream commit SHAs/tags in the [lab log](lab-log.md).
- [ ] Read the board-specific README and QV build files; do **not** guess commands.

## Experiment

1. Obtain an appropriately version-matched copy of upstream QP/C++ and examples without altering their sources.
2. Build the official QV DPP for NUCLEO-L152RE using its documented toolchain.
3. Record the exact build command, compiler version, exit status and generated artifact.
4. Program with ST-LINK, reset, and document observable evidence the application is running (LED, output, tracing, or debugger, depending on the BSP).
5. Retain relevant build output and document the result in [lab-log.md](lab-log.md).

**Definition of done:** an observed working official DPP image on the actual L152RE board, with reproducible commands and pinned upstream versions.

## Afterwards: questions to investigate

- Which QP/C++ ARM Cortex-M port variant and kernel do we actually execute?
- How does the BSP configure SysTick and NVIC priorities?
- Where are critical sections entered/exited and what hardware mechanism protects them?
- Which interrupts can post events to QP active objects?
- What does the idle callback do in the unmodified example?
- What would need to change for low-power Stop mode and a tickless timer?

No power measurements or deep-sleep changes should be mixed into the baseline milestone.
