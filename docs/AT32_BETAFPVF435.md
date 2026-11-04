# BETAFPV 20A AIO / AT32F435 port

## Scope and provenance

This independent branch is based on Rotorflight 4.6.0 commit `118e9120260bb33f46df4f92052fb0e9fd4e9ebc`.
It retains the Rotorflight flight-control/configuration layer and adds the AT32 hardware port. Board defaults reflect the Align T15 wiring used during development; other helicopters need their own configuration.

Hardware used: BETAFPV F4 2–3S 20A AIO FC V1, AT32F435RGT7 (1MiB flash, default 288MHz), ICM42605, onboard UART ELRS, 16MiB SPI flash and Bluejay ESCs.
The original Betaflight target name `BETAFPVF435` is retained. `AT32` is the MCU family; `F435` identifies the model. Do not select this firmware for the STM32F405 board variant.

Porting references:

- [Rotorflight firmware](https://github.com/rotorflight/rotorflight-firmware), base commit above.
- [flightng/atbetaflight](https://github.com/flightng/atbetaflight), `4.3-bsp-rebase`, commit `6bf2667698b51adae90ef917005fbc5059dd4504`.
- [Betaflight](https://github.com/betaflight/betaflight), 4.5.2 commit `024f8e13d4e642eb6a380308685b9ea3aa3ef1a2`.
- Artery AT32F435/437 BSP v2.1.0, shipped with its original notices under `lib/main/AT32F43x`.

Original project licenses and vendor copyright notices remain in place. See [LICENSE](../LICENSE).

## Default T15 connections

| Function | Board connector | MCU / timer |
|---|---|---|
| Main motor / MOTOR1 | Physical ESC4 | PA2 / TMR2 CH3 |
| Tail motor / MOTOR2 | Physical ESC2 | PB1 / TMR3 CH4 |
| Swash servo1 | T1 | PA9 / TMR1 CH2 |
| Swash servo2 | R1 | PA10 / TMR1 CH3 |
| Swash servo3 | T6 | PC6 / TMR8 CH1 (AF3) |
| Onboard ELRS | UART3 | PB10 TX / PB11 RX |
| Battery voltage / total current | onboard ADC | PC2 / PC1 |

The TMR3 CH4 DShot output switches to CH3 indirect capture for bidirectional RPM reception. Do not allocate PB0/TMR3 CH3 as a concurrent extra output in this layout.
Defaults use DShot300, bidirectional RPM, motorized tail, 12 main-motor poles and 6 tail-motor poles. Those pole counts were physically checked on the development T15 motors. They are configuration defaults, not universal values for this board.
PWM servos use timers 1/8; motor timers are 2/3. DShot bitbang/burst defaults remain OFF for this wiring. Full aircraft settings and personal receiver binding data are not bundled with this repository.

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

The prepublication bench binary was built on 2026-11-04 at 13:20:03 with revision text `norevision`. Rebuilds can differ due to build time and Git revision metadata. Release assets include a SHA256 file for the distributed binary.

## Validation record

The publication snapshot has the same firmware code as the installed bench image. Documentation/ignore rules were added for publication.
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
- T15용 기본 출력 배치이며 다른 기체에 사용할 때는 서보·모터·수신기 설정을 해당 기체에 맞춰야 합니다.
- FC 펌웨어, Bluejay ESC 펌웨어, ELRS 수신기 펌웨어는 각각 별도입니다.
