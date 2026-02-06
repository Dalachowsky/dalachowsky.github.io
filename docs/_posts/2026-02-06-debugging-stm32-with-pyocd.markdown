---
layout: post
title:  "Debugging STM32 with pyOCD"
date:   2026-02-06
categories: guides stm32
permalink: /guides/stm32/pyocd-debugging
comments_id: 101
---

PyOCD allows for debugging STM32 targets regardless of the IDE or ecosystem you're using.
<!--more-->
This guide will show you how to debug STM32 devices using pyOCD + GDB.
Thanks to using pyOCD you can debug projects that are built using ST-provided tools
or e.g Zephyr RTOS.

**NOTE** This guide assumes that you have an `.elf` file compiled for your target already.
Remember to compile with debug symbols enabled.

# Preparation

We'll have to install pyOCD and the Arm GNU toolchain.

## Installing the Arm GNU toolchain

You'll need the Arm GNU toolchain for debugging the microcontroller.
Most likely you'll have it on your PC either from one of the ST-provided tools,
such as STM32CubeIDE or perhaps from Zephyr SDK.

If for some reason you do not have it installed, you can find the info on how to get it [here](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain).

Either way **make sure that it is available in your PATH**.

```bash
export PATH="$PATH:/path/to/arm/toolchain"
```

## Installing PyOCD

Start with installing the pyOCD itself. It can be safely installed
inside a [venv](https://docs.python.org/3/library/venv.html).

```bash
pip install pyocd
```

Now we have to install package for our target MCU.
PyOCD uses CMSIS-packs for adding support for MCUs.
You can read more about them [here](https://pyocd.io/docs/open_cmsis_pack_support.html).

Let's say we have a NUCLEO board with STM32F746ZG. We have to install the CMSIS-pack
for the F7 family:

```bash
pyocd pack install stm32f7
```

Now to validate if we have support for our target we can run:

```bash
pyocd list --target | grep stm32f746
```

This gives us a list of all targets that PyOCD currently supports. In the first column,
we can see the target name that we will use when connecting to the target.

```bash
stm32f746be               STMicroelectronics       STM32F746BE                  STM32F7 Series, STM32F746   pack     
stm32f746betx             STMicroelectronics       STM32F746BETx                STM32F7 Series, STM32F746   pack     
stm32f746bg               STMicroelectronics       STM32F746BG                  STM32F7 Series, STM32F746   pack     
stm32f746bgtx             STMicroelectronics       STM32F746BGTx                STM32F7 Series, STM32F746   pack     
stm32f746ie               STMicroelectronics       STM32F746IE                  STM32F7 Series, STM32F746   pack     
stm32f746iekx             STMicroelectronics       STM32F746IEKx                STM32F7 Series, STM32F746   pack     
stm32f746ietx             STMicroelectronics       STM32F746IETx                STM32F7 Series, STM32F746   pack     
stm32f746ig               STMicroelectronics       STM32F746IG                  STM32F7 Series, STM32F746   pack     
stm32f746igkx             STMicroelectronics       STM32F746IGKx                STM32F7 Series, STM32F746   pack     
stm32f746igtx             STMicroelectronics       STM32F746IGTx                STM32F7 Series, STM32F746   pack     
stm32f746ne               STMicroelectronics       STM32F746NE                  STM32F7 Series, STM32F746   pack     
stm32f746nehx             STMicroelectronics       STM32F746NEHx                STM32F7 Series, STM32F746   pack     
stm32f746ng               STMicroelectronics       STM32F746NG                  STM32F7 Series, STM32F746   pack     
stm32f746nghx             STMicroelectronics       STM32F746NGHx                STM32F7 Series, STM32F746   pack     
stm32f746ve               STMicroelectronics       STM32F746VE                  STM32F7 Series, STM32F746   pack     
stm32f746vehx             STMicroelectronics       STM32F746VEHx                STM32F7 Series, STM32F746   pack     
stm32f746vetx             STMicroelectronics       STM32F746VETx                STM32F7 Series, STM32F746   pack     
stm32f746vg               STMicroelectronics       STM32F746VG                  STM32F7 Series, STM32F746   pack     
stm32f746vghx             STMicroelectronics       STM32F746VGHx                STM32F7 Series, STM32F746   pack     
stm32f746vgtx             STMicroelectronics       STM32F746VGTx                STM32F7 Series, STM32F746   pack     
stm32f746ze               STMicroelectronics       STM32F746ZE                  STM32F7 Series, STM32F746   pack     
stm32f746zetx             STMicroelectronics       STM32F746ZETx                STM32F7 Series, STM32F746   pack     
stm32f746zeyx             STMicroelectronics       STM32F746ZEYx                STM32F7 Series, STM32F746   pack     
stm32f746zg               STMicroelectronics       STM32F746ZG                  STM32F7 Series, STM32F746   pack     
stm32f746zgtx             STMicroelectronics       STM32F746ZGTx                STM32F7 Series, STM32F746   pack     
stm32f746zgyx             STMicroelectronics       STM32F746ZGYx                STM32F7 Series, STM32F746   pack   
```

# Connecting to the target

Now that we have everything prepared we can connect to our board.
We will start a GDB server using PyOCD and then connect to it using GDB.

```bash
pyocd gdbserver --target stm32f746zgtx
```

If the connection is succesfull, you should see similiar output:

```bash
0000734 I Target type is stm32f746zgtx [board]
0000828 I DP IDR = 0x5ba02477 (v2 rev5) [dap]
0000866 I debugvar 'DbgMCU_APB1_Fz' = 0x0 (0) [pack_target]
0000871 I debugvar 'DbgMCU_APB2_Fz' = 0x0 (0) [pack_target]
0000871 I debugvar 'DbgMCU_CR' = 0x7 (7) [pack_target]
0000871 I debugvar 'TraceClk_Pin' = 0x40002 (262146) [pack_target]
0000871 I debugvar 'TraceD0_Pin' = 0x40003 (262147) [pack_target]
0000871 I debugvar 'TraceD1_Pin' = 0x40004 (262148) [pack_target]
0000871 I debugvar 'TraceD2_Pin' = 0x40005 (262149) [pack_target]
0000871 I debugvar 'TraceD3_Pin' = 0x40006 (262150) [pack_target]
0000940 I AHB-AP#0 IDR = 0x74770001 (AHB-AP var0 rev7) [discovery]
0000943 I AHB-AP#0 Class 0x1 ROM table #0 @ 0xe00fd000 (designer=020:ST part=449) [rom_table]
0000946 I [0]<e00fe000:ROM class=1 designer=43b:Arm part=4c8> [rom_table]
0000946 I   AHB-AP#0 Class 0x1 ROM table #1 @ 0xe00fe000 (designer=43b:Arm part=4c8) [rom_table]
0000948 I   [0]<e00ff000:ROM class=1 designer=43b:Arm part=4c7> [rom_table]
0000948 I     AHB-AP#0 Class 0x1 ROM table #2 @ 0xe00ff000 (designer=43b:Arm part=4c7) [rom_table]
0000951 I     [0]<e000e000:SCS v7-M class=14 designer=43b:Arm part=00c> [rom_table]
0000952 I     [1]<e0001000:DWT v7-M class=14 designer=43b:Arm part=002> [rom_table]
0000954 I     [2]<e0002000:FPB v7-M class=14 designer=43b:Arm part=00e> [rom_table]
0000955 I     [3]<e0000000:ITM v7-M class=14 designer=43b:Arm part=001> [rom_table]
0000957 I   [1]<e0041000:ETM M7 class=9 designer=43b:Arm part=975 devtype=13 archid=4a13 devid=0:0:0> [rom_table]
0000960 I [1]<e0040000:TPIU M7 class=9 designer=43b:Arm part=9a9 devtype=11 archid=0000 devid=ca1:0:0> [rom_table]
0000966 I CPU core #0: Cortex-M7 r0p1, v7.0-M architecture [cortex_m]
0000966 I   Extensions: [DSP, FPU, FPU_V5, MPU] [cortex_m]
0000966 I   FPU present: FPv5-SP-D16-M [cortex_m]
0000968 I 4 hardware watchpoints [dwt]
0000971 I 8 hardware breakpoints, 1 literal comparators [fpb]
0000988 I STDIO server started on port 4444 (core 0) [server]
0001033 I GDB server listening on port 3333 (core 0) [gdbserver]
```

We can see that the connection was succesfull and GDB server is listening
on port 3333. Now we can open another terminal and launch our debug session.
By typing the following commands we start GDB, connect to pyocd gdbserver, load
our firmware into the microcontroller and run it.

```bash
arm-none-eabi-gdb ./my_firmware.elf
target remote localhost:3333
load
continue
```

And this is everything you need to know. Now you can use GDB commands to debug your
firmware.

After you rebuild your firmware, you just have to type `load` and `continue` into the
GDB console again.

**NOTE** You can run make from inside of the GDB console. I usually `cd` into the `build`
directory and launch GDB from there, so that I can `make` and `load` without jumping
between terminals.
