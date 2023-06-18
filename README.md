# ESP-IDF with Adafruit-ST7735-Library


This is a repository containing information regarding the use of an *Adafruit-ST7735-Library*. 
**Prerequisite**: ESP-IDF version 4.4 or 5.1
## Project Files Structure
It is assumed that the ESP-IDF project has the normal project structure described in the [ESP-IDF documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/build-system.html)

```
- myProject/
             - CMakeLists.txt
             - sdkconfig
             - components/ - component1/ - CMakeLists.txt
                                         - Kconfig
                                         - src1.c
                           - component2/ - CMakeLists.txt
                                         - Kconfig
                                         - src1.c
                                         - include/ - component2.h
             - main/       - CMakeLists.txt
                           - src1.c
                           - src2.c

             - build/
```

In our case we are using the [Adafruit-ST7735 Display Library](https://github.com/adafruit/Adafruit-ST7735-Library), [Arduino Core Graphics Library](https://github.com/adafruit/Adafruit-GFX-Library), [Arduino BusIO Library](https://github.com/adafruit/Adafruit_BusIO), and finally [Arduino Core](https://github.com/espressif/arduino-esp32) as ESP-IDF components.

There are two changes that are necessary to make in order for this to work.

1. The [Adafruit-ST7735-Library](https://github.com/adafruit/Adafruit-ST7735-Library) does not contain a CMakeList.txt, I created one in a [forked repository](https://github.com/DaintyJet/Adafruit-ST7735-Library): https://github.com/DaintyJet/Adafruit-ST7735-Library
2. The [Arduino Core](https://github.com/espressif/arduino-esp32) needs to have the CMakeList.txt file edited so a esp component will be discovered, this is described in the setup phase.

## Component Setup  
**Notice**: When running a project with the arduino core for the first time the ``` idf.py ``` commands will *fail*. We need to edit the generated SDKCONFIG to increase the ``` CONFIG_FREERTOS_HZ ``` value.
``` 
# Before 
CONFIG_FREERTOS_HZ=100

# After 
CONFIG_FREERTOS_HZ=1000
```

1. We need to make the components folder if it is not created yet
    ```
    mkdir components
    ```
2.  **Install** the [arduino-esp library](https://github.com/espressif/arduino-esp32) into a folder named arduino
    ```sh
    cd components && \                                            # Enter into components folder                                           
    git clone https://github.com/espressif/arduino-esp32 arduino  # Clone arduino core
    ```
3. **Install** the [Adafruit-ST7735 Display Library](https://github.com/adafruit/Adafruit-ST7735-Library)
    ```sh
    # You should still be in the components folder
    git clone https://github.com/DaintyJet/Adafruit-ST7735-Library
    ```
4. **Install** the [Arduino Core Graphics Library](https://github.com/adafruit/Adafruit-GFX-Library)
    ```sh
    # You should still be in the components folder
    git clone https://github.com/adafruit/Adafruit-GFX-Library
    ```
5. **Install** the [Arduino BusIO Library](https://github.com/adafruit/Adafruit_BusIO)
    ``` 
    # You should still be in the components folder
    git clone https://github.com/adafruit/Adafruit_BusIO
    ```
6. Enter into the ESP-Display directory 
    ```sh
    cd components/arduino 
    ```
7. Switch the branch used by the arduino repository
    ```sh
    git switch esp-idf-v5.1-libs
    ```
8. Modify the ``` components/arduino/CMakeList.txt ``` so the esp_partition can be located.
    ```
    # Before
    set(requires spi_flash mbedtls mdns wifi_provisioning wpa_supplicant esp_adc esp_eth http_parse)

    # After (add esp_partition)
    set(requires spi_flash mbedtls mdns wifi_provisioning wpa_supplicant esp_adc esp_eth http_parser esp_partition)
    ``` 
9. Enabled backwards compatibility within FREERTOS
    ```sh
    # compoent config -> FreeRtos -> Kernel -> configENABLE_BACKWARDS_COMPATIBILITY 
    idf.py menuconfig 
    ```
    * This may not work, due to a FREERTOS_HZ value. We need to manually edit this as described as a pre-step.
10. Enable Enable pre-shaired-cipher suites 
    ```sh
    #  Component config -> mbedTLS -> TLS Key Exchange Methods -> Enable pre shared-key ciphersuites
    #  Component config -> mbedTLS -> TLS Key Exchange Methods -> Enable PSK based ciphersuite modes
    idf.py menuconfig   
    ```
11. Enable Arduino Structure (If wanted)
    ```
    # Turn on Autostart Arduino setup and loop on boot
    # Arduino Configuration -> Autostart Arduino setup and loop on boot -> select
    idf.py menuconfig 
    ```
12. Add necessary component dependency
    ```sh
    # Do not run this in the component library, do it from the project directory
    idf.py add-dependency "espressif/mdns^1.1.0"
    ```
## Arduino Specific Steps
1. Rename the main/project_file.c to a **cpp** file
    ``` 
    mv main/project_file.c main/project_file.cpp
    ```
2. Edit the CMakeList.txt file to reflect this change 
    ```
    # project_file.c -> project_file.cpp
    nano /main/CMakeList.txt
    ```
## Final Project Structure 
The only thing to pay attention to are the components and the .cpp file.
```
- myProject/
             - CMakeLists.txt
             - sdkconfig
             - components/ 
                           - arduino/ 
                                         - CMakeLists.txt
                                         - Kconfig
                                         - ....
                           - Adafruit-ST7735 Display Library/ 
                                         - CMakeLists.txt
                                         - Kconfig
                                         - ....
                           - Arduino Core Graphics Library/ 
                                         - CMakeLists.txt
                                         - Kconfig
                                         - ....
                           - Arduino BusIO Library/ 
                                         - CMakeLists.txt
                                         - Kconfig
                                         - ....
             - main/       - CMakeLists.txt
                           - src1.cpp
                           - Kconfig

             - build/
```
