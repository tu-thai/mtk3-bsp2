# μT-Kernel 3.0 BSP2 user's manual <!-- omit in toc -->
## RA FSP edition <!-- omit in toc -->
## Version 01.00.B4 <!-- omit in toc -->
## 2024.05.24 <!-- omit in toc -->

- [1. Overview](#1-overview)
  - [1.1. Target board](#11-target-board)
  - [1.2. Development environment](#12-development-environment)
  - [1.3. Software configuration](#13-software-configuration)
- [2. BSP specific features and functions](#2-bsp-specific-features-and-functions)
  - [2.1. Stack pointer monitoring function](#21-stack-pointer-monitoring-function)
  - [2.2. Serial output for debugging](#22-serial-output-for-debugging)
  - [2.3. Use of standard header files](#23-use-of-standard-header-files)
  - [2.4. FPU support](#24-fpu-support)
- [3. Device drivers](#3-device-drivers)
  - [3.1. A/DC device driver](#31-adc-device-driver)
    - [3.1.1. Overview](#311-overview)
    - [3.1.2. How to use the device driver](#312-how-to-use-the-device-driver)
  - [3.2. I2C device driver](#32-i2c-device-driver)
    - [3.2.1. Overview](#321-overview)
    - [3.2.2. How to use the device driver](#322-how-to-use-the-device-driver)
- [4. Procedure for program creation](#4-procedure-for-program-creation)
  - [4.1. Create a project using e2studio](#41-create-a-project-using-e2studio)
  - [4.2. Integrate μT-Kernel 3.0 BSP2](#42-integrate-μt-kernel-30-bsp2)
    - [4.2.1. Include the source code](#421-include-the-source-code)
    - [4.2.2. Add build settings](#422-add-build-settings)
    - [4.2.3. Call OS startup processing](#423-call-os-startup-processing)
  - [4.3. User program](#43-user-program)
    - [4.3.1. Create user program](#431-create-user-program)
    - [4.3.2. usermain function](#432-usermain-function)
    - [4.3.3. Example program](#433-example-program)
  - [4.4. Build and run](#44-build-and-run)
    - [4.4.1. Build the program](#441-build-the-program)
    - [4.4.2. Configure debugging](#442-configure-debugging)
    - [4.4.3. Run the program](#443-run-the-program)
- [5. Revision history](#5-revision-history)


# 1. Overview
This document explains how to use μT-Kernel 3.0 BSP2.
μT-Kernel 3.0 BSP2 is a BSP (Board Support Package) for using the real-time OS μT-Kernel 3.0 with the microcontroller development environment and tools provided by microcontroller manufacturers as well as utilizing provided firmware.

This document describes the μT-Kernel 3.0 BSP2 for the microcontroller board equipped with the Renesas Electronics RA microcontroller.

## 1.1. Target board
The μT-Kernel 3.0 BSP2 supports the following RA microcontroller boards.

| Target board   | MCU                  | CPU core      | Remarks                                          |
| -------------- | -------------------- | ------------- | ------------------------------------------------ |
| EK-RA6M3       | RA6M3(R7FA6M3AH3CFC) | Arm Cortex-M4 |Renesas Electronics RA6M3 MCU Group Evaluation Kit|
| EK-RA8M1       | RA8M1(R7FA8M1AHECBD) | Arm Cortex-M85|Renesas Electronics RA8M1 MCU Group Evaluation Kit|
| Arduino UNO R4 | RA4M1(R7FA4M1AB3CFM) | Arm Cortex-M4 |                                                  |

## 1.2. Development environment
The development environment is e2studio, an integrated development environment from Renesas Electronics Corporation.
In addition, FSP (Flexible Software Package) is used as firmware.
This document has been confirmed to work with the following versions.

`Renesas e² studio Version: 2023-10 (23.10.0)`
`FSP version 5.1.0`

For more information, please visit the following website:

Integrated Development Environment e² studio Information for RA Family
https://www.renesas.com/jp/ja/software-tool/e2studio-information-ra-family

FSP (Flexible Software Package )
https://www.renesas.com/jp/ja/software-tool/flexible-software-package-fsp

## 1.3. Software configuration
μT-Kernel 3.0 BSP2 consists of the real-time OS μT-Kernel 3.0, dependent programs for the target microcontroller board, and sample device drivers.
The following versions of μT-Kernel 3.0 are used.

- μT-Kernel 3.0 (v3.00.07.B0)

The file structure of μT-Kernel 3.0 BSP2 is shown below.

- mtk3_bsp2  Root directory
  - config  Configuration definition files
  - include  Included files
  - mtkernel  OS source code
  - sysdepend  Microcontroller-dependent source code
    - ra_fsp  Source code for RA microcontroller and FSP
      - CPU  CPU (ARM core) dependent source code
      - lib  Hardware-dependent parts of the library
      - device  Sample device drivers
      - doc  Document

The mtkernel directory is a Git submodule of μT-Kernel 3.0 released by TRON Forum. The source code under the mtkernel directory only uses common code that is not hardware dependent.
The source code that depends on the microcontroller board or firmware is in the sysdepend directory.
The device directory contains sample programs for device drivers that use FSP. Basic functions of I2C and A/DC are available.

# 2. BSP specific features and functions
## 2.1. Stack pointer monitoring function
The CPU stack pointer monitor function of the RA microcontroller is used to monitor task stack overflow.
This function can be enabled or disabled by changing the following in the BSP configuration file (config/config_bsp/ra_fsp/config_bsp.h) and building.

```C
/*
 *  Stack pointer monitoring function
 */
#define USE_SPMON (1)  // 1:Valid   0:invalid
```

The stack pointer monitoring function operates differently depending on the CPU core of the microcontroller.

(1) Arm Cortex-M4 core:
The RA MCUs have a unique feature that monitors the stack pointer. If a stack overflow is detected, an NMI exception occurs.

(2) Arm Cortex-M85 core:
The stack pointer is monitored using the ARMv8-M feature. When a stack overflow is detected, a UsageFault exception is raised.

## 2.2. Serial output for debugging
Serial output is available for debugging.
This function can be enabled or disabled by changing the following in the configuration file (config/config.h) and building.

```c
/*---------------------------------------------------------------------- */
/* Use T-Monitor Compatible API Library  & Message to terminal.
 *  1: Valid  0: Invalid
 */
#define	USE_TMONITOR (1)  /* T-Monitor API */
```

The T-Monitor compatible API tm_printf can be used for serial output for debugging. tm_printf has almost the same functionality as the C language standard function printf, but cannot use floating points.
The settings for serial output are as follows.

| Item        | Value     |
| --------    | --------- |
| Rate        | 115200bps |
| Data length | 8bit      |
| Parity      | None      |
| Stop bit    | 1bit      |
| Flow control| None      |

The serial signals use the TX and RX of the Arduino compatible connectors on the board. If you want to use this function, set these terminals to `Peripheral mode` in the FSP configuration.
Also, set the clock supplied to the peripheral. It is not necessary to cancel the low power consumption mode as it is done by initializing this function.
The following shows the microcontroller pins used for debugging serial output.

| Target board   | TX         | RX         |
| -------------- | ---------- | ---------- |
| EK-RA6M3       | P613(TXD7) | P614(RXD7) |
| EK-RA8M1       | P310(TXD3) | P309(RXD3) |
| Arduino UNO R4 | P302(TXD2) | P301(RXD2) |


## 2.3. Use of standard header files
Standard C language header files can be used. The μT-Kernel 3.0 program also uses <stddef.h> and <stdint.h>.
You can specify whether or not to use the standard header file by changing the following in the configuration file (config/config.h). If you do not use the standard header file, an error may occur when you include the standard header file from your program.

```C
/*---------------------------------------------------------------------- */
/*
 *	Use Standard C include file
 */
#define USE_STDINC_STDDEF (1)  /* Use <stddef.h> */
#define USE_STDINC_STDINT (1)  /* Use <stdint.h> */
```

## 2.4. FPU support
This BSP supports the FPU built into the microcontroller. The task context information includes information about the FPU registers.
You can specify whether to use the FPU by changing the following in the configuration file (config/config.h). However, since the FSP enables the FPU, the FPU support of this BSP must always be enabled.

```C
/*---------------------------------------------------------------------- */
/* Use Co-Processor.
 *  1: Valid  0: Invalid
 */
#define	USE_FPU (1)  /* Use FPU */
```

Tasks that use the FPU must specify TA_FPU as a task attribute. However, in the development environment for this microcontroller, it is not possible to specify whether or not to use the FPU for part of a program. Therefore, if you are using the FPU, we recommend that you specify TA_FPU as an attribute for all tasks.
By setting the following to 1 in the configuration file (config/config.h), you can make all tasks FPU-enabled regardless of the task attribute values.

```C
#define	ALWAYS_FPU_ATR (1)  /* Always set the TA_FPU attribute on all tasks */
```

# 3. Device drivers
## 3.1. A/DC Device driver
### 3.1.1. Overview
The A/DC device driver can control the A/D converter built into the microcontroller.
This BSP has A/DC device drivers that support the following devices.

| Device name | BSP device name | Description         |
| ------------| --------------- | ------------------- |
| ADC12       | hadc            | 12bit A/C Converter |

The device driver uses FSP's HAL for internal processing. This device driver is a sample program that shows how to use FSP's HAL with μT-Kernel 3.0, and supports only the basic functions of the device.
The source code for the A/DC device driver is below.

```mtk3_bsp2\sysdepend\ra_fsp\device\hal_adc```

This device driver can be enabled or disabled by changing the following in the BSP configuration file (config/config_bsp/ra_fsp/config_bsp.h) and building the file.

```C
/* ------------------------------------------------------------------------ */
/* Device usage settings
 *	1: Use   0: Do not use
 */
#define DEVCNF_USE_HAL_ADC 1  // A/D conversion device
```

Since the A/DC device driver uses the FSP HAL, please enable the A/DC HAL. The relationship between HAL and this device driver will be explained in the next section.

### 3.1.2. How to use the device driver
(1) A/D converter (hardware) settings:
Configure the A/D converter to be used from the A/DC device driver in e2studio.
Set the `Mode` of the pin you want to use in `Pin Configuration` to `Analog mode`.

(Reference) The correspondence between the analog inputs (A0 to A5) of the Arduino compatible interface of each board and the inputs of the A/D converter of the microcontroller is as follows.

| Arduino analog input | EK-RA6M3   | EK-RA8M1    | Arduino UNO R4 |
| -------------------- | ---------- | ----------- | -------------- |
| A0                   | P000/AN000 | P004(AN000) | P014/AN009     |
| A1                   | P001/AN001 | P003(AN104) | P000/AN000     |
| A2                   | P002/AN002 | P007(AN004) | P001/AN001     |
| A3                   | P507/AN119 | P001(AN101) | P002/AN002     |
| A4                   | P508/AN020 | P014(AN007) | P101/AN021     |
| A5                   | P014/AN005 | P015(AN105) | P100/AN022     |

(2) HAL configuration:
In `Stacks Configuration`, select `New Stack` → `Analog` → `ADC(r_adc)` and set each item of the property `Module g_adc0 ADC(r_adc)` as follows.

| Classification        | Item                        | Setting                              |
| --------------------- | --------------------------- | ------------------------------------ |
| General               | Name                        | g_adc0 (can be changed to any name)  |
|                       | Unit                        | A/D converter unit number to be used |
|                       | Other                       | Leave as default                     |
| Input                 | Channel Scan Mask           | Check the channel you want to use    |
|                       | Other                       | Leave as default                     |
| Interrupt             | Scan End Interrupt Priority | Any interrupt priority               |
|                       | Other                       | Leave as default                     |

If you are using multiple A/D converters, repeat the above steps as many times as necessary.
After setting, click `Generate Project Content` to generate the FSP(HAL) code.

(3) Device driver initialization:
Before using the A/DC device driver, first initialize it with the `dev_init_hal_adc` function. This will generate an A/DC device driver with an associated HAL. This function is defined as follows.

```C
ER dev_init_hal_adc(
  UW unit,  // The unit number of the device(0～DEV_HAL_ADC_UNITNM)
  adc_instance_ctrl_t *hadc,  // ADC control structure
  const adc_cfg_t *cadc,  // ADC configuration structure
  const adc_channel_cfg_t *cfadc  // ADC channel configuration structure
);
```

Specify the parameter unit in order starting from 0. You cannot skip numbers.
The parameters hadc, cadc, and cfadc are HAL information that is automatically generated when FSP is configured.
If the initialization is successful, a device driver with the device name `hadc*` will be generated. In the device name, `*` is assigned an alphabetical character starting from `a`. The device name for unit number 0 will be `hadca`, and the device name for unit number 1 will be `hadcb`.

The device driver is initialized using the `knl_start_device` function in the startup process of μT-Kernel 3.0 BSP2. The knl_start_device function is described in the following file.

`mtk3_bsp2/sysdepend/ra_fsp/devinit.c`

The contents of the knl_start_device function are shown below. Here, a device driver using FSP (HAL) with the name `g_adc0` is initialized, and a μT-Kernel 3.0 device driver with the device name `hadca` is generated.
Please change it according to the FSP (HAL) you actually use.

```C
EXPORT ER knl_start_device(void)
{
  ER err = E_OK;

  // Some parts omitted

#if DEVCNF_USE_HAL_ADC
  err = dev_init_hal_adc(0, &g_adc0_ctrl, &g_adc0_cfg, &g_adc0_channel_cfg);
#endif

  return err;
}
```

(4) Device driver operations:
You can operate device drivers using the μT-Kernel 3.0 device management API. For details on the API, see the μT-Kernel 3.0 specifications.
First, use the open API tk_opn_dev to specify the target device name and open the device.
After opening, data can be acquired using the synchronous read API tk_srea_dev. Specify the A/D converter channel in the parameter data start position.
This device driver can only retrieve one piece of data from one channel per access.

Below is a sample program that uses the A/DC device driver.
This program is the execution function of a task that acquires data from channel 0 and channel 1 of the A/D converter at 500 ms intervals and sends the values ​​to the debug serial output. Match the channel number to the A/D converter you are actually using.

```C
LOCAL void task_1(INT stacd, void *exinf)
{
  UW adc_val1, adc_val2;
  ID dd;     // Device descriptor
  ER err;    // Error code

  dd = tk_opn_dev((UB*)"hadca", TD_UPDATE);  // Open device
  while(1) {
    err = tk_srea_dev(dd, 0, &adc_val1, 1, NULL);  // Get data from A/DC channel 0
    err = tk_srea_dev(dd, 1, &adc_val2, 1, NULL);  // Get data from A/DC channel 1
    tm_printf((UB*)"A/DC A0 =%06d A/DC A1 =%06d\n", adc_val1, adc_val2);  // Debugging output
    tk_dly_tsk(500);  // Wait 500ms
  }
}
```

## 3.2. I2C device driver
### 3.2.1. Overview
The I2C device driver can control the I2C communication device built into the microcontroller.
This BSP has I2C device drivers that support the following devices.

| Device name | BSP device name | Descriptions                                          |
| ----------- | --------------- | ----------------------------------------------------- |
| IIC         | hiic            | IIC bus interface                                     |
| SCI         | hsiic           | Simple IIC mode of serial communication interface SCI |
| I3C         | htiic           | I2C bus interface function of I3C bus interface       |

The device driver uses FSP's HAL for internal processing. This device driver is a sample program that shows how to use FSP's HAL with μT-Kernel 3.0, and supports only the basic functions of the device.
Below is the source code for the I2C device driver.

- I2C(Use IIC)
```mtk3_bsp2/sysdepend/ra_fsp/device/hal_i2c```

- SCI_I2C(Use SCI)
```mtk3_bsp2/sysdepend/ra_fsp/device/hal_sci_i2c```

- I3C_I2C(Use I3C)
```mtk3_bsp2/sysdepend/ra_fsp/device/hal_i3c_i2c```

This device driver can be enabled or disabled by changing the following in the BSP configuration file (config/config_bsp/ra_fsp/config_bsp.h) and building the file.
The I2C communication device installed varies depending on the type of microcontroller. Do not set it to use an I2C communication device that is not installed.

```C
/* ------------------------------------------------------------------------ */
/* Device usage settings
 *	1: Use   0: Do not use
 */
#define DEVCNF_USE_HAL_IIC 1  // I2C communication device (Use IIC)
#define DEVCNF_USE_HAL_SCI_IIC 1  // I2C communication device (Use SCI)
#define DEVCNF_USE_HAL_I3C_IIC 1  // I2C communication device (Use I3C)
```

The I2C device driver uses FPS HAL, so please enable I2C HAL. The relationship between HAL and this device driver will be explained in the next section.

### 3.2.2. How to use the device driver
(1) I2C (hardware) settings:
Configure the I2C settings to be used from the I2C device driver in e2studio.
Set the Mode of the pin used in `Pin Configuration` to `Peripheral mode`.

(Reference) The correspondence between the I2C signals of each board and the I2C terminals of the microcontroller is as follows.

| I2C signals on the board  | EK-RA6M3      | EK-RA8M1      | Arduino UNO R4 |
| ------------------------- | ------------- | ------------- | -------------- |
| Grove-1 I2C SDA           | P409/SCI3_SDA | P401/I3C_SDA0 | -              |
| Grove-1 I2C SCL           | P408/SCI3_SCL | P400/I3C_SCL0 | -              |
| Grove-2 I2C SDA           | P409/SCI3_SDA | P511/SDA1     | -              |
| Grove-2 I2C SCL           | P408/SCI3_SCL | P512/SCL1     | -              |
| Arduino I2C SDA           | P511/SDA2     | P401/I3C_SDA0 | P101/SDA1      |
| Arduino I2C SCL           | P512/SCL2     | P400/I3C_SCL0 | P100/SCL1      |

**Note** When using the Grove-1 and Arduino I2C interfaces as I2C on the EK-RA8M1 board, please configure the following pins.

| Pin  | Setting                    |
| ---- | -------------------------- |
| P115 | Output mode (Initial Low)  |
| P711 | Output mode (Initial High) |
| PB00 | Output mode (Initial High) |

(2) HAL configuration:
In `Stacks Configuration`, select the target I2C peripheral from `New Stack` → `Connectivity`.

- I2C(Use IIC)
Select the I2C Master (r_iic_master) and set each item of the properties `Module g_i2c_master0` as follows.

| Item               | Setting                                    |
| ------------------ | ------------------------------------------ |
| Name               | g_i2c_master0 (Can be changed to any name) |
| Unit               | I2C channel number to use                  |
| Interrupt Priority | Any interrupt priority                     |
| Other              | Leave as default                           |

- SCI_I2C(Use SCI)
Select I2C Master (r_sci_i2c) and set each item of the properties `Module g_i2c0` as follows.

| Item               | Setting                             |
| ------------------ | ----------------------------------- |
| Name               | g_i2c0 (Can be changed to any name) |
| Unit               | I2C channel number to use           |
| Interrupt Priority | Any interrupt priority              |
| Other              | Leave as default                    |

- I3C_I2C(Use I3C)
Select I3C (r_i3c) and set each item of the properties `Module g_i3c` as follows.

| Item               | Setting                              |
| ------------------ | ------------------------------------ |
| Name               | g_i3c0  (Can be changed to any name) |
| Device Type        | Main Master                          |
| Interrupt Priority | Any interrupt priority               |
| Other              | Leave as default                     |

If you are using multiple I2Cs, repeat the above steps as many times as necessary.
After setting, click `Generate Project Content` to generate the FSP(HAL) code.

(3) Device driver initialization:
Before using the I2C device driver, first initialize it using the I2C device driver initialization function. This will generate an I2C device driver with the specified HAL associated with it. This function is defined as follows.

- I2C(Use IIC)
```C
ER dev_init_hal_i2c(
  UW unit,  / Device unit number (0～DEV_HAL_ADC_UNITNM)
  i2c_master_ctrl_t *hi2c,  // I2C control structure
  const i2c_master_cfg_t *ci2c  // I2C configuration structure
);
```

- SCI_I2C(Use SCI)
```C
ER dev_init_hal_sci_i2c(
  UW unit,  // Device unit number (0～DEV_HAL_ADC_UNITNM)
  i2c_master_ctrl_t *hi2c,  // SCI_I2C control structure
  const i2c_master_cfg_t *ci2c  // I2C configuration structure
);
```

- I3C_I2C(Use I3C)
```C
ER dev_init_hal_i3c_i2c(
  UW unit,  // Device unit number (0～DEV_HAL_ADC_UNITNM)
  i3c_instance_ctrl_t *hi3c,  // I3C control structure
  const i3c_cfg_t *ci3c  // I3C configuration structure
);
```

Specify the parameter unit in order starting from 0. You cannot skip numbers.
The parameters hai2c, ci2c (or hai3c, ci3c) are HAL information that is automatically generated when FSP is configured.
If the initialization is successful, a device driver with the device name `hiic*` (depending on the type of I2C peripheral) will be generated.
The `*` in the device name is assigned an alphabetical character starting from `a`. The device name for unit number 0 is `hiica`, and the device name for unit number 1 is `hiicb`.

The device driver is initialized using the `knl_start_device` function in the startup process of μT-Kernel 3.0 BSP2. The knl_start_device function is described in the following file.

`mtk3_bsp2/sysdepend/ra_fsp/devinit.c`

The contents of the knl_start_device function are shown below. Here, a μT-Kernel 3.0 device driver associated with HAL is generated.
Change the specifications according to the FSP (HAL) you actually use.

```C
ER knl_start_device(void)
{
  ER err = E_OK;

#if DEVCNF_USE_HAL_IIC
  err = dev_init_hal_i2c(0, &g_i2c_master0_ctrl, &g_i2c_master0_cfg);
  if(err < E_OK) return err;
#endif

#if DEVCNF_USE_HAL_SCI_IIC
  err = dev_init_hal_sci_i2c(0, &g_i2c0_ctrl, &g_i2c0_cfg);
  if(err < E_OK) return err;
#endif

#if DEVCNF_USE_HAL_I3C_IIC
  err = dev_init_hal_i3c_i2c(0, &g_i3c0_ctrl, &g_i3c0_cfg);
  if(err < E_OK) return err;
#endif
  // The rest is omitted
}
```

(4) Device driver operations:
You can operate device drivers using the μT-Kernel 3.0 device management API. For details on the API, see the μT-Kernel 3.0 specifications.
This device driver only supports I2C controller (master) mode.
First, use the open API tk_opn_dev to specify the target device name and open the device.
After opening, data can be received using the synchronous read API tk_srea_dev, and data can be sent using the synchronous write API tk_swri_dev. Specify the target (slave) address in the data start position of the parameter.

(5) Device register access:
The following functions are provided to access registers in a target device connected via I2C. This corresponds to a register access procedure within a relatively commonly used device. However, please note that it is not available for all devices.

The register read function sends the register address value (1 byte) to the target device, and then receives data (1 byte).
The function name varies depending on the type of device driver.

| Device type | Function name        |
| ----------- | -------------------- |
| I2C         | hal_i2c_read_reg     |
| SCI_I2C     | hal_sci_i2c_read_reg |
| I3C_I2C     | hal_i3c_i2c_read_reg |

```C
/* Register read function */
ER hal_***_read_reg(
  ID dd,  // Device descriptor
  UW sadr,  // Target address
  UW radr,  // Register address (Only the lower 8 bits are valid)
  UB *data  // Read data (1byte)
);
```
The register write function sends the register address value (1 byte) to the target device, followed by data (1 byte).
The function name varies depending on the type of device driver.

| Device type | Function name         |
| ----------- | --------------------- |
| I2C         | hal_i2c_write_reg     |
| SCI_I2C     | hal_sci_i2c_write_reg |
| I3C_I2C     | hal_i3c_i2c_read_reg  |

```C
/* Register write function */
ER hal_***_write_reg(    // hal_sci_i2c_write_reg for SCI_I2C
  ID dd,  // Device descriptor
  UW sadr,  // Target address
  UW radr,  // Register address (Only the lower 8 bits are valid)
  UB data   // Data to be written (1byte)
);
```

# 4. Procedure for program creation
This section explains the steps to create a program project in e2 studio, integrate the μT-Kernel 3.0 BSP2, build it, and run it.
It is assumed that the FSP for RA microcontrollers is installed in e2studio.

## 4.1. Create a project using e2studio
Follow the steps below to create a project for the target microcontroller board.

(1) Select the menu [File] → [New] → [Renesas C/C++ Project] → [Renesas RA].

(2) Select [Renesas RA C/C++ Project].

(3) Select the target microcontroller board. If the target board is not available, select `Custom User Board` and specify `Device`.
As you cannot select the board for the Arduino UNO R4, please set it as follows.

- Board : Custom User Board
- Device : R7FA4M1AB3CFM
- Clock settings
  - HOCO : 48MHz
  - PLL Src : Disabled
  - Clock src : HOCO
- Subclock Populated: Not Populated

(4) For [Project Template Selection], select Bare Metal - Minimal.

(5) Configure the microcontroller pin settings, clock settings, and other hardware settings according to the application you are developing.

## 4.2. Integrate μT-Kernel 3.0 BSP2
### 4.2.1. Include the source code
Integrate the μT-Kernel 3.0 BSP2 source code into the created project.
Get μT-Kernel 3.0 BSP2 from the GitHub repository below, and place the μT-Kernel 3.0 BSP2 source code directory mtk3_bsp2 in the project directory.

`https://github.com/tron-forum/mtk3_bsp2.git`

When using git commands, make the project directory the current directory and run the following command.

`git clone --recursive  https://github.com/tron-forum/mtk3_bsp2.git`

Since μT-Kernel 3.0 BSP2 includes the μT-Kernel 3.0 repository as a git submodule, --recursive is required to obtain the entire source code.

The imported μT-Kernel 3.0 BSP2 directory may not be configured to be built by e2studio, so change the properties.
If `Exclude resource from build` is checked, uncheck it.

### 4.2.2. Add build settings
Add settings to the project properties to build the source code for μT-Kernel 3.0 BSP2.
Configure the following in [C/C++ Build] → [Settings] → [Tool Settings].

(1) [GNU Arm Cross C compiler]
Set the target name of the target MCU board in [Define symbols].

| Target board   | Target name            |
| -------------- | ---------------------- |
| EK-RA6M3       | \_RAFSP_EK_RA6M3_      |
| EK-RA8M1       | \_RAFSP_EK_RA8M1_      |
| Arduino UNO R4 | \_RAFSP_ARDUINO_UNOR4_ |


(2) [GNU Arm Cross C Compiler] → [includes]
Set the following in [Include paths].
It is assumed that the μT-Kernel 3.0 BSP2 directory mtk3_bsp2 is at the top of the project directory tree. If you have changed the location of the directory, change it accordingly.

```
"${workspace_loc:/${ProjName}/mtk3_bsp2}"
"${workspace_loc:/${ProjName}/mtk3_bsp2/config}"
"${workspace_loc:/${ProjName}/mtk3_bsp2/include}"
"${workspace_loc:/${ProjName}/mtk3_bsp2/mtkernel/kernel/knlinc}"
```

(3) [GNU Arm Cross Assembler] → [Preprocessor]
Set up the same as (1).

(4) [GNU Arm Cross Assembler] → [includes]
Set the same as (2).

### 4.2.3. Calling OS startup processing
The generated project executes the hal_entry function after performing hardware initialization processing.
Add code to execute the μT-Kernel 3.0 startup process knl_start_mtkernel function from the hal_entry function.
The hal_entry function is written in the file below.

`<Project Directory>/src/hal_entry.c`

Write the following in the hal_entry function.

```C
void hal_entry(void)
{
  void knl_start_mtkernel(void);
  knl_start_mtkernel();
}
```

## 4.3. User program
### 4.3.1. Create user program
Create a user program to run with μT-Kernel 3.0.
You can choose any name and location for the directory to create the user program in. For example, create a directory named "application" at the top of the project directory tree.

### 4.3.2. usermain function
When μT-Kernel 3.0 starts up, it executes the usermain function of the user program. Therefore, we define the usermain function.
The usermain function has the following format.

```C
INT usermain(void);
```

The usermain function is called from the task (initial task) that μT-Kernel 3.0 first generates and executes.
Also, μT-Kernel 3.0 shuts down when the usermain function ends. Therefore, normally the usermain function is not terminated, but put into a wait state using μT-Kernel 3.0 task suspend API tk_slp_tsk or similar.
The initial task is set to a high priority, so no other tasks will be executed while the usermain function is running.

If the user program does not define a usermain function, the default usermain function written in the following file will be executed.
This usermain function exits without doing anything. Therefore, μT-Kernel 3.0 will shut down immediately.

`mtk3_bsp2/mtkernel/kernel/usermain/usermain.c`

The default usermain function has the weak attribute specified, so if the usermain function exists in the user program, it will be disabled. There is no need to modify the file.

### 4.3.3. Example program
Below is an example of a user program. This program executes two tasks. Both tasks display a string to the debug serial output at regular intervals.

```C
#include <tk/tkernel.h>
#include <tm/tmonitor.h>

LOCAL void task_1(INT stacd, void *exinf);  // Task execution function
LOCAL ID tskid_1;  // Task ID number
LOCAL T_CTSK ctsk_1 = {  // Task creation information
  .itskpri = 10,
  .stksz = 1024,
  .task = task_1,
  .tskatr = TA_HLNG | TA_RNG3,
};

LOCAL void task_2(INT stacd, void *exinf);  // Task execution function
LOCAL ID tskid_2;  // Task ID number
LOCAL T_CTSK ctsk_2 = {  // Task creation information
  .itskpri = 10,
  .stksz = 1024,
  .task = task_2,
  .tskatr = TA_HLNG | TA_RNG3,
};

LOCAL void task_1(INT stacd, void *exinf)
{
  while(1) {
    tm_printf((UB*)"task 1\n");
    tk_dly_tsk(500);
  }
}

LOCAL void task_2(INT stacd, void *exinf)
{
  while(1) {
    tm_printf((UB*)"task 2\n");
    tk_dly_tsk(700);
  }
}

/* usermain function */
EXPORT INT usermain(void)
{
  tm_putstring((UB*)"Start User-main program.\n");

  /* Create & Start Tasks */
  tskid_1 = tk_cre_tsk(&ctsk_1);
  tk_sta_tsk(tskid_1, 0);

  tskid_2 = tk_cre_tsk(&ctsk_2);
  tk_sta_tsk(tskid_2, 0);

  tk_slp_tsk(TMO_FEVR);

  return 0;
}
```

## 4.4. Build and run
### 4.4.1. Build the program
Select the project and build it by selecting [Project] → [Build Project] from the menu or by right-clicking the project.
If there are no errors and the process finishes successfully, an executable program (ELF file) is generated.

### 4.4.2. Configure debugging
Configure the debug settings according to the debugger you use.
The type of debugger is specified when creating a project (it can be changed later). For RA microcontrollers, the debugger can be selected from `E2`, `E2 Lite`, or `J-Link`. If the microcontroller board is equipped with a debugger, specify the type of debugger.
Select [Run] → [Debug Configuration] from the menu and configure various debug settings according to the microcontroller board and debugger.

### 4.4.3. Run the program
Connect the microcontroller board to the PC, select [Run] → [Debug Configuration] from the menu, check the settings, and then click the [Debug] button. The execution program will be transferred to the microcontroller board and debugging will begin.
Also, if you select [Run] → [Debug] from the menu, the program that was previously debugged will be debugged again.
The execution program is written to the flash memory of the microcontroller board, so if you turn on the microcontroller board with the debugger disconnected, the program will be executed directly.


# 5. Revision history

| Version | Date       | Contents                                                                                                                 |
| ------- | ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| 1.00.B4 | 2024.05.24 | Corrected typos                                                                                                          |
| 1.00.B3 | 2024.04.10 | Added explanation of I2C device                                                                                          |
| 1.00.B2 | 2024.03.21 | Corrected typos                                                                                                          |
| 1.00.B1 | 2024.02.29 | - Added EK-RA8M1 to the supported boards. Added related information.</br>- Updated device drivers and other content |
| 1.00.B0 | 2023.12.15 | Create New                                                                                                               |
