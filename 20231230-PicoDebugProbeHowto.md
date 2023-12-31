# December 2023 Guide To Pico Debug Probe With Ubuntu 22.04 64 bits

## Prerequisites

- A Raspberry Pi Pico Debug Probe, cf. <https://www.raspberrypi.com/products/debug-probe/>
- An Ubuntu 22.04 64 bits machine (x86-64 in my case)

## Firmware setup

The Raspberry Pi Pico Debug Probe uses the `debugprobe.uf2` UF2 file.

Grab it from <https://github.com/raspberrypi/picoprobe/releases>.

Install it on the probe in `bootsel` mode:

- Unplug probe from USB
- Open probe lid
- Press Bootsel button
- Plug probe back to USB
- Wait for `RPI-PI2` mount point to appear
- Release Bootsel button
- Drag `debugprobe.uf2` to `RPI-PI2`
- Close probe lid

NB:

- `picoprobe.uf2` is intended for when you use a "real" Pico
- Pico W is not compatible with `picoprobe.uf2`

## Software installation

### OpenOCD

It seems CMSIS-DAP support needs you to build Raspberry Pi's version of OpenOCD:

```bash
cd ~/src
git clone https://github.com/raspberrypi/openocd
cd openocd
# Should be the defaut one
git checkout rp2040-v0.12.0
./configure
make -j($nproc)
sudo make install
```

This assumes you have already installed a C compiling toolchain, this should be a good start:

```bash
sudo apt install build-essential autoconf automake
```

Make sure your user is part of `dialout` and `plugdev` groups, or only `root` will be able to launch OpenOCD.

```bash
sudo cp -pv ./contrib/60-openocd.rules /etc/udev/rules.d
sudo systemctl restart udev
```

### Other tools

We need multi-architecture versions of binutils and gdb:

```bash
sudo apt install binutils-multiarch gdb-multiarch
```

Even with this installed, scripts try to launch `nm-multiarch` and `objdump-multiarch`, so we need to have symbolic links to them:

```bash
sudo ln -s /usr/bin/nm /usr/local/bin/nm-multiarch
sudo ln -s /usr/bin/objdump /usr/local/bin/objdump-multiarch
```


## VS Code


For my example project for HAGL's Pico VGA board (<https://github.com/CHiPs44/hagl_pico_vgaboard>), a working `.vscode/launch.json` is:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "cwd": "${workspaceRoot}",
            "executable": "./example/build/hagl_pico_vgaboard_example.elf",
            "name": "Debug example with OpenOCD",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "openocd",
            "gdbPath": "gdb-multiarch",
            "configFiles": [
                "interface/cmsis-dap.cfg",
                "target/rp2040.cfg"
            ],
            "searchDir": [
                "/usr/local/share/openocd/scripts/"
            ],
            "showDevDebugOutput": "raw",
            "openOCDLaunchCommands": [
                "adapter speed 5000"
            ],
            "svdFile": "${env:PICO_SDK_PATH}/src/rp2040/hardware_regs/rp2040.svd",
            "runToEntryPoint": "main",
            "postRestartCommands": [
                "break main",
                "continue"
            ],
            "liveWatch": {
                "enabled": true,
                "samplesPerSecond": 4
            }
        }
    ]
}
```

Parameter `executable` must be changed to path of the ELF file for your project in the build directory.

For `svdFile`, you must have a clone of Pico SDK, and `PICO_SDK_PATH` environment variable must point to it, but it's rather standard Pico stuff.

### VS Code extension

It seems that installing **Cortex-Debug** extension from **marus25** is now sufficient to launch debugging with `F5`.

`EOF`

