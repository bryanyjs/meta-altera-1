**This a forked Version of 'kraj/meta-altera', the official OpenEmbedded/Yocto BSP layer for Altera SoCFPGA platform.**
The difference between these two versions are, that the today nonfunction- and not required part, to build Linux systems with the Yocto project, are removed. 
This includes the board specific u-boot- and device tree-generation and the support for only the *.tar.gz*-file type for the rootFs. 
I forked this meta layer to use it for the development of rsYocto, an embedded yocto based Linux system for Intel SoC-FPGAs. 

## Supported Device families 

| Device Family | Architecture | Machine Name
|:--|:--|:--|
| Intel (ALTERA) **Cylone V** | *ARMv7A* | *MACHINE ="cyclone5"*
| Intel (ALTERA) **Arria V**  | *ARMv7A* | *MACHINE ="arria5"*
| Intel (ALTERA) **Arria 10** | *ARMv7A* | *MACHINE ="arria10"*
| Intel (ALTERA) **Stratix 10** | *ARMv8A* | *MACHINE ="stratix10"*
| Intel (ALTERA) **Agilex** | *ARMv8A* | *MACHINE ="agilex"*

## Linux Kernel Types

| Linux Version Name | Version Type
|:--|:--|
| *"linux-altera"* | **Regular Linux Version**
| *"linux-altera-ltsi"* | **Long term stable Linux Version (LTSI)**

** The Linux Kernel source code is avalibil on this offical Intel (ALTERA) repository. 



## Usage of this BSP-layer

The following step by step guide shows how to use this layer to build a Yocto-based Linux System for an Intel SoC-FPGAs:
1. Step: **Install the latest Version of the Yocto project**
	* As a Building machine I prefere regular Ubuntu-Linux
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
	* Do not use the command: “sudo ./ oe-init-build-env”.  With this line would Yocto crash later by the build process without any traceable error message  
	* A rerun of this script do not over write the development state
	* The script should response with following and create the folder "/build"
	++++++
4. Step: **Add this BSP-layer to your yocto project solution**
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
	* **Select your Intel SoC-FPGA famaly** by adding the value **"MACHINE** this configuration file
		* For the diffrent devices use string of the tabel above
		* For example for an Intel Cyclone V SoC-FPGA add following to this file:
			````bitbake
			MACHINE ="cyclone5"
			````
		* Be sure that default "qwmux86" is removed
			````bitbake
			# MACHINE ??= "qemux86"
			````
	* **Select the Linux Kenrel type**
		* If you want to use the regualr **ALTERA socfpga-Linux Kernel** add the line above to the **"local.conf"**-file:
			````bitbake
			PREFERRED_PROVIDER_virtual/kernel = "linux-altera"
			````
		* If you want the **long term stable (LTSI) ALTERA socfpga-Linux Kernel** use this line:
			````bitbake
			PREFERRED_PROVIDER_virtual/kernel = "linux-altera-ltsi"
			````
	* **Select the Linux Kernel Version**
	 	* With following code line it is posible to select the prefered Linux Kernel Version (here with Version 5.3)
			````bibtabe
			PREFERRED_VERSION_linux-altera = "5.3%"
			````
		* All supported Linux Kernel versions are listed above
	* **Choosing Toolchain Versions**
		* The following lines select the GCC and SDK Version:
			````bibtabe
			GCCVERSION = "linaro-5.2"
			SDKGCCVERSION = "linaro-5.2"
			GCCVERSION = "linaro-5.2" 
			SDKGCCVERSION = "linaro-5.2"
			````
		* Add them to the **"local.conf"**-file indemened of your machine choose
 	* **Select the used CPU Version** *
	* For an Dual Core Intel (ALTERA) **Cyclone V**, **Arria 5** or **Arria 10** add the following line to the "**local.conf"**-file:
		````bibtabe
		DEFAULTTUNE = "cortexa9hf-neon"
		````
			* This selects the ARMv7 Cortex-A9 dual core CPU with the NEON-Engine and a vector floating-point unit
		* For an Intel (ALTERA) **Cyclone V**, **Arria 5** or **Arria 10** add the following line to the "**local.conf"**-file:
		````bibtabe
		DEFAULTTUNE = "cortexa9hf-neon"
		````
			* This selects the ARMv7 Cortex-A9 dual core CPU with the NEON-Engine and a vector floating-point unit
	* **Save and close this file**
	
6. Step: **Check if your settings vailed and executebil**
	* The following shell-command lists all for this build used layers (executed inside *poky/build/*):
		````bibtabe
		bitbake-layers show-layers
		````
		* If an error ocurre properly something with the "**local.conf**- or "**bblayers.conf"**-file went wrong
	* This command give the used Linux Kernel Version
		````bibtabe
		bitbake --show-versions | grep linux  
		````
6. Step: **Optinal: Change the Linux Kernel configuration**
	* To configure the Linux problery for a spezific device famaly it is nessary to change the Linux Kernel configuration
	* But for a first Yocto project build is the Linux Kernel confirgered well enough
	* Read and change the BSP-layer with **"defcongig"**
		* One part is configerd by a "*defconfig*-file"
		* With that it is possible to enable or disenable every component, like for example ETHERNET, CAN, EXT2, HPS-Bridges and PCI
		* The following bitbake shell-commad stores the "*defconfig*-file localy (executed inside *poky/build/*)
		````bash
		bitbake -c savedefconfig virtual/kernel 
		````
		* This command prints the direcory of the saved file at the end
	* Read and change the Linux Kernel with **menueconfig**
		* Use this command to start the "*menueconfiguration*"-tool:
		````bash
		bitbake -c menuconfig -f virtual/kernel
		````
		* A window like this should appear: 
		+++++++++
		
		* Here is is possible to change any kernel setting, ARM-Platform specific settings or enable or disable some peripheral components
	* To execute the any BSP-layer chnage use following command:
		````bash
		bitbake -f -c compile virtual/kernel && bitbake -f -c deploy virtual/kernel
		````
7. Step: **Build the complite Yocto-project**
	* With this command the full yocto project build process starts (executed inside *poky/build/*): 
	````bash
	bitbake core-image-minimal
	````
	* This procres can take some time
	* For an Intel Arria-FPGA following start print should appear:
	+++++++++++++++
	* This shows that bitbake was able to decode the previously shown configuration 
	
8. Step: **Locate the final Kernel- and and rootFs-File** 
	* After a sucsefull build the final compressed Linux Kernel file and the rootfs tar.gz-arvice this avibile here: 
Maintainer(s): Khem Raj <raj.khem@gmail.com>
