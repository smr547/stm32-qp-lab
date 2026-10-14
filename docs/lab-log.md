# Lab log

Record one reproducible experiment per dated entry.

## 2026-10-14 — Repository initialisation

**Objective:** Document the STM32 + QP/C++ laboratory and the initial DPP milestone.

**Evidence:** Confirmed repository access and verified the official [Quantum Leaps L152RE DPP example directory](https://github.com/QuantumLeaps/qpcpp-examples/tree/main/arm-cm/dpp_nucleo-l152re), which includes QV, QK and QXK variants.

**Result:** Initial project documentation established. No firmware was compiled, flashed, or tested at this stage.

---

## 2026-10-14 — Official DPP/QV successfully built and run

- **Objective:** Build and run the unmodified upstream QP/C++ Dining Philosophers Problem (DPP) QV example on a real NUCLEO-L152RE.
- **Hardware:** ST NUCLEO-L152RE, STM32L152RE Cortex-M3; integrated ST-LINK/V2.1 (USB ID `0483:374b`). Initially a different Nucleo board was connected; this was corrected before the successful run.
- **Host:** Ubuntu 24.04.1 LTS; OpenOCD 0.12.0; Arm GNU toolchain 13.2.Rel1 (GCC 13.2.1).
- **Upstream source:** `~/projects/qpcpp` at `cad6c100`; `~/projects/qpcpp-examples` at `08a097d`; QP `3rd_party` submodule at `827ac80bef352748b495e108178ed3576a8f34bf`.
- **Example directory:** `~/projects/qpcpp-examples/arm-cm/dpp_nucleo-l152re/qv/gnu`.
- **Dependency issue resolved:** Initial build lacked `stm32l1xx.h` and `system_stm32l1xx.c`. Initialised the QP `3rd_party` submodule:

```bash
git -C ~/projects/qpcpp submodule update --init 3rd_party
```

- **Successful build:**

```bash
cd ~/projects/qpcpp-examples/arm-cm/dpp_nucleo-l152re/qv/gnu
make norun QP="$HOME/projects/qpcpp" GNU_ARM="$HOME/arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi"
```

- **Outputs:** `dbg/dpp-qv.elf` and `dbg/dpp-qv.bin`. `arm-none-eabi-size -A dbg/dpp-qv.elf` reported `.isr_vector=292`, `.text=7948`, `.rodata=600`, `.init_array=24`, `.fini_array=4` bytes in flash-related sections; `.stack=2048`, `.data=24`, `.bss=940`, `.heap=4` bytes of allocated RAM sections. These figures are section allocations, not a runtime high-water measurement.

- **Flash and verify:**

```bash
openocd -f interface/stlink.cfg -f target/stm32l1.cfg \
  -c "program dbg/dpp-qv.elf verify reset exit"
```

OpenOCD reported `** Programming Finished **`, `** Verified OK **`, `** Resetting Target **`. SWD detected Cortex-M3 r2p0, six hardware breakpoints and four watchpoints.

- **Observed behaviour:** Green user LED LD2 flashes; red LD3 remains steady. Application BSP controls LD2 on GPIOA PA5; no evidence that the application controls LD3. This is evidence of active execution, not a formal proof of DPP correctness.
- **LED semantics:** `BSP::displayPhilStat(n, stat)` sets LD2 when `stat[0]=='e'` (EATING), clears it for other reported states; it ignores `n` for LED output. Thus LD2 shows the **last reported status**, not whether any philosopher is eating.
- **Event exchange inspected:** `Philo::hungry_e` allocates a `HUNGRY_SIG` `TableEvt` and directly `POST`s it to `AO_Table`. The Table AO tracks fork ownership, publishes `EAT_SIG` when permission is granted, and reports `THINKING`, `HUNGRY`, `EATING` through the BSP. A philosopher in its hungry state accepts `EAT_SIG` only when `TableEvt::philoId == m_id`, then transitions to eating via `qm_tran`.
- **QS tracing:** `displayPhilStat` emits a `PHILO_STAT` QS record with philosopher number and status string. Trace transport and host QSpy were subsequently verified with archived QP/C++ 7.3.2; see the later entry.
- **Host debugger issue:** `arm-none-eabi-gdb` cannot launch due to missing `libncursesw.so.5`; defer repair until needed.
- **Next experiment (completed below):** Determine QS transport and QSpy host-tool availability, then capture actual `PHILO_STAT` trace output.

---

## 2026-10-14 — Archived QP/C++ 7.3.2 QS live tracing (Experiments 33–37)

- **Objective:** Obtain human-readable QS execution records from the official DPP/QV example on NUCLEO-L152RE, without mixing framework versions.
- **Framework/example:** Matching archived QP/C++ 7.3.2 and DPP example at `~/qp_old/qpcpp/examples/arm-cm/dpp_nucleo-l152re/qv/gnu`. Current QP/C++ 8.1.6 public checkout lacked the separate QS extras; request for evaluation access sent to Quantum Leaps.
- **Instrumented build:**

```bash
cd ~/qp_old/qpcpp/examples/arm-cm/dpp_nucleo-l152re/qv/gnu
make CONF=spy GNU_ARM="$HOME/arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi"
openocd -f interface/stlink.cfg -f target/stm32l1.cfg -c "program spy/dpp-qv.elf verify reset exit"
```

- **Observed:** Build/link succeeded (nonfatal syscall/RWX warnings); OpenOCD programmed, verified, reset; green LD2 flashes.
- **USB serial discovery:** `ls -l /dev/serial/by-id/` identified `usb-STMicroelectronics_STM32_STLink_0671FF34314B4E3043233705-if02 -> ../../ttyACM1`. The LG monitor control interface occupied `ttyACM0`. Prefer persistent by-id path for repeatability.
- **Host decoder:** `~/qp_old/qtools/bin/qspy -h` reports QSPY **7.3.1**. Other archived executables: `~/qp_old/qp/qtools/bin/qspy` and `~/qp_old/qtools/qspy/posix/rel/qspy`.
- **Successful live capture:**

```bash
~/qp_old/qtools/bin/qspy -c /dev/ttyACM1 -b 115200 -v 7.3
```

  Live QS records appeared, including time events, event allocation/recycling, AO queue operations, state transitions, and QV scheduling. Pressing the board RESET button while QSpy was connected emitted startup dictionaries.
- **Reset evidence:** `Trg-RST QP-Ver=732,Build=261014_170325`. Dictionaries resolved `TIMEOUT_SIG`, `HUNGRY_SIG`, `EAT_SIG`, `DONE_SIG`; five `Philo::inst[0..4]` AOs; `Table::inst`; philosopher states `thinking/hungry/eating`; Table states `serving/paused`; event pool `EvtPool1`.
- **One concrete sequence (philosopher 4):** `TE0-Post TIMEOUT_SIG` -> `AO-GetL` -> `Philo::thinking -> Philo::hungry` -> `QF-New HUNGRY_SIG` -> `AO-Post` to Table -> `Sch-Next Pri=7->12` -> Table handles request -> `QF-Pub EAT_SIG` to five subscribed philosophers -> `Sch-Next Pri=12->7` -> philosopher 4 consumes EAT_SIG and enters `eating`. Other philosophers consume EAT_SIG without changing state when not eligible.
- **Important distinction:** Table grants eating permission and publishes an event; the philosopher changes its own state upon later event dispatch. `QF-gcA/QF-gc` and `MP-Put` show reference-counted event recycling. Trace establishes live behaviour, not a complete concurrency/correctness proof.
- **Next:** Follow one `TIMEOUT_SIG` from SysTick/time event into philosopher AO through **archived 7.3.2 source**, one source inspection at a time; then inspect QV scheduling implementation.

---

## Experiment template

### YYYY-MM-DD — Short title

- **Objective:**
- **Hardware and wiring:**
- **Software versions / upstream commits:**
- **Commands and output:**
- **Measurements / observations:**
- **Conclusions and uncertainty:**
- **Next experiment:**
