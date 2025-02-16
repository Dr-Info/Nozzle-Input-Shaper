# Nozzle-Input-Shaper
FYSETC - Nozzle Input Shaper (NIS)

## IMPORTANT NOTICE
**You should NEVER keep the NIS Attached while the Nozzle is being heated, ensure you remove after measurements have been taken**

## Firmware
The Nozzle Input Shaper does come pre-flashed with Klipper. It is always best practice to not have missmatched versions of your Klipper Firmware, so this guide will explain how to flash your NIS with Klipper. 

### Klipper
#### Compile Klipper firmware for the Nozzle Input Shaper

1. Connect to your Klipper host computer via ssh. 
2. cd to the Klipper directory 
```bash
cd ~/klipper
```
3. Run make clean 
```bash
make clean
```
4. Open menuconfig 
```bash
make menuconfig
```
5. Set the following settings
   - [*] Enable extra low-level configuration options
   - Micro-controller Architecture (STMicroelectronics STM32)
   - Processor model (STM32F042)
   - Bootloader offset (No bootloader)
   - Clock Reference (Internal clock)
   - Communication interface (USB (on PA09/PA10))
   - Optional features (to reduce code size) --->
       - [*] Support GPIO "bit-banging" devices
       - [_] Support LCD devices
       - [*] Support external sensor devices
       - [_] Support lis2dw 3-axis accelerometer
       - [_] Support software based I2C "bit-banging"
       - [*] Support software based SPI "bit-banging"

![menuconfig](3D/NIS_menuconfig.JPG)
![menuconfig](3D/NIS_optional_features.JPG)

6. To quit (press Q) and to save the configuration (press Y)
7. Run Make to compile the firmware
```bash
make
```
#### Flash Klipper over USB
1. Ensure that you are still connected via SSH, if not, reconnect to your Klipper host via SSH. 
2. Connect the board to the host Raspberry Pi via USB, while pluggin the board in, you must ensure the BOOT button is pressed down.
3. Run the following command
```bash
lsusb
```
4. It should produce a result showing a STM Device in DFU Mode
   
   Example
   
   `Bus 001 Device 038: ID 0483:df11 STMicroelectronics STM Device in DFU Mode`
   
5. Run the following command to Flash your NIS, replace the xxxx:yyyy with the ID from the previous step.
```bash
make flash FLASH_DEVICE=xxxx:yyyy
```
Example
```bash
make flash FLASH_DEVICE=0483:df11
```
   - You may see what appears to be an "error" after flashing your board.
   - As long as you see the File downloded successfully text you are good to proceed.
6. After it has finished flashing, run the following command again. As long as the device is not showing it is in DFU Mode like in step 3, you are good to move onto the next step.
7. Run the following command to find the devices serial port name
```bash
ls /dev/serial/by-id/
```
It should return a device beginning with `/dev/serial/by-id/usb-Klipper_stm32f042x6` take a note of this, you will need it later on. 

## Configuration 

### Klipper
1. Navigate to your Mainsail / Fluidd Page on the printer you have added your NIS too.
2. Navigate to edit your printer.cfg file.
3. Near the top of the page add the following to your config
```yaml
[mcu NIS]
# Obtain definition by "ls -l /dev/serial/by-id/" then unplug to verify
serial: /dev/serial/by-id/<your serial id>

[adxl345]
cs_pin: NIS:PA4
spi_software_sclk_pin: NIS:PA5
spi_software_mosi_pin: NIS:PA7
spi_software_miso_pin: NIS:PA6

[resonance_tester]
accel_chip: adxl345
probe_points:
    100,100,20 # an example - set this to the center of your BED.
```
4. Save your config and restart firmware
5. Follow the [Klipper documentation](https://www.klipper3d.org/Measuring_Resonances.html)  on how to Measure Resonance on Klipper.
6. Once your results are in, remove the Nozzle Input Shaper from the Nozzle and comment out or remove the above config from your Printer.cfg

## Updating

### Klipper
1. Update all software in UI Dashboard
2. cd to the Klipper directory 
```bash
cd ~/klipper
```
3. Run make clean 
```bash
make clean
```
4. Open menuconfig 
```bash
make menuconfig
```
5. Set the following settings
   - [*] Enable extra low-level configuration options
   - Micro-controller Architecture (STMicroelectronics STM32)
   - Processor model (STM32F042)
   - Bootloader offset (No bootloader)
   - Clock Reference (Internal clock)
   - Communication interface (USB (on PA09/PA10))
   - Optional features (to reduce code size) --->
       - [*] Support GPIO "bit-banging" devices
       - [_] Support LCD devices
       - [*] Support external sensor devices
       - [_] Support lis2dw 3-axis accelerometer
       - [_] Support software based I2C "bit-banging"
       - [*] Support software based SPI "bit-banging"

![menuconfig](3D/NIS_menuconfig.JPG)
![menuconfig](3D/NIS_optional_features.JPG)

6. To quit (press Q) and to save the configuration (press Y)
7. Flash with command below (INPUT YOUR ID ACCORDINGLY). You can get it with ```ls /dev/serial/by-id/```

```bash
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/<your id>
sudo service klipper start
```
8. On restart, klipper should show most current version in UI Dashboard.
