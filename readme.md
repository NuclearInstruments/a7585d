
# Using Arduino to control  SiPM Temperature Compensated HV Module via I2C

## Features
-  20-85V Output Voltage
- 10mA Output Current
- 1mV Output Voltage step
- Less than 300uV rms noise
- User Selectable Digital / Analog output voltage control
- Automatic temperature feedback on the output voltage
- Multi device support using I2C
- Arduino library designed to manage multiple devices
- Evaluation kit include
	 - A7585DU HV with USB and I2C interface
	- Arudino Uno
	- Display with Keypad
	- Bread Board
	- Cables
	- 12v Power Supply

- Fully working examples designed to control the module with keypad and display or monitor multiple devices using serial port
- Module compatible with ZEUS software for stand alone usage


## Description

The NIPM-12 SiPM Power Module is a compact and integrated solution to provide stable and noiseless power supply for single and array / matrix SiPM detectors.

High resolution Output Voltage and Output Current measurements enable the NIPM-12 to be used for I-V detector characterization.

Digital (UART, I2C and USB with adapter) and analog control interface are runtime selectable by a single pin or a digital command.

The module integrates a temperature HV loop that regulates the SiPM output voltage as a programmable function of the SiPM temperature coefficient.

The Arduino evaluation kit is a ready to use solution to discovery the capabilities of the A7585D integrated in a custom OEM solution. The A7585D library provide an high level abstraction of all commands supported by the module. It allows, for example, with a few lines of code to configure the module and readback voltage and current monitor values

## Pinout of the module

![pinout of the moduke](https://github.com/NuclearInstruments/a7585d/blob/master/images/img2.png?raw=true)

## Library installation

In order to use the examples of this application note it is necessary to install A7585D library and LiquidCrystals library. A7585D library is a Nuclear Instruments library developed to interface with A7585D with an high level class while LiquidCrystals library is an opensource library for LCD display control.

To install the libraries open Arduino software, Sketch menu -> Add .ZIP library.
Select the two library zip file from the repository


## Library usage

The A7585D can be controlled via UART, USB and I2C. While UART and USB are suited for PC control the I2C is the best solution to interface multiple modules with module with a single controller like an Arduino, a Raspberry PI or another microcontroller. I2C bus support device addressing: up to 120 A7585D/DU can be controlled by a single Arduino processor without any communication issue.  
In application where a large number of channel is required, this design can be the ultimate solution in order to control and monitor multiple devices.

The I2C bus use just two wires to interface a master device with multiple slave.  The master device is the Arduino Uno while the slave devices are the A7585DU modules.

The two wires are:

�SDA: data wire. Bidirectional, transport data between master and slaves. SDA is the green wire in the scheme

�SCL: clock wire. Generated by the master. SCL is the yellow wire in the scheme

Each device on the I2C has an address that must be unique. Read carefully the module address section if you need to operate with multiple A7585DU

The A7585DU must operate with a supply voltage greater than 6V. It is indeed impossible to connect the Vin pin to the Arduino 5V. An external power supply is required. Power Supply can be externally provided with the 12v power supply included in the kit through the Arduino power supply connector.

The 12V wire (violet) connect the VIN pin of the Arduino to the input voltage of the A7585DU

The A7585DU require an additional power supply of 5V (red) in order to power the I/O buffer for the I2C interface. This buffer allows to interface the A7585DU on the I2C with any chip operating with a voltage between 1.8V and 5.5V.

The black wire connects the Arduino ground to the HV module ground.

This is the minimal configuration to control with I2C the A7585DU module
![Connection to Arduino](https://github.com/NuclearInstruments/a7585d/blob/master/images/img3.png?raw=true)

## Library functions list

In the following section a full description of all function of the A7585A library is provided

    bool Init(int  IICAddress)
IN
IICAddress: address of the A7585A in 7 bit format

OUT
bool: true if a valid module has been identified

Initialize the library providing the I2C module Address. The default module address is 0x70

---------------------------------------
    void  Set_MaxV(float v)
IN
v: HV output compliance voltage

Set the hv output maximum output voltage (protection)


	void  Set_MaxI(float v)
IN
v: HV maximum output currect

Set the hv output maximum output current. The module is switched off if current exceeds the limit

---------------------------------------

	void  Set_Enable(bool v)
IN

v: Enable the output

Enable the HV output. If the HV fails due to overcurrent Set_Enable false and true to restore HV

---------------------------------------

	void  Set_RampVs(float v)

IN

v: Ramp speed in V/s

Set the HV output ramp speed in V/S

---------------------------------------

	void  Set_TemperatureSensor(HVTemperatureSensors  SensorModel,  float term_m2,  float  term_m,  float  term_q  )

IN

SensorModel: Select between standard thermometer already calibrated and custom model
term_m2: quadratic fitting parameter between temperature and ref voltage
term_m: linear fitting parameter between temperature and ref voltage
term_q: offset

Configure the thermometer connected to the A7585A in order to operate in temperature compensated mode

---------------------------------------
	void  Set_Filter(float  alfa_v,  float  alfa_i,  float  alfa_t)

IN

alfa_v: out voltage monitor filter coefficient
alfa_i: out current monitor filter coefficient
alfa_t: temperature monitor filter coefficient

Set the filter coefficient applied to the data monitor filter [0�0.9999]. A value closer to 1 means an higher number of averages and an higher resolution while a value closer to 0 means no filtering and faster response

---------------------------------------

	void  Set_SiPM_Tcoef  (float  tcomp)

IN
tcomp: Thermal coefficient compensation

This is the coefficient provided by SiPM manufacturer expressed is V/�C to compensate the gain variation due to the temperature. A typically value is -34mV/�C

---------------------------------------

	void  EmergencyOff()

Switch off HV without ramp

---------------------------------------

	void SetI0()

Compensate the current measured zeroing the measurement and removing biasing

---------------------------------------

	void  Set_DigitalFeedback(bool on)

IN
on: Enable the internal PID controller

The internal PID controller use the voltage value read by the feedback ADC in order to compensate small static error in the feedforward output setpoint

---------------------------------------

	void  Set_IIC_baseaddress(uint8_t  ba)

IN
ba: new base address for the A7585D

Change the base address of the A7585D. The module address will be the ba added to the status of the address pins.

---------------------------------------

	uint8_t  GetDigitalPinStatus()

OUT
uint8_t: binary encoded pin status

Read the status of all digital I/O of the module

---------------------------------------

	float  GetVin()

OUT
float: power supply voltage

Read the power supply voltage

---------------------------------------

	float  GetVout()

OUT
float: Output voltage

Read the power HV out voltage

---------------------------------------

	float  GetIout()

OUT
float: Output current

Read the power HV output current

---------------------------------------

	float  GetVref()

OUT
float: Reference pin voltage

Read the voltage on the reference (analog control only) pin

---------------------------------------

	float  GetTref()

OUT
float: Temperature of the SiPM

Read the temperature of the SiPM through the temperature sensor connected to the Vref pin

---------------------------------------

	float  GetVtarget()

OUT
float: Current voltage target

Read current output voltage target

---------------------------------------

	float  GetVtargetSP()

OUT
float: Current output voltage set on the module

Read the current DAC set point in Volt

---------------------------------------

	float  GetVcorrection()

OUT
float: correction voltage applied to the target

Read the correction voltage applied to the target in order to compensate the temperature

---------------------------------------

	bool  GetVCompliance()

OUT
bool: flag indicating that the module output voltage is limited by compliance

When true, the output voltage is clamped to the compliance voltage

---------------------------------------

	bool  GetICompliance()

OUT
bool: flag indicating a overcurrent protection

When true, the module has been switched off due to over current. To restore the module disable and enable the output.

---------------------------------------

	uint8_t  GetProductCode()

OUT
uint8_t: product code of the device

Return the product code of the device

---------------------------------------

	uint8_t  GetFWVer()

OUT
uint8_t: firmware version

Return the firmware version of the device

---------------------------------------

	uint8_t  GetHWVer()

OUT
uint8_t : hardware version

Return the hardware version of the device

---------------------------------------

	uint32_t  GetSerialNumber()

OUT

uint32_t: serial number of the device
Return the serial number of the device

---------------------------------------

	bool  GetHVOn()

OUT
bool: hv status

Read the current HV status (true is on)

---------------------------------------

	bool  GetConnectionStatus()

OUT
bool: flag indicating that Arduino is connected to the module

When true, the Arduino is connected to the module

---------------------------------------

	void  StoreCfgOnFlash  ()

Save configuration on flash




## Code Examples

Connect the module A7585D/U  to the Arduino
![Connection to Arduino](https://github.com/NuclearInstruments/a7585d/blob/master/images/img3.png?raw=true)


```


#include <A7585lib.h>
#include <Wire.h>
#include <stdio.h>
#include <stdlib.h>

#define DEV_ADDRESS 0x70

A7585 A7585_dev;

// Initialize device
void  InitNIPM(int  device_address)
{
	A7585_dev.Init(device_address);
	A7585_dev.Set_Mode(0);				//SET MODE DIGITAL
	A7585_dev.Set_MaxV(80);				//SET COMPLIANCE 80V
	A7585_dev.Set_MaxI(10);				//SET MAXIMUM CURRENT 10mA
	A7585_dev.Set_RampVs(25);			//SET RAMP UP SPEED 25V/s
}

void setup()  {
	Serial.begin(115200);
	Wire.begin();
	InitNIPM(DEV_ADDRESS);
	
	if  (A7585_dev.GetConnectionStatus())
		Serial.println("Probe connection successful");
	else
		Serial.println("Error  connecting device ");
	
	A7585_dev.Set_V(50);				//SET VOUT=50V		
	A7585_dev.Set_Enable(true);			//ENABLE HV
}

void loop()  {
	cvout  = A7585_dev.GetVout();		//READ VOLTAGE
	ciout  = A7585_dev.GetIout();		//READ CURRENT
	Serial.print(" V: ");  Serial.print(cvout);  Serial.print(" I: ");  Serial.println(ciout);
	delay(1000);
}
```