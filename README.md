# vitis_platform
Vitis Platform Generate Process


source vivado setting64.sh
sudo /opt/xilinx/Vivado/2019.2/bin/vivado

# Vivado : hardware description (xsa) generation

### On GUI,
  - Create Project : check default 
  - Board Search : What? I don't have ultra96v2 board design files(bdf)
  - Return the CLI

### For Download BDF,
  - wget https://github.com/Avnet/bdf/archive/master.zip
  - unzip master.zip
  - cp -rf bdf-master/* /opt/xilinx/Vivado/2019.2/data/boards/board_files
  
### For Install Xilinx USB--JTAG Pod Driver,
  - cd /opt/xilinx/Vivado/2019.2/data/xicom/cable_drivers/lin64/install_script/install_drivers
  - sudo ./install_drivers
  
### On Open GUI vivado, 
  - Create Project with ultra96v2 baord design files
  - Create IPI Block Design Zynq Ultrascale+ MPSoC Processing System IP & Auto connection
  - Create clocking wizard block 5ea and also reset(Processor System Reset) block 5ea 
  - Create a Concat block and set value 1 for 'Number of Port'
  - Connetion making : 
      PS - clkwiz_0, 
      clkwiz_0 - proc_sys_reset_#, 
      pl_reset0 - ext_reset_in, 
      xlconcat's dout - pl_ps_irq[0:0]

### Platform Interface & Property Description,
  - Window -> Platform Interface -> Enable Platform Interface
  - Board Name, Interface Ports Enable, Set default Clock property (is_default, id=0)
  - XRT : Tcl command type to make platform design intent
  - Validate on Block Design and Bitstream generate, xsa export...
  - Tcl CLI : validate_hw_platform <Write Path> /ultra96base.xsa
  
  
  
# PetaLinux Build : bsp and sdk file generation
  - in vivado project location
  - petalinux-create -t project -n petalinux --template zynqMP
  - cd petalinux
  - petalinux-config --get-hw-description=../ultra96base (location of xsa, do not use empty space at type)
  - configuration menu window
      * DTG Settings ---> MACHINE_NAME : avnet-ultra96-rev1
      * Subsystem AUTO Hardware Settings ---> Serial Settings ---> psu_uard_1
      * Image Packaging Configuration ---> Root filesystem type ---> EXT(SD/ eMMC/ QSPI/ STAT/ USB)
      
  - for XRT using, Open gedit 'user-rootfsconfig'
      * gedit ./project-spec/meta-user/conf/user-rootfsconfig
      
              CONFIG_xrt
              CONFIG_xrt-dev
              CONFIG_zocl
              CONFIG_opencl-clhpp-dev
              CONFIG_opencl-headers-dev
              CONFIG_packagegroup-petalinux-opencv
              
  - Edit device tree to use xrt driver at zynq series
      * gedit ./project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
  - petalinux-config -c rootfs
  - on configuration menu window, user packages ---> select all 6ea of programs by spacebar ---> exit
  - For DMA Transfer memory margine, kernel need to some task.
      * petalinux-config -c kernel
      * On configuration menu window, 
            Device Drivers ---> Generic Driver Options ---> 
            DMA Contifuous Memory Allocator ---> Size in Mega Butes 
            Click and Change value 256 -> 1024 : exit (and save)
  - petalinux-build
  
  ###### cf. petalinux-build error : device tree typing error revision
      * petalinux-build -x distclean
      * petalinux-build

  
  - cd images/linux
  - petalinux-build --sdk
  - cd ../../..   <--- for finding sdk.sh file
  
  
