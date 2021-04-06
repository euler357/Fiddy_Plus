[![uBld Electronics, LLC Logo](/images/ublditlogo_color_blue.png)](https://ubld.it)

# Fiddy_Plus
### Docs for the uBld.it Fiddy Plus FT2232H Board
We designed this because we got tired of getting clone chips and unnecessarily large breakout boards that were difficult to connect.  This does exactly what it is supposed to with a Genuine FTDI FT2232H high-speed IC.  All of the I/O is clearly labelled for SPI, JTAG, and UART use.  An EEPROM is on-board that can be used for setting the FT2232H into more unusual modes.
[![Fiddy Plus Front Side](/Docs/Fiddy_Plus_Render_Front_RevD.png)
![Fiddy Plus Back Side](/Docs/Fiddy_Plus_Render_Back_RevD.png)](https://www.amazon.com/dp/B091MDFHH3)

[This board is available here](https://www.amazon.com/dp/B091MDFHH3)

## Specifications
* [Genuine FTDI FT2232H](http://www.ftdichip.com/Support/Documents/DataSheets/ICs/DS_FT2232H.pdf)
  * Not a clone - no clone related driver or signal issues
* SPI Interface
* JTAG Interface
* UART Interface

## Setup
In Linux (where most users will be working with this board), you need to add a udev rule to allow all users (or a specific group) to access the board unless you always want to run with sudo or as root.

The 99-fiddy.rules file should be copied to /etc/udev/rules.d/ directory then either reboot or run:
~~~
sudo udevadm control --reload-rules
sudo udevadm trigger
~~~

## SPI Example

To dump a SPI Flash Image, connect the Fiddy Plus as follows:

| Fiddy Plus  | SPI Flash Chip   |
| ----------- | ----------- |
| GND         | GND         |
| CS          | CS          |
| MISO        | MISO        |
| MOSI        | MOSI        |
| CLK         | CLK         |
| 3.3V        | 3.3V (Optional) |


~~~
sudo flashrom -p ft2232_spi:type=2232H,port=B
~~~
or you may need to slow it down to get a stable read
~~~
sudo flashrom -p ft2232_spi:type=2232H,port=B,divisor=4
~~~

It should look like this:
| SPI Write | SPI Read | SPI Diff |
|----------------|------------|------------|
| ![SPI Write Screenshot](/images/SPI_Write.jpg) | ![SPI Read Screenshot](/images/SPI_Read.jpg) | ![SPI Read-Write-Compare Screenshot](/images/SPI_Read_Write_Compare.jpg)  

## JTAG Example

To get the openocd debugger running with a connection to a JTAG ARM processor, you need to connect the JTAG pins and run openocde with the appropriate options.  Here is an example using the STM32F103C8T6 on a "Blue Pill" board.

| Fiddy Plus  | Blue Pill   |
| ----------- | ----------- |
| GND         | GND         |
| ~SRST       | R (Reset)   |
| ~TRST       | B4          |
| TMS         | IO (SWD Header) |
| TDO         | B3          |
| TDI         | A15         |
| TCK         | CLK (SWD Header) |
| 3.3V        | 3.3V (SWD Header) |

You will then first run the openocd server in one session:
~~~
openocd -f fiddy-jtag.cfg -f target/stm32f1x.cfg
~~~
or (if you didn't install the udev rule) 
~~~
sudo openocd -f fiddy-jtag.cfg -f target/stm32f1x.cfg
~~~

The "target/stm32f1x.cfg" defines the architecture and should come with openocd

Then, in another session:
~~~
telnet localhost 4444 
~~~

It should look like this:
| openocd server | openocd telnet session |
|----------------|------------------------|
| ![openocd server screenshot](/images/fiddy_plus_blue_pill_jtag_openocd_screenshot.png) |![openocd telnet session screenshot](/images/fiddy_plus_blue_pill_jtag_openocd_screenshot2.png) 

## UART Example

The first UART port on the Fiddy Plus's FT2232H is connected to the UART pins and associated TX / RX LEDs.  In Linux, the screen command can be used to access the port on /dev/ttyUSB0 as follows with a 115,200 bps data rate.  (sudo is necessary unless /dev/ttyUSB0 permissions are set to allow access by the current user)

~~~
sudo screen /dev/ttyUSB0 115200
~~~

![Fiddy Plus UART screenshot](/images/Fiddy_plus_uart_screen_example.png)

## Advanced Modes
FTDI publishes a lot of information about this chip:

* [FTDI FT2232HL Main Page](https://ftdichip.com/products/ft2232hl/)
* [AN 113 FTDI FT2232H Hi Speed USB To I2C Example](https://ftdichip.com/wp-content/uploads/2020/08/AN_113_FTDI_Hi_Speed_USB_To_I2C_Example.pdf)
* [AN 114 FTDI FT2232H Hi Speed USB To SPI Example](https://ftdichip.com/wp-content/uploads/2020/08/AN_114_FTDI_Hi_Speed_USB_To_SPI_Example.pdf)
* [AN 124 User Guide for FTDI FT_PROG Utility](https://ftdichip.com/wp-content/uploads/2020/08/AN_124_User_Guide_For_FT_PROG.pdf)
* [AN 129 Interfacing FT2232H Hi-Speed Devices to a JTAG TAP](https://ftdichip.com/wp-content/uploads/2020/08/AN_129_FTDI_Hi_Speed_USB_To_JTAG_Example.pdf)
* [AN 130 FT2232H Used In An FT245 Style Synchronous FIFO Mode](https://ftdichip.com/wp-content/uploads/2020/08/AN_130_FT2232H_Used_In_FT245-Synchronous-FIFO-Mode.pdf)

The FT2232H can be used for I2C, SUMP logic analyzer, and many other interfaces.
