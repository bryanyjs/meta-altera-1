**This is a forked Version of `kraj/meta-altera`, the official OpenEmbedded/Yocto BSP layer for Altera SoCFPGA platforms.**

**With this layer is the board support package (BSP) for *ARM* based *Intel (ALTERA) SoC-FPGAs* added to the Yocto Project.**

Usually the Yocto project can generate all required components (rootfs, device tree, bootloaders,...) to boot up a final embedded Linux. But this is not compatible with [Intel's Boot flow](https://www.intel.com/content/dam/www/programmable/us/en/pdfs/literature/an/an709.pdf).
This Boot flow uses the Intel Embedded Design Suite (EDS) to build the device tree and all necessary bootloader. 

For that reason, I forked this BSP-layer and I removed all nonrequired parts.
This includes the board specific u-boot- and device tree-generation and the support for only the *.tar.gz*-file type for the rootFs. 

I used this layer to build [*rsYocto*](https://github.com/robseb/rsyocto), an open source embedded Linux System for Intel SoC-FPGAs, by me own. 
 

## Supported Device families 

| **Device Family** | **Architecture** | **Machine Name**
|:--|:--|:--|
| Intel (ALTERA) **Cylone V** | *ARMv7A* | *MACHINE ="cyclone5"*
| Intel (ALTERA) **Arria V**  | *ARMv7A* | *MACHINE ="arria5"*
| Intel (ALTERA) **Arria 10** | *ARMv7A* | *MACHINE ="arria10"*
| Intel (ALTERA) **Stratix 10** | *ARMv8A* | *MACHINE ="stratix10"*
| Intel (ALTERA) **Agilex** | *ARMv8A* | *MACHINE ="agilex"*

## Linux Kernel Types

| **Linux Version Name** | **Version Type** | **Supported Linux Kernel Versions** 
|:--|:--|:--|
| *"linux-altera"* | **Regular Linux Version** | `5-0`, `5.1`, `5.2`, `5.3`
| *"linux-altera-ltsi"* | **Long term stable Linux Version (LTSI)** | `4.14.130` 
| *"linux-altera-ltsi-rt"* | **Long term stable Linux Version (LTSI) with real time support** |  `4.14.126`

**The Linux Kernel source code is available on this official [Intel (ALTERA) repository](https://github.com/altera-opensource/linux-socfpga)**. 


## Usage of this BSP-layer

The following step by step guide shows how to use this layer to build a Yocto-based Linux System for an Intel SoC-FPGAs:
1. Step: **Install the latest Version of the Yocto project**
	* As a Building machine I prefer regular Ubuntu-Linux
	````bash
	 git clone git://git.yoctoproject.org/poky
	````
2. Step: **Download this BSP-layer**
	````bash
	cd poky/
	git clone https://github.com/robseb/meta-altera.git
	````
	Your *poky*-folder should now look like this:
	+++++
3. Step: **Run the bitbake initialization script**
	````bash 
	source oe-init-build-env
	````
	* Do not run this command or any other Yocto command as root!
	* Do not use the command: “*sudo ./ oe-init-build-env*”.  With this line would Yocto crash later by the build process without any traceable error message  
	* A rerun of this script do not over write the development state
	* The script should response with following and create the folder "/build"
	++++++
4. Step: **Add this BSP-layer to your Yocto project solution**
	* Open the **"bblayers.conf"**-file *(poky/build/conf)* with a text editor, for example with *MS Visual Studio Code*:
		````bash 
		code conf/bblayers.conf
		````
	* At following line to this file to include the BSP-layer:
		````bitbake
		/home/vm/poky/meta-altera \
		````
		* **Note:** Replace the user name *"vm"* with your user name
	* Now should the *"bblayers.conf"*-file look like this:
		````bitbake
		# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
		# changes incompatibly
		POKY_BBLAYERS_CONF_VERSION = "2"

		BBPATH = "${TOPDIR}"
		BBFILES ?= ""
		BBLAYERS ?= " \
		  /home/vm/poky/meta \
		  /home/vm/poky/meta-poky \
		  /home/vm/poky/meta-yocto-bsp\
		  /home/vm/poky/meta-altera \
		"
		````
5. Step: **Configure the machine type and Linux Version**
	* Open the **"local.conf"**-file *(poky/build/conf)* with a text editor, for example with *MS Visual Studio Code*:
		````bash 
		code conf/local.conf
		````
	* **Select your Intel SoC-FPGA family** by adding the value **"MACHINE** this configuration file
		* For the different devices use string of the table above
		* For example, for an Intel Cyclone V SoC-FPGA add following to this file:
			````bitbake
			MACHINE ="cyclone5"
			````
		* Be sure that default *"qwmux86"* is **removed**
			````bitbake
			# MACHINE ??= "qemux86"
			````
	* **Select the Linux Kernel type**
		* If you want to use the regular **ALTERA socfpga-Linux Kernel** add the line above to the **"local.conf"**-file:
			````bitbake
			PREFERRED_PROVIDER_virtual/kernel = "linux-altera"
			````
		* If you want the **long term stable (LTSI) ALTERA socfpga-Linux Kernel** use this line:
			````bitbake
			PREFERRED_PROVIDER_virtual/kernel = "linux-altera-ltsi"
			````
	* **Select the Linux Kernel Version**
	 	* With following code line,it is possible to select the preferred Linux Kernel Version (here with Version 5.3)
			````bibtabe
			PREFERRED_VERSION_linux-altera = "5.3%"
			````
		* All supported Linux Kernel versions are listed above
	* **Choosing Toolchain Versions**
		* The following lines select the GCC and SDK Version:
			````bibtabe
			GCCVERSION = "linaro-5.2"
			SDKGCCVERSION = "linaro-5.2"
			````
		*Add this two lines to the **"local.conf"**-file independent of your machine choose 
 	* **Select the used CPU Version** *
	* For an Dual Core Intel (ALTERA) **Cyclone V**, **Arria 5** or **Arria 10** add the following line to the "**local.conf"**-file:
		````bibtabe
		DEFAULTTUNE = "cortexa9hf-neon"
		````
		* This selects the ARMv7 Cortex-A9 dual core CPU with the NEON-Engine and a vector floating-point unit
	* **Save and close this file**
	
6. Step: **Check if your settings are vailed and executable**
	* The following shell-command lists all for this build used layers (executed inside *poky/build/*):
		````bibtabe
		bitbake-layers show-layers
		````
		* If an error occurred properly something with the "**local.conf**- or "**bblayers.conf"**-file went wrong
	* This command give the used Linux Kernel Version
		````bibtabe
		bitbake --show-versions | grep linux  
		````
6. Step: **Optimal: Change the Linux Kernel configuration**
	* To configure the Linux property for a specific device family it is necessary to change the Linux Kernel configuration
	* But for a first Yocto project build is the Linux Kernel configured well enough
	* Read and change the BSP-layer with **"defcongig"**
		* One part is configured by a "*defconfig*-file"
		* With that it is possible to enable or disenable every component, like for example ETHERNET, CAN, EXT2, HPS-Bridges and PCI
		* The following bitbake shell-command stores the "*defconfig*-file locally (executed inside *poky/build/*)
		````bash
		bitbake -c savedefconfig virtual/kernel 
		````
		* This command prints the directory of the saved file at the end
	* Read and change the Linux Kernel with **menueconfig**
		* Use this command to start the "*menueconfiguration*"-tool:
		````bash
		bitbake -c menuconfig -f virtual/kernel
		````
		* A window like this should appear: 
		+++++++++
		
		* Here it is possible to change any kernel setting, ARM-Platform specific settings or enable or disable some peripheral components
	* To execute the any BSP-layer change use following command:
		````bash
		bitbake -f -c compile virtual/kernel && bitbake -f -c deploy virtual/kernel
		````
7. Step: **Build the complete Yocto-project**
	* With this command the full yocto project build process starts (executed inside *poky/build/*): 
	````bash
	bitbake core-image-minimal
	````
	* This process can take some time
	* For an Intel Arria 10 SoC-FPGA following start print should appear:
	+++++++++++++++
	* This shows that bitbake was able to decode the previously shown configuration 
	
8. Step: **Locate the final Kernel- and rootFs-File** 
	* After a successful build the final compressed Linux Kernel file and the rootfs tar.gz- archive this here stored: 
		* for an **Intel Cyclone V:**
		````txt
		poky/build/tmp/delopy/images/cyclone5/
		````
		* for an **Intel Arria 10:**
		````txt
		poky/build/tmp/delopy/images/arria10/
		````
	* The rootFs-file is called: **core-image-minimal-cyclone5-<*Date Code*>.rootfs.tar.gz**
	* The Linux Kernel file is called: **zImage-<*...+>.bin**
	* Be sure that the files are **not a Shortcut**!

At this point a Linux for an Intel SoC-FPGA is generated. Unfortunately to boot this up a also an device tree, a primary- and secondary bootloader and for Intel Arria and Stratix are two FPGA configuration files required.
# Continuation
Soon, I will continue my work and upload I guide to show all required steps to my [*rsYocto*-project](https://github.com/robseb/rsyocto).

For adding a toll to access the FPGA-Manager or to execute shell scripts at boot up you can use my [**meta-rstools**](https://github.com/robseb/meta-rstools)-layer.
Also a wrote a python script to automatically **pre-install Python pip (PyPI)- Packages within a final Yocto Project Linux Image** (see [here](https://github.com/robseb/PiP2Bitbake)).

# Credits & Contribution
Big thanks to [**Khem Raj**](https://github.com/kraj) for this maintaince work!

# Author
* **Robin Sebastian**

This guide is my own non official solution, as a part of the is a academic project [*rsYocto*](https://github.com/robseb/rsyocto).
Today I'm a Master Student of electronic engineering with the major embedded systems. 
I ‘m looking for an interesting job offer to share and deepen my shown skills starting summer 2020.
