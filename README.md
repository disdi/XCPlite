
# XCPlite V6 Demo

Copyright 2024 Vector Informatik GmbH

XCPlite is a lightweight demo implementation of the ASAM XCP V1.4 standard protocol for measurement and calibration of electronic control units. 
The demo implementation uses Ethernet UDP or TCP communication on POSIX based Systems. 
XCPlite is provided to test and demonstrate calibration tools such as CANape or any other XCP client implementation. 
It demonstrates some capabilities of XCP and may serve as a base for individually customized implementations. 

New to XCP? Checkout Vector�s XCP Reference Book here: https://www.vector.com/int/en/know-how/protocols/xcp-measurement-and-calibration-protocol/xcp-book# or visit the Virtual VectorAcedemy for an E-Learning on XCP: https://elearning.vector.com/ 

A list of restrictions compared to Vectors free XCPbasic or commercial XCPprof may be found in the source file xcpLite.c.
XCPbasic is an implementation optimized for smaller Microcontrollers and CAN as Transport-Layer.
XCPprof is a product in Vectors AUTOSAR MICROSAR and CANbedded product portfolio.   

XCPlite
- Supports TCP or UDP with jumbo frames. 
- Is thread safe, has minimal thread lock and single copy data acquisition. 
- Compiles as C or C++. 
- Has no dependencies but includes some boilerplate code to abstract socket communication and clock
- Achieves up to 100 MByte/s throughput on a Raspberry Pi 4. 

XCPlite has been testet on CANFD, there is some experimental code included, but there is no example target to showcase this.
XCPlite is not recomended for CAN.

No manual A2L creation (ASAP2 ECU description) is required for XCPlite. 
An A2L with a reduced featureset may be generated through code instrumentation during runtime and can be automatically uploaded by XCP. 


## Included code examples (Build Targets):  

XCPlite:
  Getting started with a simple demo in C with minimum code and features. Shows the basics how to integrate XCP in existing applications. Compiles as C.   

## Code instrumentation for measurement events:

Very simple code instrumentation needed for event triggering and data copy, event definition and data object definition. 

Example: 

### Definition:

Define a global variable which should be acquired and visualized in realtime by the measurement and calibration tool

```
  double channel1; 
```

### A2L generation:

A2L is an ASCII file format (ASAM standard) to describe ECU internal measurement and calibration values.
With XCPlite, the A2L file may be generated during runtime at startup of the application:

```
  A2lCreateEvent("ECU"); // Create a new event with name "ECU""
  A2lSetEvent("ECU"); // Set event "ECU" to be associated to following measurement value definitions
  A2lCreatePhysMeasurement(channel1, 2.0, 1.0, "Volt", "Demo floating point signal"); // Create a measurement signal "channel1" with linear conversion rule (factor,offset) and unit "Volt"
```


### Measurement data acquisition event:

A measurement event is a trigger for measurement data acquisition somewhere in the code. Multiples measurement objects such as channel1, even complexer objects like structs and instances can be associated to the event. This is done during runtime in the GUI of the measurement and calibration tool. An event will be precicly timestamped with ns resolution, timestamps may obtained from PTP synchronized clocks and the data attached to it, is garantueed to be consistent. The blocking duration of the XcpEvent function is as low as possible:

```
  channel1 += 0.6;
  XcpEvent(1); // Trigger event number 1, attach a timestamp and copy measurement data
```

This is a screenshot of the tool GUI.

![CANape](Screenshot.png)



## Configuration options:

All settings and parameters for the XCP protocol and transport layer are located in xcp_cfg.h and xcptl_cfg.h. 
Compile options for the different demo targets are located in main_cfg.h. 


## Notes:

- Specify the IP addr to bind (in main_cfg.h or on the command line (-bind)), if there are multiple Ethernet adapters. Otherwise the IP address of the Ethernet adapter found first, will be written to A2L file. 

- If A2L generation and upload is disabled, make sure CANape (or any other tool) is using an up to date A2L file with correct memory addresses and data types.  
The A2L from ELF updater in CANape may be activated to achieve this.  
Be aware that XCP uses direct memory access, wrong addresses may lead to access fault or even worse to corrupt data.  
You may want to enable EPK check, to make sure the A2L description matches the ECU software.  
When using MS Visual Studio, generate Debug Information optimized for sharing and publishing (/DEBUG:FULL)

- If A2L upload is enabled, you may need to set the IP address manually once. 
When connect is refused in CANape, press the flashing update icon in the statusbar.

- For the A2L Updater or CANapes automatic A2L address update, use Linker Map Type ELF extended for Linux a.out format or PDB for Microsoft .exe

- 64 bit builds needs all objects located within one 4 GByte data segment. 
Note that XCP addresses are 32 Bit plus 8 Bit extension. 
The conversion methods from pointer to A2l/XCP address and vice versa, are in xcpAppl.c and maybe changed for specific needs. xcpLite.c does not make assumptions on addresses. 
The only exception is during measurement, where XcpEvent creates pointers by adding the XCP/A2L address to ApplXcpGetBaseAddr(). 
To save space, the 32 Bit addresses, not 64 Bit pointers are stored in the DAQ lists. 
During measurement setup, ApplXcpGetPointer is called once to check for validity of the XCP/A2L address conversion. 
  
- Multicast time synchronisation (GET_DAQ_CLOCK_MULTICAST) is enabled in CANape by default. 
When measurement does not start, it is most probably a problem with multicast reception. 
Multicast provides no benefit with single clients or with PTP time synchronized clients and is therefore just unnessesary effort. 
Turn Multicast off in device/protocol/event/TIME_CORRELATION_GETDAQCLOCK by changing the option from "multicast" to "extended response"

## Build

### Linux

#### Install development tooling
for Linux:
``` sh
sudo apt-get install cmake g++ clang ninja-build
```

#### Build

``` sh
cd XCPlite
cmake -DCMAKE_BUILD_TYPE=Release -S . -B build  
cd build
make
./XCPlite.out

```