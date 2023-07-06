# CANBUS-setup-documentation

## Setting CAN in (BTT Manta M5P + BTT CB1) + BTT EBB36

The following steps consists of instruction to setup CAN communication in between BTT Manta M5P with BTT CB1 embed onto it and BTT EBB36 with the help of CanBoot Bootloader that is to be installed.

### Initial setup

For the first time use of the BTT CB1, make sure the OS is installed and the Wifi is configured. For first time installation of the OS image, refer to the [Official CB1 GitHub release page](https://github.com/bigtreetech/CB1/releases) or the local [Fracktal CB1 documentation](https://github.com/FracktalWorks/BTT-Manta-CB1-Development-Documentation)

After the above steps, SSH into the CB1, preferably using 'Bitvise SSH Client'. Open the terminal and update the OS using `sudo apt update`. After the update, Follow the steps to install `USB to CAN` communication based klipper firmware in the BTT Manta M5P. Make sure klipper is installed in the CB1.

1. `cd ~/klipper/`
2. `make menuconfig`
```
[\*] Enable extra low level configuration options
 Micro controller Architecture (STMicroelectronics STM32 )
 Processor model (STM32G0B1 )
 Bootloader offset 8 KiB bootloader)
 Clock Reference (8 MHz crystal)
 Communication interface (USB to CAN bus bridge (USB on PA11/PA12))
 CAN bus interface (CAN bus (on PD0/PD1))
 (500000) CAN bus speed
```
After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'klipper.bin' file

Now, enter DFU mode by holding `BOOT` and tapping `RESET`, and then letting go of `BOOT`. Send `lsusb` in the terminal, and then search for a port `0483:df11`. If it is present, it means that the board is in DFU mode.

Flash the firmware by entering in the command:
```
make flash FLASH_DEVICE=0483:df11
```

In order to get the UUID of the board, enter the command in the terminal:
```
 ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
Store this UUID safely in order to use in the klipper's configuration.

Following are the steps to install CanBoot Bootloader in the EBB36 board in order to setup CanBus and get the unique UUID. 

SSH into the CB1 and enter the following commands:

1. `cd ~`
2. `git clone https://github.com/Arksine/CanBoot.git`
3. `cd ~/CanBoot`
4. `make menuconfig`

Here, we initially set the CanBoot Bootloader installation in the BTT EBB36 Toolboard. After `make menuconfig`, the following are to be set up:
```
    Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32G0B1)  --->
    Build CanBoot deployment application (8KiB bootloader)  --->
    Clock Reference (8 MHz crystal)  --->
    Communication interface (CAN bus (on PB0/PB1))  --->
    Application start offset (8KiB offset)  --->
(500000) CAN bus speed
()  GPIO pins to set on bootloader entry
[*] Support bootloader entry on rapid double click of reset button
[ ] Enable bootloader entry on button (or gpio) state
[ ] Enable Status LED
```
After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'canboot.bin' file

1. In the SSH client, select the SFTP option to browse through the files of the CB1.
2. Now go through CB1 directories and enter the path of `home/biqu/CanBoot/out` and download the `canboot.bin` file into the system.
3. Download the software of `STM32CubeProgrammer` into your local system from [here](https://www.st.com/en/development-tools/stm32cubeprog.html) and select the version according to your system.

Note: Preferably download the version of 2.10.0 since the latest version could cause problems in flashing the EBB boards.

4. Connect your EBB36 board to the system using 'USB type C' cable. Make sure the `jumper` is put into the `VUSB` pins of the board.
5. ![image](https://github.com/FracktalWorks/CANBUS-setup-documentation/assets/80109965/41fbffe4-1a53-461b-863e-33a98e318c56)

  After connecting with the USB cable, Press and hold the Boot button, then click the RST button to enter DFU mode.
  
6. ![image](https://github.com/FracktalWorks/CANBUS-setup-documentation/assets/80109965/713f3d30-d547-4822-b0cf-bdd17551bf13)
   
   Click the "Refresh" button in the STM32CubeProgrammer until the Port changes from "No DFU d..." to "USB1", then click "Connect" to connect the chip.
   
7. ![image](https://github.com/FracktalWorks/CANBUS-setup-documentation/assets/80109965/7149ed44-2e97-4534-97ef-f778b1121499)

  1) Select the `download` button on the left column.
  2) Click on `Full chip erase`. It will ask for confirmation, click `OK` after it. Then a message pops up saying the 'mass erase command correctly executed.'   Ignore the rest of the message

8. ![image](https://github.com/FracktalWorks/CANBUS-setup-documentation/assets/80109965/66cf3da3-b79c-460b-a861-8b63b9af2376)

   1) click on the 'Browse' button. Select the path of the `canboot.bin` file wherever it was downloaded.
   2) Click on `Start Programming`. A message should pop up saying 'Download verified successfully'. Another pop up will come up saying 'File download complete'

9. Click on 'Disconnect' on top right and disconnect the board from the system.
   
Make the connections of CANBUS between both EBB36 and the Manta M5P boards. Refer the [Twin Dragon Electronics](https://github.com/FracktalWorks/TwinDragon600-Electronics) to know about it.

After making the connections, check the UUID of the EBB board using the command
```
 ~/CanBoot/scripts/flash_can.py -i can0 -q  
 ```

Note the UUID for future use. Make sure that, it is mentioned as `CanBoot` beside the UUID

Now, we make the Klipper FIrmware for the EBB36 Toolboard.

SSH into the CB1 and enter the following commands:

1. `cd ~/klipper/`
2. `make menuconfig`
```
    Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32G0B1)  --->
    Build CanBoot deployment application (8KiB bootloader)  --->
    Clock Reference (8 MHz crystal)  --->
    Communication interface (CAN bus (on PB0/PB1))  --->
    Application start offset (8KiB offset)  --->
(500000) CAN bus speed
```

After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'klipper.bin' file

In order to flashh the EBB with Klipper, enter the command:

```
 python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u (Put_Your_UUID_Here)
```

In the terminal, enter the command:
```
 ~/CanBoot/scripts/flash_can.py -i can0 -q  
 ```

 or

 ```
 ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```


Make sure that, it is mentioned as `Klipper` beside the UUID.

Note: check the `can0` setup by entering:
```
sudo nano /etc/network/interfaces.d/can0
```

and enter in:

```
allow-hotplug can0
iface can0 can static
      bitrate 500000
      up ifconfig $IFACE txqueuelen 2048
```

Now, in the `printer.cfg` file, enter these UUIDs in different `[mcu]` sections of each motherboard and toolboard. The UUID can be written in the `canbus_uuid:` line

## Setting CAN in BTT Manta M5P + RP2040 based CAN toolboard

The following steps consists of instruction to setup CAN communication in between BTT Manta M5P with BTT CB1 embed onto it and a RP2040 based CAN toolboard with the help of CanBoot Bootloader that is to be installed.

### Initial setup

For the first time use of the BTT CB1, make sure the OS is installed and the Wifi is configured. For first time installation of the OS image, refer to the [Official CB1 GitHub release page](https://github.com/bigtreetech/CB1/releases) or the local [Fracktal CB1 documentation](https://github.com/FracktalWorks/BTT-Manta-CB1-Development-Documentation)

After the above steps, SSH into the CB1, preferably using 'Bitvise SSH Client'. Open the terminal and update the OS using `sudo apt update`. After the update, Follow the steps to install `USB to CAN` communication based klipper firmware in the BTT Manta M5P. Make sure klipper is installed in the CB1.

1. `cd ~/klipper/`
2. `make menuconfig`
```
[\*] Enable extra low level configuration options
 Micro controller Architecture (STMicroelectronics STM32 )
 Processor model (STM32G0B1 )
 Bootloader offset 8 KiB bootloader)
 Clock Reference (8 MHz crystal)
 Communication interface (USB to CAN bus bridge (USB on PA11/PA12))
 CAN bus interface (CAN bus (on PD0/PD1))
 (500000) CAN bus speed
```
After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'klipper.bin' file

Now, enter DFU mode by holding `BOOT` and tapping `RESET`, and then letting go of `BOOT`. Send `lsusb` in the terminal, and then search for a port `0483:df11`. If it is present, it means that the board is in DFU mode.

Flash the firmware by entering in the command:
```
make flash FLASH_DEVICE=0483:df11
```

In order to get the UUID of the board, enter the command in the terminal:
```
 ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
Store this UUID safely in order to use in the klipper's configuration.

Following are the steps to install CanBoot Bootloader in the RP2040 in order to setup CanBus and get the unique UUID. 

SSH into the CB1 and enter the following commands:

1. `cd ~`
2. `git clone https://github.com/Arksine/CanBoot.git`
3. `cd ~/CanBoot`
4. `make menuconfig`

Here, we initially set the CanBoot Bootloader installation in the RP2040 Toolboard. After `make menuconfig`, the following are to be set up:
```
    Micro-controller Architecture (Raspberry Pi RP2040)  --->
    Flash chip (W25Q080 with CLKDIV 2)  --->
    Build CanBoot deployment application (Do not build)  --->
    Communication interface (CAN bus)  --->
(0) CAN RX gpio number
(1) CAN TX gpio number
(250000) CAN bus speed
()  GPIO pins to set on bootloader entry
[*] Support bootloader entry on rapid double click of reset button
[ ] Enable bootloader entry on button (or gpio) state
[*] Enable Status LED
(gpio2) Status LED GPIO Pin
```

After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'canboot.uf2' file

Now, enter bootloader mode by holding `BOOT` and tapping `RESET`, and then letting go of `BOOT` (OR) Remove the USB cable, hold `BOOT` and while holding, reconnect the USB cable. Send `lsusb` in the terminal, and then search for a port `2e8a:0003`. If it is present, it means that the board is in bootloader mode.

Flash the CanBoot firmware by entering in the command:
```
sudo make flash FLASH_DEVICE=2e8a:0003
```

In order to get the UUID of the board, enter the command in the terminal:

```
python3 ~/CanBoot/scripts/flash_can.py -q
```


Note the UUID in order to use it to flash the klipper.uf2 file. Make sure that, it is mentioned as `CanBoot` beside the UUID

Now, we make the Klipper FIrmware for the EBB36 Toolboard.

SSH into the CB1 and enter the following commands:

1. `cd ~/klipper/`
2. `make menuconfig`

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (Raspberry Pi RP2040)  --->
    Bootloader offset (16KiB bootloader)  --->
    Communication interface (CAN bus)  --->
(0) CAN RX gpio number
(1) CAN TX gpio number
(250000) CAN bus speed
()  GPIO pins to set at micro-controller startup
```

After setting, hit `Q` and then hit `Y` when asked to save changes

Enter `make clean`, and then enter `make`. This will compile the 'klipper.uf2' file

In order to flash the RP2040 toolboard with Klipper, enter the command:

```
 python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u (Put_Your_UUID_Here)
```

In the terminal, enter the command:
```
 ~/CanBoot/scripts/flash_can.py -i can0 -q  
 ```

 or

 ```
 ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```


Make sure that, it is mentioned as `Klipper` beside the UUID.

Note: check the `can0` setup by entering:
```
sudo nano /etc/network/interfaces.d/can0
```

and enter in:

```
allow-hotplug can0
iface can0 can static
      bitrate 500000
      up ifconfig $IFACE txqueuelen 2048
```

Now, in the `printer.cfg` file, enter these UUIDs in different `[mcu]` sections of each motherboard and toolboard. The UUID can be written in the `canbus_uuid:` line
