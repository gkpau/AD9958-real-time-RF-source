## Introduction
This **AD9958 real time RF source** repository consists of a set of Python and C++ libraries for real-time control of a AD9958 direct digital synthesizer (DDS). The sythesizer has been successfully implemented in the group of Prof. Morgan Mitchell ([ICFO](www.ICFO.eu)) for RF/uW state preparation and manipulation of a Rb 87 Bose-Einstein Condensate.

Python API documentation available under https://gkpau.github.io/AD9958-real-time-RF-source/.

#### Operating principle
The operation of this real-time RF source can be understood in 3 effective layers. The first one is a Python API, designed to send commands to you RF synthesizer in a coprehensive and user-friendly way. Behind the scenes, this layer converts the commands into register adresses and register values of the AD9958. The sets of register addresses and values are forwarded (via serial communication) to the second layer: the chipKit Max 32. This microcontroller builds up a *functions stack* which saves the requested commands/tasks and executes them sequentially on demand. Note that the sequential execution of the commands is entirely performed on the side of the microcontroller (exact timing) and can be triggered by an external TTL trigger. During most of the tasks, the microcontroller communicates over an SPI bus with the AD9958, setting its internal registers. The AD9958 itself is the third effective layer, which generates the RF signal according to its internal registers and profile pins.

#### Highlighted capabilities
* Separate control of amplitude, phase and frequency of channel 0 (ch0) and channel 1 (ch1) of the AD9958.
* 2,4,8,16 bit modulation mode.
* Sweep mode with automatic ramp optimization.
* Triggered execution (<0.1 us time jitter).
* Progammable internal delay (~62.5ns step size, ~125ns minimum delay time).
* Up to 1000 programmble instructions.




In the following I will give closer details on the hardware needed, internal connection, programming of the Chikit Max 32 microcontroller and the Python API.

## Hardware
* **AD9958 eval board**. Dual channel DDS with a DAC sampling rate of upt to 500 MHz. For further details please check the documentation on the [AD9958](https://www.analog.com/en/products/ad9958.html) and the [evaluation board](https://www.analog.com/en/design-center/evaluation-hardware-and-software/evaluation-boards-kits/eval-ad9958.html).

* **chipKIT Max32**. PIC32 board with a 80MHz on board clock. This microcontroller recieves the requested RF setting over its serial port and sets the corresponding register on the AD9958 via an SPI bus. For detailed information please refer to the [chipKIT Max 32 reference manual](https://reference.digilentinc.com/chipkit_max32/refmanual),  the [PIC32MX5XX/6XX/7XX data sheet](http://ww1.microchip.com/downloads/en/DeviceDoc/60001156J.pdf) and the more complete [PIC32MX Familiy Reference Manual](http://hades.mech.northwestern.edu/images/2/21/61132B_PIC32ReferenceManual.pdf).

* **Reference RF clock** for operating the DDS. In order to achieve the maximum sampling rate of the AD9958, a minimum clock frequency of 25 MHz is required. Please refer to the  AD9958 eval board documentation for the max/min ratings in terms of power and frequency.

* **Linear volatge regulators** for supplying the 3.3V and 1.8V required by the Ad9958 eval board. A good example of a LM317T variable voltage regulator diagram can be found [here](https://www.electronics-tutorials.ws/blog/variable-voltage-power-supply.html).

* **Computer** with a Python 2.7 distribution installed (my personal preference is to directly install the appropiate [Anaconda](https://www.anaconda.com/download/) distribution). Please note that the current project was developed under Python 2.7, the compatibility with Python 3 has not been tested yet. In order to programm the chipKIT Max32, the Arduino IDE and an additional board manager have to be installed. The whole procedure is well explained [here](https://chipkit.net/wiki/index.php?title=ChipKIT_core).

* **RF Transformer (optional)**. By default, the two DAC outputs of the AD9958 eval board are decoupled by using [ADTT1-1](https://www.minicircuits.com/WebStore/dashboard.html?model=ADTT1-1) RF transformers, which operate from 0.3-300 MHz. For my application lower RF frequencies were required and I replaced the transformers by two [ADT1-6T+](https://www.minicircuits.com/WebStore/dashboard.html?model=ADT1-6T%2B) (dynamic range of 0.03-125 MHz).

#### Setting up the AD9958 eval board
Please follow the isntructions in the above mentioned [documentation] for:
* Setting the **jumpers** for *External Control*.
* Setting the **jumper** for *External REF_CLK*.
* Proving the 1.8V and 3.3V bias voltages on the required connectors.

#### Connections between AD9958 eval board chipKit Max32 
In the following table the wiring between the microcontroller and the Manual I/0 Control Header of the AD9958 eval board are described. Please note that although the Manual I/O Control Headers have two row of pins available only the lower one is connected to the AD9958.




|AD9958 eval header |Chipkit Max32 Pin |Description       |
| ------------- | ------------- | ------------- |
| P0 | 30 |Profile pin 0|
| P1 | 31 |Profile pin 1|
| P2 | 32 |Profile pin 2|
| P3 | 33 |Profile pin 3|
| IO_UPDATE | 34 |IO Update|
| CSB | 35 |Chip select (SPI) |
| SCLK | 52 |Serial clock (SPI)|
| RESET | 36 |Reset|
| PWR_DWN | 37 |Power down|
| SDIO_3\* | 38 |Sync I/O |
| SDIO_2\* | 50 | MISO (SPI) |
| SDIO_1\* | 39 | Not used |
| SDIO_0\* | 39 | MOSI (SPI) |
| NC\*\* | 18 | Trigger in |
| NC \*\* | 19 |Trigger out |


\* The current project uses the *Single-Bit Serial 3-Wire mode* of the AD9958.

\*\*Not connected.

#### Triggers
Two *triggers* have been implemented onto the chipKit Max32:
* **Trigger in**. The execution of the *functions stack* can be held until a rising edge is detected on the corresponding PIN 18. Please make sure to adjust the voltage level of your trigger input to 3.3V.

* **Trigger out**. Used for monitoring purposes (e.g. triggering the adquisition of your oscilloscope) or possible external RF switches.

## Setting up the chipKit Max32
The firmware required for the chipKit Max32 microcontroller can be found in the *DriverChipkit/AD9958Driver* folder:
* **AD9958Driver.ino** -> Main code running on the ChipKit Max32. Contains the functions for the construction and execution of the *functions stack*.
* **AD9958_definitions.h** -> Basic definitions of functions an types used in *AD9958Driver.ino*.
* **SPI_simple library** -> Basic library for fast execution of SPI commands (avoids unused overhead).
* **SerialCommand library** -> Serial command interpreter. Developed by S. Rado and S. Cogswell. 

For setting up the chipKit Max32 please compile and upload the *AD9958Driver.ino* file. Make shure you have installed the IDE and the additional board manager (explained above). 


## Python API
The Python APi is available by adding the *AD9958* folder into the working directory of your Python interpreter. A full documentation is available under https://gkpau.github.io/AD9958-real-time-RF-source/.

Please refer to the provided **Examples** for a detailed implementation of the different functionalities of the AD9958. Note that when downloading the current repository the set of examples are ready to use from inside the *Examples* folder.

## Final remark
The current repository is MIT licensed, feel free to use it, improrve it and give me feedback on paugomezkabelka@gmail.com.  



