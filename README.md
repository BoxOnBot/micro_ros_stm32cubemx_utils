![banner](.images/banner-dark-theme.png#gh-dark-mode-only)
![banner](.images/banner-light-theme.png#gh-light-mode-only)

# Disclaimer 

This is a forked version of the original uROS repository for STM32CubeMX/IDE. Modifications have been made to ease the setup and configuration process. 
colcon.meta has been configured to run on an STM32 chip with 20kb RAM and 64kb flash. 

# micro-ROS for STM32CubeMX/IDE

This tool aims to ease the micro-ROS integration in a STM32CubeMX/IDE project.

- [micro-ROS for STM32CubeMX/IDE](#micro-ros-for-stm32cubemxide)
  - [Middlewares available](#middlewares-available)
  - [Using this package with STM32CubeMX](#using-this-package-with-stm32cubemx)
  - [Transport configuration](#transport-configuration)
    - [U(S)ART with DMA](#usart-with-dma)
    - [U(S)ART with Interrupts](#usart-with-interrupts)
  - [Customizing the micro-ROS library](#customizing-the-micro-ros-library)
  - [Adding custom packages](#adding-custom-packages)
  - [Purpose of the Project](#purpose-of-the-project)
  - [License](#license)
  - [Known Issues/Limitations](#known-issueslimitations)
## Middlewares available

This package support the usage of micro-ROS on top of two different middlewares:
- [eProsima Micro XRCE-DDS](https://micro-xrce-dds.docs.eprosima.com/en/latest/): the default micro-ROS middleware.
- [embeddedRTPS](https://github.com/embedded-software-laboratory/embeddedRTPS): an experimental implementation of a RTPS middleware compatible with ROS 2. **Instructions on how to use it available [here](./embeddedrtps.md).**

## Using this package with STM32CubeMX

1. Clone this repository in your STM32CubeMX project folder. 
2. Make sure that your STM32CubeMX project is using a `Makefile` toolchain under `Project Manager -> Project`
3. Depending on your use case, you can configure Micro XRCE-DDS to vary how much memory is required in the micro-ros task. 
4. Configure the transport interface on the STM32CubeMX project, check the [Transport configuration](#Transport-configuration) section for instructions on the custom transports provided.
5. Modify the generated `Makefile` to include the following code **before the `build the application` section**:


```makefile
#######################################
# micro-ROS addons
#######################################
LDFLAGS += micro_ros_stm32cubemx_utils/microros_static_library/libmicroros/libmicroros.a
C_INCLUDES += -Imicro_ros_stm32cubemx_utils/microros_static_library/libmicroros/microros_include

# Add micro-ROS utils
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/custom_memory_manager.c
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_allocators.c
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_time.c

# Set here the custom transport implementation
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_transports/dma_transport.c

print_cflags:
	@echo $(CFLAGS)
```

6. Execute the static library generation tool. Compiler flags will retrieved automatically from your `Makefile` and user will be prompted to check if they are correct.


```bash
docker pull microros/micro_ros_static_library_builder:foxy
docker run -it --rm -v $(pwd):/project --env MICROROS_LIBRARY_FOLDER=micro_ros_stm32cubemx_utils/microros_static_library microros/micro_ros_static_library_builder:foxy
```

1. Modify your `freertos.c` to use micro-ROS. An example application can be found in `sample_main.c`.
2. Continue your usual workflow building your project and flashing the binary:

```bash
make -j$(nproc)
```
## Transport configuration

Available transport for this platform are:
### U(S)ART with DMA

Steps to configure:
   - Enable U(S)ART in your STM32CubeMX
   - For the selected USART, enable DMA for Tx and Rx under `DMA Settings`
   - Set the DMA priotity to `Very High` for Tx and Rx
   - Set the DMA mode to `Circular` for Rx: [Detail](.images/Set_UART_DMA1.jpg)
   - For the selected, enable `global interrupt` under `NVIC Settings`: [Detail](.images/Set_UART_DMA_2.jpg)

### U(S)ART with Interrupts

Steps to configure:
   - Enable U(S)ART in your STM32CubeMX
   - For the selected USART, enable `global interrupt` under `NVIC Settings`: [Detail](.images/Set_UART_IT.jpg)
## Customizing the micro-ROS library

All the micro-ROS configuration can be done in `colcon.meta` file before step 3. You can find detailed information about how to tune the static memory usage of the library in the [Middleware Configuration tutorial](https://micro.ros.org/docs/tutorials/core/microxrcedds_rmw_configuration/).

## Adding custom packages

Note that folders added to `microros_static_library/library_generation/extra_packages/` and entries added to `/microros_static_library/library_generation/extra_packages/extra_packages.repos` will be taken into account by this build system.

## Purpose of the Project

This software is not ready for production use. It has neither been developed nor
tested for a specific use case. However, the license conditions of the
applicable Open Source licenses allow you to adapt the software to your needs.
Before using it in a safety relevant setting, make sure that the software
fulfills your requirements and adjust it according to any applicable safety
standards, e.g., ISO 26262.

## License

This repository is open-sourced under the Apache-2.0 license. See the [LICENSE](LICENSE) file for details.

For a list of other open-source components included in this repository,
see the file [3rd-party-licenses.txt](3rd-party-licenses.txt).

## Known Issues/Limitations

There are no known limitations.
