# BETAFPV 20A AIO / AT32F435 port

## Scope and provenance

This independent branch is based on Rotorflight 4.6.0 commit `118e9120260bb33f46df4f92052fb0e9fd4e9ebc`.
It retains the Rotorflight flight-control/configuration layer and adds the AT32 hardware port. The target describes the board hardware. Motor pole counts, tail type, servo allocation and aircraft tuning are user settings and are not imposed by this firmware.

Hardware used: BETAFPV F4 2–3S 20A AIO FC V1, AT32F435RGT7 (1MiB flash, default 288MHz), ICM42605, onboard UART ELRS, 16MiB SPI flash and Bluejay ESCs.
The original Betaflight target name `BETAFPVF435` is retained. `AT32` is the MCU family; `F435` identifies the model. Do not select this firmware for the STM32F405 board variant.

Porting references:

- [Rotorflight firmware](https://github.com/rotorflight/rotorflight-firmware), base commit above.
- [flightng/atbetaflight](https://github.com/flightng/atbetaflight), `4.3-bsp-rebase`, commit `6bf2667698b51adae90ef917005fbc5059dd4504`.
- [Betaflight](https://github.com/betaflight/betaflight), 4.5.2 commit `024f8e13d4e642eb6a380308685b9ea3aa3ef1a2`.
- Artery AT32F435/437 BSP v2.1.0, shipped with its original notices under `lib/main/AT32F43x`.

Original project licenses and vendor copyright notices remain in place. See [LICENSE](../LICENSE).

## Board connections and neutral defaults

| Board function | MCU / timer |
|---|---|
| Onboard ESC1 / MOTOR1 | PB0 / TMR3 CH3 |
| Onboard ESC2 / MOTOR2 | PB1 / TMR3 CH4 |
| Onboard ESC3 / MOTOR3 | PA3 / TMR2 CH4 |
| Onboard ESC4 / MOTOR4 | PA2 / TMR2 CH3 |
| T1 / UART1 TX | PA9 / TMR1 CH2 when remapped to PWM |
| R1 / UART1 RX | PA10 / TMR1 CH3 when remapped to PWM |
| T6 / UART6 TX | PC6 / TMR8 CH1 (AF3) when remapped to PWM |
| Onboard ELRS / UART3 | PB10 TX / PB11 RX |
| Battery voltage / total current | PC2 / PC1 |

Default motor numbering follows the physical ESC labels. There are no assigned servo outputs, forced motor pole counts or forced motorized-tail mode. Normal Rotorflight aircraft defaults remain in effect; the target selects DShot300 for its onboard Bluejay ESC hardware. Bidirectional RPM is enabled by the user when configuring the aircraft. UART1 and UART6 TX are available until their pins are explicitly remapped.

Configure motor/servo resources, tail type, pole counts, gearing and tuning for the actual helicopter. Remapping a UART pin to PWM requires releasing the corresponding serial resource. No personal aircraft settings are bundled.

Native bidirectional DShot on a channel4 output uses channel3 of the same timer for indirect RPM capture: ESC2 therefore reserves TMR3 CH3 (ESC1), and ESC3 reserves TMR2 CH3 (ESC4). Those pairs cannot be used concurrently with this native capture method. Choose a non-conflicting assignment or a compatible bitbang/timer allocation; resource checks are retained. This is a timer/backend constraint, independent of aircraft model. Native bitbang/burst defaults remain OFF and can be configured through normal Rotorflight settings.

## Board-default regression test

```sh
python src/test/test_betafpvf435_defaults.py --compiler clang
```

This host test uses the production timer table, resource lookup and motor/servo reset functions. It checks physical ESC numbering, absence of preset servo channels, preservation of motor pole counts/RPM choice/tail settings, and normal UART pin defaults. A local negative-control run also verified that the earlier aircraft-specific override fails this test.

## Build

Validated toolchain: GNU Make 4.4.1, Arm GNU GCC 9.3.1 (`gcc-arm-none-eabi-9-2020-q2-update`), Git for Windows shell. All required AT32 source files are included in the repository.

With GNU Make and the compiler installed, run from the repository root:

```sh
make TARGET=BETAFPVF435 ARM_SDK_DIR=/path/to/gcc-arm-none-eabi-9-2020-q2-update -j4 hex
```

On Windows, the validated build explicitly uses Git's shell:

```powershell
& 'C:/path/to/make.exe' TARGET=BETAFPVF435 ARM_SDK_DIR='C:/path/to/gcc-arm-none-eabi-9-2020-q2-update' 'SHELL=C:/Progra~1/Git/usr/bin/sh.exe' -j4 hex
```

Output: `obj/rotorflight_4.6.0_BETAFPVF435.hex`.
The local development build wrapper and workstation tool installation are not required to build this source tree. Other host platforms have not been validated in this project.

Release assets include a SHA256 file. Rebuilds can differ due to build time and Git revision metadata. The initial port.1 bench binary contained aircraft-specific defaults; use the corrected board-generic release rather than treating that legacy asset as neutral board defaults.

## Validation record

The board-generic revision removes the earlier aircraft-specific default overrides and restores the physical board connector map. It has its own build and host-default checks; older bench results below apply to the earlier driver checkpoint, not a physical retest of the new default mapping.
Historical local validation included:

- Maintained upstream host tests: 23 suites / 167 tests, 0 failures and 0 build-unavailable.
- Host regression checks of peripheral, timer/DMA, USB control, serial, storage and protocol error paths.
- Deliberate-fault sensitivity checks: 17/17 compiled mutations detected.
- AT32 and STM32F405 builds, plus a fresh official-base checkout with the complete port patch reapplied.
- 425 production C translation units analyzed; 320 raw diagnostics retained and reviewed, 0 unresolved. Raw diagnostics include expected MMIO/model limitations; this is not a claim of a warning-free analyzer run or proof that all defects are absent.
- After programming: 567,852 HEX data bytes independently read back and matched; reboot/MSP, ICM42605 and 16MiB flash identified.
- Earlier port checkpoints included physical servo movement and short blade-free motor/RPM checks. These observations are distinguished from validation of every optional peripheral or real-time load case.

The host harness and raw workstation reports remain in the local development records; this first publication includes the upstream test-source changes and a summary, not a packaged portable runner for all workstation checks. The inherited upstream GitHub workflows do not constitute an AT32 CI result.

Firmware was developed for the tested board/wiring. Other sensor revisions, peripheral combinations, electrical timing/DMA contention and aircraft flight tuning require their own validation. The published asset is a pre-release independent port.

## Configurator USB selection

Some Rotorflight Configurator 2.3.0 builds recognize desktop USB ports only by STM/CP210 display names, excluding this AT32 CDC device. The device USB identity is VID `2E3C` / PID `5740`. A Configurator recognition fix is maintained separately; no STM32 USB identity is spoofed in this firmware.

## 한국어 요약

- 이 브랜치는 AT32F435 보드용입니다. STM32F405 버전과 구분하세요.
- `BETAFPVF435`는 내부 타깃명이며, 배포 파일명에는 `AT32F435`를 함께 표시합니다.
- 특정 기체용 펌웨어가 아닙니다. ESC1~4는 보드 표기를 따르며, 서보 배치·모터 극수·테일 형식은 사용자가 설정합니다.
- FC 펌웨어, Bluejay ESC 펌웨어, ELRS 수신기 펌웨어는 각각 별도입니다.
