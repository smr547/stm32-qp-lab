# Architecture notes and open questions

## Initial architecture

The immediate learning target is **NUCLEO-L152RE + QP/C++ + QV + official DPP**.

Longer-term application direction:

```text
Rain gauge AOs / HSMs
       |
Events and hardware-independent service interfaces
       |
STM32 + QP/C++ QV
       +-- GPIO / EXTI: tipping-bucket input
       +-- SPI / DIO / RESET: SX1276 radio module
       +-- RTC / timers: scheduling and wake-up
       +-- nonvolatile storage: persistent counts and reset epochs
       +-- power-management service: sleep / wake sequencing
       +-- optional UART/SPI: external Wi-Fi network coprocessor
```

The existing ESP32 LilyGO/LoRa node is a working comparison baseline; no change to its firmware or protocol is implied by this lab.

## Portability boundary

QP/C++ AO/HSM application logic should be reusable, **subject to** device-independent events and service contracts. Board-support code, startup/linker scripts, interrupt vectors/priorities, peripheral drivers, persistence, scheduling, and low-power transitions are not automatically portable. An official port/example is not equivalent to certification.

## Low-power research

- **Sleep:** inspect core idle entry and interrupt wake-up; verify QV idle handling.
- **Stop:** test SRAM retention, clocks and peripheral reinitialisation, interrupt/wake sources, SysTick implications, and debugger effects.
- **Standby:** test reset-like wake-up, loss/retention of state, backup-domain storage, and persistence across reset.
- **Event integrity:** ensure closely spaced tipping-bucket events are accepted or counted reliably, including events during wake/recovery.
- **Energy:** measure both quiescent power and wake/process/transmit/sleep energy. Account for shunt resistance, instrument bandwidth, regulator and radio consumption.

Do **not** assume the production design will use the STM32L152RE or that its Nucleo current measurements equal standalone target-PCB current.

## Radio design research

The SX1276 LoRa radio uses SPI (SCK/MOSI/MISO/NSS), RESET, and DIO interrupts. Confirm the exact **module** pinout and radio voltage, not just the LilyGO board's GPIO mapping. Begin with a register read, then TX/RX with a known-working base. Prefer interrupt-driven operation and explicit radio sleep control. Retain compatibility with the existing packet format.

## Wi-Fi research (later)

If needed for future instruments, evaluate a separately managed Wi-Fi network coprocessor with UART/SPI and a QP Wi-Fi service AO. Decide where TCP/IP/TLS actually runs. **Wi-Fi is not required by the LoRa rain-gauge node.**

## Open decisions

1. QP version and upstream example commit; how to pin them reproducibly.
2. Compiler/build/debug environment on the Linux workstation.
3. Preferred QP kernel after proving QV baseline.
4. Specific MCU for production and required sleep-mode retention.
5. Robust tipped-switch event counting and recovery design.
6. SX1276 module selection and STM32 driver ownership.
7. INA228 board shunt values, calibration, dynamic range, and suitable measurement wiring.
