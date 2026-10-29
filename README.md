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
up everything the labs use: the Pico SDK and its submodules, the Raspberry Pi OpenOCD fork
(Debug Probe), the FreeRTOS kernel, the udev rule and the PicoGremlin logic-analyzer tools.
**picotool is only installed when no debug probe is attached** — it is what probe-free
`picolab flash --direct` drives, while on a probe bench SWD flashes and every build fetches
its own copy (setup §5).

Prefer to answer each fix yourself? Run it step by step:

```bash
./picolab install              # symlink into ~/.local/bin (once)
picolab doctor                 # check toolchain + board (green / orange / red)
picolab doctor --fix           # offer each fix, ask before running
picolab doctor --fix --all     # + OpenOCD, FreeRTOS kernel, udev rule (probe or not)
```

The **default flashing path is the Debug Probe (SWD)**. With no probe — a student at home,
just a plain Pico — the same install provisions the probe-free path: `picolab flash --direct`
over plain USB.

> Arch keeps picotool in the AUR, so `doctor --fix` builds it from source — that works even
> with CMake 4: the configure passes `-DCMAKE_POLICY_VERSION_MINIMUM=3.5`, the compatibility
> the mbedTLS bundled with older SDKs still asks for.

On Ubuntu, `doctor --fix` installs the Pico udev rule before the system's seat-access rules,
then adds the current user to `plugdev` for persistent access (including SSH and after reboot).
In a graphical session it can request administrator approval with `pkexec`; in SSH, an
administrator must approve the `sudo usermod -aG plugdev <user>` step. A normal user cannot
grant themselves USB access without administrator approval.

## Daily use

```bash
picolab init my-blink          # create a project, explained step by step
picolab build                  # cmake configure once, then compile + link -> .elf/.uf2
picolab flash                  # SWD via OpenOCD (Debug Probe / PicoGremlin)
picolab flash --direct         # no probe: upload straight over USB with picotool (--uf2: alias)
picolab monitor usb            # read USB-CDC output (or: picolab monitor probe)
picolab debug                  # GDB: step, break, inspect (needs a probe)
picolab la gui                 # PicoGremlin logic analyzer: Qt oscilloscope (or: la info/cap/stream)
picolab help                   # all commands;  picolab help env  for settings
```

`picolab flash --direct` selects the target with `--bus/--address` when the Debug Probe is also
on USB, and stops before picotool when no target is on USB — the probe answers picotool too
(runtime reset), so picolab pins the target itself and never flashes the debugger.

## Logic analyzer

The PicoGremlin's second function (the probe's `if03` CDC port). The tools live in their own
project — [picogremlin-la](https://github.com/gsebik/picogremlin-la) (GPL-3.0, its own
installer); `picolab la` just runs them on the probe it already resolved for flashing:

```bash
picolab la info                # probe identity + capability lines
picolab la cap -c 14,15 -t 14  # one-shot capture (GP16/GP17 = ch14/15) -> CSV/VCD; --plot
picolab la stream -c 0-7       # continuous stream for --seconds (rate, edges)
picolab la gui                 # Qt oscilloscope GUI
```

`picolab doctor --fix` installs the CLI + GUI when a PicoGremlin is attached, and
`install --tools --all --yes` provisions them probe or not (point `PICOGREMLIN_LA_SRC` at a
checkout to use its `install.sh` instead of the published one).

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
| `PICOLAB_LA` | Path to the `la` CLI (logic analyzer) if not on PATH / in the shared venv |
| `PICOGREMLIN_LA_SRC` | picogremlin-la checkout whose `install.sh` `doctor --fix` runs |

## Vendored vs standalone

- **Standalone (this repo):** set `PICOLAB_REPO=/path/to/course` (or `PICOLAB_LABS=…`) so
  `picolab` can find the lab projects.
- **Vendored:** a course repo may carry picolab at `tools/picolab/`; `REPO` then resolves to
  the course root automatically.

## License

MIT — see [LICENSE](LICENSE).
