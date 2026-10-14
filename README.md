# picolab

The lab CLI shared by the **es101** embedded courses (AoES, EOS, …): build, flash, debug,
and monitor Pico (RP2040/RP2350) projects — showing every command it runs.

It is one bash script. No dependencies beyond the tools you already use (Pico SDK, CMake,
Ninja/Make, OpenOCD, picotool, GDB).

## Install

The default install is one shot, no prompts — link `picolab` onto your PATH and provision
the whole toolchain:

```bash
./picolab install --tools --all --yes
```

It symlinks `picolab` into `~/.local/bin`, then runs `doctor --fix --all --yes`, which sets
up everything the labs use: the Pico SDK and its submodules, **picotool** (both for builds
and for probe-free `picolab flash --uf2`; built from source when your distro packages
none), the Raspberry Pi OpenOCD fork (Debug Probe), the FreeRTOS kernel and the udev rule.

Prefer to answer each fix yourself? Run it step by step:

```bash
./picolab install              # symlink into ~/.local/bin (once)
picolab doctor                 # check toolchain + board (green / orange / red)
picolab doctor --fix           # offer each fix, ask before running
picolab doctor --fix --all     # + OpenOCD, FreeRTOS kernel, udev rule (probe or not)
```

The **default flashing path is the Debug Probe (SWD)**. With no probe — a student at home,
just a plain Pico — the same install provisions the probe-free path: `picolab flash --uf2`
over plain USB.

> Arch keeps picotool in the AUR, so `doctor --fix` builds it from source — that works even
> with CMake 4: the configure passes `-DCMAKE_POLICY_VERSION_MINIMUM=3.5`, the compatibility
> the mbedTLS bundled with older SDKs still asks for.

## Daily use

```bash
picolab init my-blink          # create a project, explained step by step
picolab build                  # cmake configure once, then compile + link -> .elf/.uf2
picolab flash                  # SWD via OpenOCD (Debug Probe / PicoGremlin)
picolab flash --uf2            # probe-free: picotool over plain USB
picolab monitor usb            # read USB-CDC output (or: picolab monitor probe)
picolab debug                  # GDB: step, break, inspect (needs a probe)
picolab help                   # all commands;  picolab help env  for settings
```

`picolab flash --uf2` selects the target with `--bus/--address` when the Debug Probe is also
on USB (picotool otherwise refuses with *"requires a single RP-series device"*).

## Settings (environment)

| Variable | Meaning |
|---|---|
| `PICO_SDK_PATH` | Pico SDK checkout (required to build) |
| `FREERTOS_KERNEL_PATH` | FreeRTOS-Kernel checkout (for RTOS labs) |
| `PICO_BOARD` | Board type (default `pico2_w`) |
| `SERIAL` | Force a serial device for `monitor` |
| `ADAPTER_SPEED` | SWD clock kHz (default 2000; lower if flashing fails) |
| `CMSIS_DAP_QUIRK` | 1 = one USB request at a time (default; needed by the course probe) |
| `PICOLAB_LABS` | Colon-separated lab roots (else every `src/<course>/` with Pico projects) |
| `PICOLAB_REPO` | The course repo picolab serves (else the git root of `$PWD`) |
| `PICOLAB_SDK_VERSION` | SDK release `doctor --fix` clones when none exists |
| `PICOLAB_PICOTOOL_VERSION` | picotool release to build when no SDK says which |

## Vendored vs standalone

- **Standalone (this repo):** set `PICOLAB_REPO=/path/to/course` (or `PICOLAB_LABS=…`) so
  `picolab` can find the lab projects.
- **Vendored:** a course repo may carry picolab at `tools/picolab/`; `REPO` then resolves to
  the course root automatically.

## License

MIT — see [LICENSE](LICENSE).
