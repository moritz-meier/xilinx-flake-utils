# Getting started

First, checkout the outputs provided by this flake: `nix flake show`. It
contains builds for both __Vitis__ and __Vivado__. Note that typically the
__Vitis__ package include a version of vivado, so choose __Vitis__ if you want
both.

__Note__: If you experience an issue, pay the
 [troubleshooting section](docs/TROUBLESHOOTING.md) a visit!

# Example usage

__Note__: this presumes that your current environment contains various
software. One way of entering an environment that fulfills this requirement
would be to enter a nix development shell via `nix develop`.

Typically, you start of by creating a project using the `create-project` command. You shall have to chose the actual platform (zynq7000, ultrascale, zedboard or coraz7) you want your project to be build up with, the directory where the project should be located and the project name. Optionally you can select a template (lane-commanding, lane-monitoring) to apply a basic configuration of either a commanding or monitoring fcc lane.  
This is followed by the generation of the hardware description with the `build-hw-config` command. The only argument you shal pass is the directory of your hardware project you want to generate. If you chose a zynq7000 or ultrascale platform in the previous step you are asked to select the carrier (devboard or fcc) for which the harwdare configuration should be generated. The script then automatically selected the correct constraints file expecting that you have specified the required constraints in the corresponding file. After the script is completed it automatically creates an export directory where the hardware configuration can be found.  
The next step is the building of the bootloader using the `build-bootloader` command. You need to decide which type of bootloader shall be builded (fsbl or u-boot) and enter the directory of the hardware project. In the case of the FSBL the script builds the first stage bootloader and export the artifacts to a created export directory. In the case of U-Boot the script creates a U-Boot image based on the hardware configuration. This image acts as a second stage bootloader and offers advanced functionalities like initialization of the network interface compared to the FSBL. U-Boot can be used to fetch and boot application software images via a TFTP server. If a template (lane-commanding, lane-monitoring) was chosen before the script automatically applies patches to the U-Boot configuration which are required for the corresponding FCC lanes.  
Now a software image can be loaded and booted via JTAG (requires a builded FSBL) using the `jtag-boot` command when a target is connected to the host pc. It requires the directory of the hardware project for which the FSBL bootloder was build and an application.elf file. The boot steps are printed on the console. The `lanch-picocom` command can then be used to open a terminal to view the prints generated by your running application on the processor.  
If you want to load the U-Boot image you need to execute the `build-image` command and passing the project directory. This script takes the previously builded harwdare configuration, FSBL and U-Boot images and compiles them to a BOOT.bin file. This BOOT.bin file can then be loaded into the non-volatile flash memory of the connected SoC using the `program-flash` command. All that is required by you is to pass the project directory. At the end of the script you need to follow the instructions printed in the console to finally complete the process. Otherwise your board will not be able to succesfully fetch application software images from a TFTP server. 

Further, a `store` script is provided to compress your Vivado harwdare project into just one tcl file (restore.tcl). This enables you to manage your hardware project in a GitHub or GitLab repository without uploading the whole IDE related files. The `restore` command can be used to rebuild the Vivado project from the restore.tcl file (usefull after a pull or checkout). 

This is an example for the usage of all functions.
```console
create-project zynq7000 /home/Documents/example_dir zynq7000_example -t lane-commanding
build-hw-config example_dir/zynq7000_example -c fcc
build-bootloader fsbl example_dir/zynq7000_example
jtag-boot example_dir/zynq7000_example application.elf
launch-picocom 
build-bootloader u-boot example_dir/zynq7000_example
build-image example_dir/zynq7000_example
program-flash example_dir/zynq7000_example
store example_dir/zynq7000_example
retore example_dir/zynq7000_example
```

