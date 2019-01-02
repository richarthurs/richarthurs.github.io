---
title: "CubeSat Onboard Computer Design"
author: Richard
tags: 
  - Projects
image: ../img/post-headers/obc1-post-header.jpg
date: "2018-06-06"
draft: false
---
Over the past year and a half, I have been working on the SFU Satellite Design Team as the computing subsystem lead. The team and I have been developing a low-cost, reliable, custom onboard computer (OBC) to meet the current and future mission requirements. This post features quite a few details about the project.

The original mission the OBC was designed for was calibration of the CHIME experiment, which was SFUSat’s first CubeSat mission developed during the Canadian Satellite Design Challenge.

##Features
The SFUSat OBC is an inexpensive command and data handling system designed to be flexible across CubeSat missions. It features:

  - Automotive and extended temperature range hardware wherever possible
  - Cortex R4 processor with lockstep architecture
  - Extensive self-tests, error handling, SECDED
  - RTC with minimum 7-day supercapacitor backup
  - Nonvolatile storage of mission data
  - Optional triple-redundant data storage
  - Flexible bus architecture with PC/104 form factor
  - 2x SPI, I2C, ADC pins broken out to bus connector
  - Multiple interrupt-capable GPIO on bus connector
  - Simple-to-integrate 3.3 V power supply requirement
  - External USB power for development purposes
  - External watchdog timer
  - Current monitoring and auto reset for SEL protection
  - Onboard temperature sensor
  - FreeRTOS-based software stack ensuring priority and multi-tasking
  - Immediate and time-tagged commands
  - Automatic logging and timestamp of any data with friendly API
  - Minimum 7-day file history capacity
  - File downlink capability

The TMS570 microcontroller used in the design has flight heritage with Robonaut, several satellites, and was confirmed to function correctly to 2.5 KRad by SFUSat. [1]

## File Handling & Telemetry
The satellite is intended to be frequently transmitting its most up-to-date telemetry packet, called standard telemetry. Telemetry points are also saved to a file onboard the satellite. One file is created per sensor, per day, and after 7 days have passed (or no room remains), the oldest set of log files are deleted. Log files can be downloaded at any time by a remote command. Standard telemetry sending can be suspended for a defined period of time for power saving purposes.

 
The filesystem chosen for the project is SPIFFS, an open-source and lightweight filesystem designed for SPI NOR flash devices. It proved simple to integrate and performed well in all of our tests.

The ring buffer style file architecture was simple and worked well overall. I developed an easy-to-use API to log data to files, where arbitrary text could be formatted and written into a file with a single function call using printf-style format specifiers. All writes to files were automatically timestamped. Developing easy-to-use APIs is a common theme of this project, and helps immensely when getting new members up to speed.

 

Files are named with a simple prefix-suffix methodology. The prefix ‘a’, ‘b’, and so on, represents the current “day” or time step. Over the course of a week, files with prefixes ‘a’ to ‘g’ will be created, and once ‘g’ files are finished, the ‘a’ will be deleted and replaced with new ‘a’-prefixed files. The suffix is another letter, ‘A’, corresponding to the file’s function, such as OBC temperature logs or general error logs. This naming methodology was chosen to make finding files quick, and to exploit the batch delete feature of SPIFFS.

We also have a “flag file” capability, which is used for storing and retrieving short, non-volatile flags from external flash memory. At startup, the system looks for the presence of this file, creates it if it doesn’t exist (and populates sensible defaults), or uses the existing flags.

 

## Self-Tests
Upon startup, the OBC executes self tests to verify core functionality and connection to peripherals. Self-tested features include:

- CPU and all peripherals
- Battery management system
- External non-volatile memory

Since the TMS570 self-tests wipe RAM, we save the results of self tests into a spare CAN data register that is not used by our application. The data in this register survives the self test, allowing us to determine which (if any) self tests have failed.

 

## OBC Epoch
The OBC epoch is the real-time clock (RTC) based timestamp used throughout the system. Upon deployment, once the satellite has enough power to turn on the OBC, the real-time clock will start ticking from time 0. With a large super capacitor backup on the RTC, it is able to keep time for weeks without main system power. This capability can be used to determine how long the satellite had lost power, and it is used to timestamp files and coordinate antenna deployments.

 

The RTC chip is the Abracon AB-RTCMC-EA09-S3-D-B-T. It was chosen for low power, wide temperature range, and temperature compensated crystal features. It also has an alarm feature connected to a GPIO, which can be used to wake up the OBC from extended deep sleep periods. The time is represented in a human-readable date format which is converted to an absolute zero-based seconds format, with time 0 being the time the satellite first powered on.

 

## Command System
The OBC software features a command system designed to be flexible. Commands have a main command, optional subcommand, and optional arguments (represented as hex data). The command system is linked to the UART for tethered development, and it’s linked to the radio so commands can be sent wirelessly.

Some of the commands include:

    - state get, set, and previous state
    - log file dump by prefix and file name
    - schedule a command to run in the future
    - get OBC epoch, minimum heap
    - get RTOS task snapshot (running tasks, task priorities)
    - suspend and enable RTOS tasks
    - execute CHIME calibration sequence
    - external reset
    - file system erase
 

## FreeRTOS
FreeRTOS is used as the real-time operating system solution. Now on version 10, version 9 is used on the OBC. It was chosen for its ease of use, excellent documentation, and vendor support with the TMS570. We rely heavily on the following RTOS features:

- task priorities
- task suspension and deletion
- mutexes and queues
We have found that many features that might be otherwise difficult to implement, have become straightforward by placing the functionality into a task that can be suspended. Good examples are external reset capability, and automatic SAFE mode entry if no signals are received over a long period of time. Task stack usage values are determined through experimentation, aided immensely by the command system’s task snapshot command, which can show tasks that are not running because of insufficient stack.

## Radiation Effects Mitigation
The OBC has been tested to 3.0 KRad at the TRIUMF facility. Tests were successful with zero occurrences of permanent data corruption, zero software lockups, and zero detected hardware latchups.

Single event upset effects are mitigated in the following ways:

- TMS570 internal SECDED ECC
- External watchdog triggered reset and self-test upon software lockup
- Reset initiates TMS570 self tests which validate ECC and exercise on-chip peripherals
- Important configuration data and logs stored in flash memory
- Single event latchup events are detected by an OBC-wide current monitor. In the event of an overcurrent event, a full power reset is triggered.
- Fast reset prevents damage from overcurrent condition

Startup procedures are run, including on-chip self-tests which can clear invalid bits
Resets are logged and can be analyzed on the ground.

## Watchdog
An external watchdog is used to mitigate the effects of a software lockup. With an approximately 0.5 second timeout period, the watchdog will execute a power-on reset of the MCU if not ‘pet’ within the timeout period. The watchdog ‘pet’ feature is implemented in an RTOS task. By suspending the watchdog task from the command system, external hard reset capability is provided.

 

## Technical Specifications
Nominal Power Draw:                        313 mW
Maximum Power Draw:                     380 mW
Voltage Supply:                                   3.3 V
Tested Radiation Tolerance:              3.0 KRAD
Mechanical Form Factor:                   PC/104
Nonvolatile Memory:                          8 MB
RTC Backup Time:                               1 week
Clock Speed:                                        60 MHz

 

## Ground Segment
SFUSat has developed a custom ground segment application called Houston. Huston provides the following functionality:

- Live monitoring of all debug messages transmitted by the satellite
- Compatibility with RF or wired connections
- Command sequence creator
- Load and save of command sequences
- Uplink of command sequences
- Validation of uplinked commands
- Parsing of downloaded files and save to .csv
 

## Project Management
The project was managed with google docs as the document storage solution, and GitHub issues as the primary task tracking tool. I also experimented with Trello and Asana. A spreadsheet-based task tracker was adopted in the last phase of the project when the development team ballooned to 8 people under my management.

We preferred to use the GitHub wiki for software documentation.

## Versioning
The hardware versions are outlined below:

**0.1** – experiments on LaunchPad

**0.2** – “DemoBoard” PCB built to prototype individual circuit designs for core features: latchup protection, flash memory, RTC, etc.

**0.3** – First full revision, fully functional once reset circuit was bodged. 2-layer white PCB.

**0.4** – Second revision, minor bug fixes from 0.3 version. 2-layer black PCB.

**0.5** – Version used at CSDC final testing, black + ENIG 4-layer PCB. Minor bug fixes from 0.4, remove before flight programming jumper, full PC/104 connector layout.
The software operated on a continuous release schedule.

## References
[1- On-board data handling for ambitious nanosatellite missions](http://www.academia.edu/11992711/On-board_data_handling_for_ambitious_nanosatellite_missions_using_automotive-grade_lockstep_microcontrollers)