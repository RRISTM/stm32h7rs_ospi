# STM32H7RS + OSPI example

Example will show how to create a STM32H7RS project with OSPI on nucleo board. 


## What we will create

1. Bootloader to initalize OSPI and jump to our application
2. Application simple project which will tohggle with LED and will be sored in OSPI
3. Embedded flash loader which allow the CubeIDE(or different IDE) or STM32CubeProgrammer to programm the OSPI

## What we need

- CubeMX
- CubeIDE (or different IDE)
- STM32H7RS nucleo

## CubeMX 

### Project startup

#### 1. Run CubeMX
#### 2. Menu>File>New Project

   ![Start new project](./img/24_03_11_387.gif)

#### 3. Search for STM32H7S (same as on nucleo)

![Search for STM32](./img/24_03_11_389.gif)

#### 4. Start new project 

#### 5. Allow default configuration  on MPU

![MPU default](./img/24_03_11_391.gif)


### CubeMX configuration for STM32H7RS

The STM32H7RS allow for each periphery to be configured for Bootloader, Application and External loader

#### Bootloader 
Code will be put into internal FLASH. His job will be to initate OSPI and jump into it

#### Application 
This is out main application. Code will be linked do OSPI area

#### External loader
Code which will handled the OSPI initalization and erase/programming needed why programming tools(CubeIDE, CubeProgrammer, IAR, Keil)

### Pinout

1. LED is on PD13 set it as output. 

![led setup](./img/24_03_11_393.gif)

2. Right click on pin select pin reservation to Application
  
![led reservation](./img/24_03_11_442.gif)

Pin reservation allow to decide which application can use the pin.

### PWR configuration

1. Select RCC
2. Set Supply source to PWR_LDO_SUPPLY (nucleo have only LDO)

![rcc config](./img/24_03_11_435.gif)

### MPU confguration

By default the MPU disable access to external memory, we need to enable it. 

This will be same for `CORTEX_M7_BOOT` and `CORTEX_M7_APPLI`
We will eable acess to OSPI area and allow to execute code from this area and use core cache

1. Open `CORTEX_M7_BOOT` and `CORTEX_M7_APPLI`
2. Region 1 `enable`
3. Region base address `0x90000000`
4. Region size `32MB`
5. Access permission to `ALL ACCESS ALLOWED`
6. Instruction access `ENABLE`
7. Cachable `ENABLE`
8. Bufferable `ENABLE`

![mpu config](./img/24_03_11_439.gif)

### XSPI Mode

We will use XSPI1 (currently for OSPI dosn't matter)

1. Select XSPI1 configuration for `Bootloader` and for `External loader`
   
![xspi selection](./img/24_03_11_395.gif)

2. Select XSPI1
3. Select Mode to `OSPI Mode`
4. Select port to `Port 2 Octo`
5. Select Chip select override to `NCS1 - Port 2`

![xspi mode](./img/24_03_11_397.gif)

### XSPI configration

1. Select Memory type to `Macronix` (memory on nucleo) 
2. Memory size to `32MB`
3. Chip Select High Time Cycle to `2`
4. Select memory type to `Flash`
5. Set De;ay Hold Quarter Cycle to `Enable`

![xspi configruation](./img/24_03_11_403.gif)

### SBS configruation

The OSPI is powered from XSP2 domain. Which is 1.8V. For this reason HSLV(high speed low voltage) feature must be used. 

The SBS must be still enabled in option bytes. 
!!! Be sure that you enable it only when the domain is really low voltage. If the memory is running from 3.3V HSLV can damage the STM32.



1. Select SBS for `Bootloader` and `External Loader`
2. Select SBS periphery
3. Activate SBS
4. Set IO HSLV for XSPIM2 to `ENABLE`

![sbs configruation](./img/24_03_11_405.gif)

The `IO HSLV for XSPIM2` is selected because we use Port 2 for XSPI1. 



### EXTMEM_MANAGER - External Memory manager

Is library which can automaticaly configura the external memory (xSPI) if the memory support  `SFDP` (Serial Flash Discoverable Parameter defined by JEDEC). Or the memory conected to SDMMC. 

1. Select EXTMEM_MANAGER for `Bootloader` and External Loader(enabled by default)
2. Check `Activate External Memeory Manager`

![extmem_manager mode](./img/24_03_11_407.gif)

3. Go to Boot usecase tab
4. Set `select boot code generation`

![extmem_manager boot usecase](./img/24_03_11_409.gif)


We keep `execution in place` (mean code is executed directly from external memory)
Second option Load and Run mean the code will be loaded from memory and stored in RAM(internal or external) where will be executed. 

Memory is `Memory 1 `

5. Go to Memory 1 tab
6. Select Number of memory data line to `EXTMEM_LINK_CONFIG_8LINES`

Rest keep in default Memory driver to `EXTMEM_NOR_SFDP`
And memory instance to `XSPI1`

### EXTMEM_LOADER

1. select EXTMEM_LOADER
2. Check Activate External Memory Loader

![extmem_loader mode](./img/24_03_11_415.gif)

3. select `External Memory Loader` tab
4. Set number of sector to `8192`
5. Set Sector size to `4096`

All this taken from memory DS 

You can change the `loader name` to recognize it if needed
Also `programmming/erase time` can be more tuned based on memory DS

![extmem_loader config](./img/24_03_11_417.gif)


### Clock Configuration

1. Go to `Clock Configuration` tab
2. We can set HCLK to 600MHz
   1. Set DIVM to `/16`
   2. DIVN1 to `300`
   3. Set System Clock Mux to `PLLCLK`
   4. Set BMPRE to `/2`
   5. Set PPRE5, PPRE1 PPRE2 and PPRE4 to `/2`

![hclk config](./img/24_03_11_419.gif)



3. Set XSPI clock to 200MHz
   1. Set XSPI1 Clock mux to `PLL2S`
   2. Set DIVM2 to `/4`
   3. set DIVN2 to `50`
   4. Set DIVS2 to `/4`

![xspi1 clock config](./img/24_03_11_421.gif)

On H7RS the XSPI can run to 200MHz

### Project Manager

1. Go to `Project Manager` tab
2. Name your project and select location 
3. select `ExtMemLoade Project`
4. Select CubeIDE as Toolchain
5. Generate Project

![project configuration](./img/24_03_11_423.gif)


### CubeProgrammer

1. Click to `Connect`
2. select `Option bytes`
3. Select `User option bytes`
4. Enable `HSLV_XSPI2`

![cube programmer](./img/24_03_11_433.gif)

### CubeIDE

1. Open CubeIDE
2. Import Project to CubeIDE or let MX to import project for you.
3. You should have one main project and three sub project in IDE. 

4. Open Application project and the main.c
5. Att there code for GPIO toggling

```c
	  HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_13);
	  HAL_Delay(500);
```

into infinite loop:
```c
  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	  HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_13);
	  HAL_Delay(500);
  }
  /* USER CODE END 3 */
}
```

6. Open Bootloader project
7. Go to main.c
8. Go to main function 
9. Add Cache invalidation functions

```c
  /* USER CODE BEGIN 1 */

  SCB_InvalidateDCache();
  SCB_InvalidateICache();
  /* USER CODE END 1 */
```

This is because after reset the cache can contain invalid data. And the JumToApplication function will clen the cache and this invalid records will cause hardfault.  

10. Compile all projects.
11. Select Application project and run debug

![run debug](./img/24_03_11_425.gif)

12. check that external loader is present in debugger tab

13. Go to setup tab
14. Add Bootloader code that it will be loaded to and we will see the code
15. Clikc OK

![debug setup](./img/24_03_11_427.gif)

If you are using application for debug. You will start in Hardfault. It is because the debugger start the Application directly. He not reset the device. You must click on reset to physically reset the device and allow him to jump to bootloader. And then run to applciation. 