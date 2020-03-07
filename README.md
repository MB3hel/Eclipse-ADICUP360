# Eclipse-ADICUP360

FOSS eclipse-based development environment for ADICUP360.

## Components
- IDE: Eclipse w/ GNU MCU plugins
- Toolchain: GNU ARM toolchain
- Debugger: GDB
- Board debug and interface tool: openocd (originally modified by Analog Devices, forked by me)

## Setup Instructions Linux
- Install the toolchain
    ```
    curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
    sudo apt install nodejs
    sudo npm install -g xpm
    xpm install --global @xpack-dev-tools/arm-none-eabi-gcc@latest
    ```


- Install [GNU MCU Eclipse](https://gnu-mcu-eclipse.github.io/install/)
- Extract it to some location (`/opt/gnu-mcu-eclipse` for example)


- Clone forked version of OpenOCD patched by Analog Devices to support the ADICUP360

    ```
    sudo apt install build-essential automake autoconf libhidapi-dev
    git clone https://github.com/MB3hel/openocd-aducm360.git ~/openocd-aducm360
    cd openocd-aducm360
    ./bootstrap
    export CFLAGS="-Wno-error=implicit-fallthrough= -Wno-error=misleading-indentation -Wno-error=format-overflow="
    export CPPFLAGS=$CFLAGS
    ./configure --enable-cmsis-dap --prefix=/opt/gnu-mcu-eclipse/eclipse/openocd-aducm360/
    make -j
    make install
    ```

    - OpenOCD supporting the ADICUP360 will now be in `/opt/gnu-mcu-eclipse/eclipse/openocd-aducm360`

- Add udev rules to fix errors with accessing the device if not running eclipse as root. Using arduino openocd rules

    ```
    cd /etc/udev/rules.d
    sudo wget https://raw.githubusercontent.com/arduino/OpenOCD/master/contrib/60-openocd.rules
    sudo udevadm control --reload-rules
    ```

    - The add your user to the plugdev group and log out then back in.
    

- Note: zip releases may be put in releases section. If using these be sure to install `libhidapi-dev` or openocd will not work. Also add the udev rules. Also still need to install the toolchain.

## Creating a Project
- Open GNU MCU Eclipse
- Select a workspace
- Create a C/C++ Project
- Choose C++ Managed build
- Choose ADuCM36x C/C++ Project under the Executable section and give the project a name. Then click next.
- Settings (only the ones that matter are listed. If not listed just pick one.)
    - Processor core = ADuCM360
    - Use system calls: Freestanding
    - Trace output: Semihosting DEBUG channel
    - Check the following boxes:
        - Check some warnings
        - Use -Og on debug
        - Use newlib nano
- Click next twice more
- Choose "GNU MCU Eclipse ARM Embedded GCC (arm-non-eabi-gcc)" for toolchain name. Leave the path empty for now.
- Open the properties of the new project
- Under MCU Choose "ARM Toolchains Paths"
- Click xPack... near toolchain folder and select the installed version of gcc.
- Click apply and close
- Build. Should build successfully.
- Create a new launch configuration (GDB OpenOCD Debugging)
- Under debugger tab
    - Executable path = `${eclipse_home}/openocd-aducm360/bin/openocd`
    - Config options = `--search ${eclipse_home}/openocd_aducm360/share/openocd/scripts -f board/eval-adicup360.cfg`
- Click run with the new launch configuration and the program should run and the debugger will be attached.