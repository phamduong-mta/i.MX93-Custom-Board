
i.MX93 CUSTOM BOARD --- DVI-TO-HDMI AND USB-HUB-4-PORT

Includes hardware design and hardware schematic diagram of i.MX93 board based on i.MX93 dev kit.

A brief guide on how to bring up the operating system, customize the kernel and devicetree to identify all peripherals.

A guide on how to test all peripherals on the above custom circuit.

## Authors
Gradien-MTA

## Appendix
1. Design PCB
2. Schematic
3. Setup environment and build OS
4. Custom Kernel
5. Custom Devicetree
6. Deploy and Test Peripherals

## 1. Design PCB
Included in source
## 2. Schematic
Included in source
## 3. Setup environment and build Yocto for iMX93 board
Git repo
```bash
mkdir ./bin
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ./bin/repo
chmod a+x ./bin/repo
PATH=${PATH}:./bin
```

Dowload and sync source
```bash
mkdir source_build && cd source_build  
../../bin/repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-mickledore -m imx-6.1.55-2.2.2.xml
../../bin/repo sync
```

Create name's forlder and build
```bash
MACHINE=imx93evk DISTRO=fsl-imx-xwayland source ./imx-setup-release.sh -b imx93-remake  
bitbake imx-image-multimedia
```

## 4. Custom Kernel
Open the menu that includes the necessary drivers.
```bash
bitbake -c devshell virtual/kernel
make menuconfig
```

Added drivers for HDMI controller ADV-7535 and USB-HUB 2514b (tick "y" to build in driver)
```bash
Device Drivers  --->
  Graphics support  --->
    Direct Rendering Manager (X.Org support)  --->
      Display Interface Bridges  --->
        Analog Devices ADV7511/33/35/37 HDMI transmitter

Device Drivers  --->
  USB support  --->
    USB Physical Layer drivers  --->
        Microchip USB251xb Hub Controller Configuration Driver

Device Drivers  --->
  USB support  --->
    <*> Support for Host-side USB
```

To deploy this image run
```bash
make -j$(nproc) Image
```

## 5. Custom Devicetree
Turn off typeC port number 2 and add usb hub 2514b
```bash
	ptn5110_2: tcpc@51 {
		compatible = "nxp,ptn5110";
		reg = <0x51>;
		interrupt-parent = <&gpio3>;
		interrupts = <27 IRQ_TYPE_LEVEL_LOW>;
		status = "disabled";
	};

	usb-hub@2c {
		pinctrl-names = "default";
		compatible = "microchip,usb2514b";
		reg = <0x2c>;
		individual-port-switching;
		self-powered;
	};
```

Add and declare dvi to hdmi converter node
```bash

	adv7535: hdmi@3d {
		compatible = "adi,adv7535";
		reg = <0x3d>;
		adi,addr-cec = <0x3b>;
		adi,dsi-lanes = <4>;
		status = "okay";

		port {
			adv7535_to_dsi: endpoint {
				remote-endpoint = <&dsi_to_adv7535>;
			};
		};
	};

&dsi {
	status = "okay";

	ports {
		port@1 {
			reg = <1>;

			dsi_to_adv7535: endpoint {
				remote-endpoint = <&adv7535_to_dsi>;
			};
		};
	};
};
```

To deploy this devicetree run
```bash
make -j$(nproc) dtbs
```

## 6. Deploy and Test Peripherals
Folder containing files to deploy the operating system via usb port and uuu tools included
```bash
- imx-boot-imx93evk-sd.bin-flash_singleboot (file boot)
- imx-image-multimedia-imx93evk.rootfs.wic (file OS)
- uuu tool (download from: https://github.com/NXPmicro/mfgtools/releases/tag/uuu_1.5.165)
- uuu.auto-imx93evk (file text in source)
```

To deploy this project run
```bash
- Change boot mode to serial download mode (refer to boot instructions on imx93 devkit)
sudo ./uuu ./uuu.auto-imx93evk
```

After entering the operating system, reset the board, at uboot run the command
```bash
ums 0 mmc 0

- Connect the circuit to typeC port 1 then connect to the computer.

- The folder containing the kernel will be mounted to the computer.

- Then copy the Image and Devicetree files customized above to replace them.
```

Test Ether 0 và 1
```bash
udhcpc eth0/eth1 or ifconfig eth0 <ipaddr> <netmask>
ping <gateway or any pc on LAN>
```

Test sdcard
```bash
- Change boot mode to sdcard boot mode (refer to boot instructions on imx93 devkit)
- Using balenEcher to write imx-image-multimedia-imx93evk.rootfs.wic (file OS) to sd card and plugin sdcard slot then power on board.

```

Test wifi
```bash
modprobe moal mod_para=nxp/wifi_mod_para.conf
wpa_passphrase <Wifi name> <Password> >> /etc/wpa_supplicant.conf
wpa_supplicant -B -i mlan0 -c /etc/wpa_supplicant.conf -D nl80211
udhcpc -i mlan0
```

Test controller HDMI
```bash
- Just plugin HDMI connector to board and Display, power on logo NXP appear.
dmesg | grep adv75xx (check if driver is built in kernel)
```

Test USB-Hub
```bash
- The usb-hub port will be recognized and when the peripheral is plugged in, it will automatically detect and supply power to the peripheral (eg keyboard, mouse, ...)
dmesg | grep usb251x (check if driver is built in kernel)
```

## Support

For support, email phamvanduongqxth@gmail.com or call "Team Three Knives".

