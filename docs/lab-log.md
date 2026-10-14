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
- **QS tracing:** `displayPhilStat` emits a `PHILO_STAT` QS record with philosopher number and status string. The board's actual trace transport and host QSpy setup have **not yet been verified**.
- **Host debugger issue:** `arm-none-eabi-gdb` cannot launch due to missing `libncursesw.so.5`; defer repair until needed.
- **Next experiment:** Determine QS transport and QSpy host-tool availability, then capture actual `PHILO_STAT` trace output. Avoid firmware edits until the current configuration is understood.

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
