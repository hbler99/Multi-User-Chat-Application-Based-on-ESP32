# Multi-user Chat Application Based on Micro-ROS & ESP-ADF

- [中文版本](./README_CN.md)
- Basic Example: ![alt text](./docs/_static/level_regular.png "Regular Example")


## Example Brief

This example demonstrates the process of subscribing to and publishing audio streams using Micro-ROS within the ADF framework.

The pipeline is as follows:

```
[codec_chip] ---> i2s_stream_reader ---> [Micro-ROS publisher]
                                                |
                                                v
                                            [PC ROS2]
                                                ^
                                                |                
                                        [Micro-ROS subscribtion] ---> i2s_stream_writer ---> [codec_chip]
```


## Environment Setup

### Hardware Required

This example runs on the boards that are marked with a green checkbox in the [table](../../README.md#compatibility-of-examples-with-espressif-audio-boards). Please remember to select the board in menuconfig as discussed in Section [Configuration](#configuration) below.


## Build and Flash

### Default IDF Branch
This example only supports IDF release/v5.2. By default, it runs on ADF's built-in branch `$ADF_PATH/esp-idf`.

### Configuration

This example requires ROS2 installed on your PC. It is recommended to install the [Humble](https://docs.ros.org/en/humble/Installation.html) version.

If you are using `Micro-ROS` for the first time, please follow the setup instructions provided at [micro-ROS ESP-IDF component](https://github.com/micro-ROS/micro_ros_espidf_component?tab=readme-ov-file#example).

The default board for this example is `ESP32-S3-Korvo-2`. If you need to run this example on other development boards, select the board in menuconfig, such as `ESP32-Lyrat V4.3`.

```
menuconfig > Audio HAL > ESP32-Lyrat V4.3
```

The dependencies for `Micro-ROS` are located in the `components` directory. If the local files do not contain the necessary library, please install it using the following command:

```bash
git submodule update --init --recursive
```

This example requires a Wi-Fi connection for `Micro-ROS` communication. Configure the Wi-Fi information `menuconfig > micro-ROS Settings`:
```
 menuconfig > micro-ROS Settings > WiFi Configuration > WiFi SSID
 menuconfig > micro-ROS Settings > WiFi Configuration > WiFi Password
```

To configure the `Micro-ROS` agent information (IP address and port number), fill in the `micro-ROS Agent IP` and `micro-ROS Agent Port`:
```
menuconfig > micro-ROS Settings > (PC ROS2 IP) micro-ROS Agent IP 
menuconfig > micro-ROS Settings > (PC ROS2 Port) micro-ROS Agent Port
```

It is recommended to enable `PSRAM` for stable operation:
```
menuconfig > Component config > ESP PSRAM > Support for external, SPI-connected RAM
``` 

If your development board uses an `8-line SPI RAM`, configure the following:
```
menuconfig > Component config > ESP PSRAM > Support for external, SPI-connected RAM > SPI RAM config > Mode (QUAD/OCT) of SPI RAM chip in use (Octal Mode PSRAM)
```

It is also recommended to set the `FreeRTOS` clock frequency to 1000 Hz:
```
menuconfig > Component config > FreeRTOS > Kernel > (1000) configTICK_RATE_HZ
```

### Build and Flash
Build the project and flash it to the board, then run monitor tool to view serial output (replace PORT with your board's serial port name):

```
idf.py -p PORT flash monitor
```

To exit the serial monitor, type ``Ctrl-]``.

See [ESP-IDF Programming Guide](https://docs.espressif.com/projects/esp-idf/en/release-v5.2/esp32/index.html) for full steps to configure and build an ESP-IDF project.


## How to Use the Example

### Example Functionality

- The example requires the `Micro-ROS` agent to be running and connected to the same Wi-Fi network as the development board. On the PC, you can start the `Micro-ROS` agent using the following Docker command:
```
# UDPv4 micro-ROS Agent
sudo docker run -it --rm --net=host microros/micro-ros-agent:humble udp4 --port [PC ROS2 Port] -v6
```

- The example provides a Python3.x script [Audio_com_sub.py](./tools/Audio_com_sub.py), which subscribes to ROS2 topics and plays AMR audio in real-time. Before running the script, please install the required libraries: `pyaudio`, `subprocess`, and `struct`.
To install the libraries, use the following command:
```
pip install pyaudio subprocess struct
```
- When running the script, ensure to modify the [ROS2 topic](./tools/Audio_com_sub.py#L90) name according to your setup. This is crucial for subscribing to the correct topic for receiving audio data.
```
python3 Audio_com_sub.py
```

- This example requires at least two boards to establish audio communication. Both boards will publish audio data to a ROS2 topic and subscribe to audio data from the topic for playback. Ensure both boards are correctly configured and connected to the same Wi-Fi network for communication. Set up **Board 1** and **Board 2** separately.

  - **Board 1**:
    - Publishes audio data to the [ROS2 topic](./main/Micro_ROS_audio_com.c#L419): `Esp32_audio_data_1`
    - Subscribes to the [ROS2 topic](./main/Micro_ROS_audio_com.c#L426): `Esp32_audio_data_2`
  
  - **Board 2**:
    - Publishes audio data to the [ROS2 topic](./main/Micro_ROS_audio_com.c#L419): `Esp32_audio_data_2`
    - Subscribes to the [ROS2 topic](./main/Micro_ROS_audio_com.c#L426): `Esp32_audio_data_1`
- Once the example starts running, it will print the following:
```c
uild:Mar 27 2021
rst:0x1 (POWERON),boot:0x8 (SPI_FAST_FLASH_BOOT)
SPIWP:0xee
mode:DIO, clock div:1
load:0x3fce3810,len:0x178c
load:0x403c9700,len:0x4
load:0x403c9704,len:0xcbc
load:0x403cc700,len:0x2d9c
entry 0x403c9914
I (27) boot: ESP-IDF v5.2.2-224-gdb07674edc 2nd stage bootloader
I (27) boot: compile time Oct 16 2024 17:21:03
I (27) boot: Multicore bootloader
I (31) boot: chip revision: v0.2
I (35) boot.esp32s3: Boot SPI Speed : 80MHz
I (40) boot.esp32s3: SPI Mode       : DIO
I (45) boot.esp32s3: SPI Flash Size : 4MB
I (49) boot: Enabling RNG early entropy source...
I (55) boot: Partition Table:
I (58) boot: ## Label            Usage          Type ST Offset   Length
I (66) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (73) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (81) boot:  2 factory          factory app      00 00 00010000 00300000
I (88) boot: End of partition table
I (92) esp_image: segment 0: paddr=00010020 vaddr=3c0c0020 size=3d66ch (251500) map
I (146) esp_image: segment 1: paddr=0004d694 vaddr=3fc9b700 size=02984h ( 10628) load
I (148) esp_image: segment 2: paddr=00050020 vaddr=42000020 size=bd12ch (774444) map
I (290) esp_image: segment 3: paddr=0010d154 vaddr=3fc9e084 size=03ad4h ( 15060) load
I (294) esp_image: segment 4: paddr=00110c30 vaddr=40374000 size=17638h ( 95800) load
I (327) boot: Loaded app from partition at offset 0x10000
I (327) boot: Disabling RNG early entropy source...
I (339) cpu_start: Multicore app
I (339) octal_psram: vendor id    : 0x0d (AP)
I (339) octal_psram: dev id       : 0x02 (generation 3)
I (342) octal_psram: density      : 0x03 (64 Mbit)
I (348) octal_psram: good-die     : 0x01 (Pass)
I (353) octal_psram: Latency      : 0x01 (Fixed)
I (358) octal_psram: VCC          : 0x01 (3V)
I (363) octal_psram: SRF          : 0x01 (Fast Refresh)
I (369) octal_psram: BurstType    : 0x01 (Hybrid Wrap)
I (375) octal_psram: BurstLen     : 0x01 (32 Byte)
I (380) octal_psram: Readlatency  : 0x02 (10 cycles@Fixed)
I (386) octal_psram: DriveStrength: 0x00 (1/1)
I (392) esp_psram: Found 8MB PSRAM device
I (396) esp_psram: Speed: 40MHz
I (1131) esp_psram: SPI SRAM memory test OK
I (1141) cpu_start: Pro cpu start user code
I (1141) cpu_start: cpu freq: 160000000 Hz
I (1141) cpu_start: Application information:
I (1144) cpu_start: Project name:     Multi-user_Chat_Application
I (1151) cpu_start: App version:      96905d1-dirty
I (1156) cpu_start: Compile time:     Oct 16 2024 17:20:55
I (1163) cpu_start: ELF file SHA256:  f1fa06f33...
I (1168) cpu_start: ESP-IDF:          v5.2.2-224-gdb07674edc
I (1174) cpu_start: Min chip rev:     v0.0
I (1179) cpu_start: Max chip rev:     v0.99 
I (1184) cpu_start: Chip rev:         v0.2
I (1189) heap_init: Initializing. RAM available for dynamic allocation:
I (1196) heap_init: At 3FCA8828 len 00040EE8 (259 KiB): RAM
I (1202) heap_init: At 3FCE9710 len 00005724 (21 KiB): RAM
I (1209) heap_init: At 3FCF0000 len 00008000 (32 KiB): DRAM
I (1215) heap_init: At 600FE010 len 00001FD8 (7 KiB): RTCRAM
I (1221) esp_psram: Adding pool of 8192K of PSRAM memory to heap allocator
I (1230) spi_flash: detected chip: gd
I (1233) spi_flash: flash io: dio
W (1237) spi_flash: Detected size(16384k) larger than the size in the binary image header(4096k). Using the size in the binary image header.
W (1251) i2c: This driver is an old driver, please migrate your application code to adapt `driver/i2c_master.h`
W (1261) ADC: legacy driver is deprecated, please migrate to `esp_adc/adc_oneshot.h`
I (1270) sleep: Configure to isolate all GPIO pins in sleep state
I (1277) sleep: Enable automatic switching of GPIO sleep configuration
I (1284) coexist: coex firmware version: d96c1e51f
I (1289) coexist: coexist rom version e7ae62f
I (1295) main_task: Started on CPU0
I (1305) esp_psram: Reserving pool of 32K of internal memory for DMA/internal allocations
I (1305) main_task: Calling app_main()
I (1315) Multi-user Chat: [ 1 ] Initialize peripherals
I (1315) AUDIO_THREAD: The button_task task allocate stack on external memory
I (1325) AUDIO_THREAD: The esp_periph task allocate stack on internal memory
I (1335) Multi-user Chat: [ 2 ] Start codec chip
I (1335) DRV8311: es8311_codec_init
I (1345) DRV8311: ES8311 in Slave mode
I (1365) gpio: GPIO[48]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0 
W (1375) I2C_BUS: I2C bus has been already created, [port:0]
I (1375) ES7210: ES7210 in Slave mode
I (1395) ES7210: Enable ES7210_INPUT_MIC1
I (1395) ES7210: Enable ES7210_INPUT_MIC2
I (1395) ES7210: Enable ES7210_INPUT_MIC3
W (1405) ES7210: Enable TDM mode. ES7210_SDP_INTERFACE2_REG12: 2
I (1405) ES7210: config fmt 60
I (1415) AUDIO_HAL: Codec mode is 3, Ctrl:1
I (1415) Multi-user Chat: [ 3 ] Create and start input key service
I (1415) AUDIO_THREAD: The input_key_service task allocate stack on internal memory
I (1425) Multi-user Chat: [ 4 ] Create audio pipeline for playback
I (1435) Multi-user Chat: [4.1] Create i2s stream to write data to codec chip and read data from codec chip
I (1445) Multi-user Chat: [4.2] Register all elements to audio pipeline
I (1445) Multi-user Chat: [4.3] Link it together [Micro-ros]-->i2s_stream_writer-->amr_decoder-->[codec_chip]     [codec_chip]-->i2s_stream_reader-->amrnb_encoder-->[Micro-ros]
I (1465) AUDIO_PIPELINE: link el->rb, el:0x3c101d50, tag:i2s_reader, rb:0x3c102b14
I (1475) AUDIO_PIPELINE: link el->rb, el:0x3c1022b4, tag:amr_decoder, rb:0x3c104b5c
I (1485) Multi-user Chat: [ 5 ] Set up  event listener
I (1485) Multi-user Chat: [ 6 ] Start audio_pipeline
W (1495) Multi-user Chat: [ 7 ] Press the [Play] keys to communicate
I (1535) wifi_station_netif: ESP_WIFI_MODE_STA
I (1535) pp: pp rom version: e7ae62f
I (1535) net80211: net80211 rom version: e7ae62f
I (1545) wifi:wifi driver task: 3fcbf690, prio:23, stack:6656, core=0
I (1555) wifi:wifi firmware version: 73edce8
I (1555) wifi:wifi certification version: v7.0
I (1555) wifi:config NVS flash: enabled
I (1555) wifi:config nano formating: disabled
I (1565) wifi:Init data frame dynamic rx buffer num: 32
I (1565) wifi:Init static rx mgmt buffer num: 5
I (1575) wifi:Init management short buffer num: 32
I (1575) wifi:Init static tx buffer num: 16
I (1575) wifi:Init tx cache buffer num: 32
I (1585) wifi:Init static tx FG buffer num: 2
I (1585) wifi:Init static rx buffer size: 1600
I (1595) wifi:Init static rx buffer num: 10
I (1595) wifi:Init dynamic rx buffer num: 32
I (1605) wifi_init: rx ba win: 6
I (1605) wifi_init: tcpip mbox: 32
I (1605) wifi_init: udp mbox: 6
I (1615) wifi_init: tcp mbox: 6
I (1615) wifi_init: tcp tx win: 5760
I (1615) wifi_init: tcp rx win: 5760
I (1625) wifi_init: tcp mss: 1440
I (1625) wifi_init: WiFi IRAM OP enabled
I (1635) wifi_init: WiFi RX IRAM OP enabled
I (1635) phy_init: phy_version 670,b7bc9b9,Apr 30 2024,10:54:13
W (1645) phy_init: failed to load RF calibration data (0xffffffff), falling back to full calibration
I (1735) wifi:mode : sta (74:4d:bd:9c:de:24)
I (1735) wifi:enable tsf
I (1745) wifi_station_netif: wifi_init_sta finished.
I (1755) wifi:new:<1,0>, old:<1,0>, ap:<255,255>, sta:<1,0>, prof:1
I (1755) wifi:state: init -> auth (b0)
I (1805) wifi:state: auth -> assoc (0)
I (1825) wifi:state: assoc -> run (10)
I (1825) wifi:connected with AudioTest, aid = 1, channel 1, BW20, bssid = 24:cf:24:5a:09:e2
I (1825) wifi:security: Open Auth, phy: bgn, rssi: -23
I (1835) wifi:pm start, type: 1

I (1835) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (1845) wifi:set rx beacon pti, rx_bcn_pti: 14, bcn_timeout: 25000, mt_pti: 14, mt_time: 10000
I (1865) wifi:<ba-add>idx:0 (ifx:0, 24:cf:24:5a:09:e2), tid:6, ssn:0, winSize:64
I (1915) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (2855) esp_netif_handlers: sta ip: 192.168.31.219, mask: 255.255.255.0, gw: 192.168.31.1
I (2855) wifi_station_netif: got ip:192.168.31.219
I (2855) wifi_station_netif: connected to ap SSID:AudioTest password:
I (2875) wifi:<ba-add>idx:1 (ifx:0, 24:cf:24:5a:09:e2), tid:0, ssn:2, winSize:64
I (3315) AUDIO_THREAD: The i2s_reader task allocate stack on internal memory
I (3315) AUDIO_ELEMENT: [i2s_reader-0x3c101d50] Element task created
I (3325) AUDIO_THREAD: The amr_encoder task allocate stack on external memory
I (3325) AUDIO_ELEMENT: [amr_encoder-0x3c10215c] Element task created
I (3335) AUDIO_PIPELINE: Func:audio_pipeline_run, Line:359, MEM Total:8454988 Bytes, Inter:139095 Bytes, Dram:139095 Bytes, Dram largest free:41984Bytes

I (3345) AUDIO_ELEMENT: [i2s_reader] AEL_MSG_CMD_RESUME,state:1
I (3355) AUDIO_ELEMENT: [amr_encoder] AEL_MSG_CMD_RESUME,state:1
I (3365) AMRWB_ENCODER: amrwb open
I (3365) AUDIO_PIPELINE: Pipeline started
I (3375) AUDIO_THREAD: The i2s_writer task allocate stack on internal memory
I (3375) AUDIO_ELEMENT: [i2s_writer-0x3c102570] Element task created
I (3385) AUDIO_THREAD: The amr_decoder task allocate stack on external memory
I (3395) AUDIO_ELEMENT: [amr_decoder-0x3c1022b4] Element task created
I (3405) AUDIO_PIPELINE: Func:audio_pipeline_run, Line:359, MEM Total:8434824 Bytes, Inter:134855 Bytes, Dram:134855 Bytes, Dram largest free:37888Bytes

I (3415) AUDIO_ELEMENT: [i2s_writer] AEL_MSG_CMD_RESUME,state:1
I (3425) AUDIO_ELEMENT: [amr_decoder] AEL_MSG_CMD_RESUME,state:1
I (3425) Multi-user Chat: Write AMRWB_HEADER_INFO: #!AMR-
I (3435) Multi-user Chat: Write AMRWB_HEADER_INFO: WB
MR-
I (3445) CODEC_ELEMENT_HELPER: The element is 0x3c1022b4. The reserve data 2 is 0x0.
I (3455) AMR_DECODER: a new song playing
I (3455) AMR_DECODER: current audio is amrwb
I (3545) AUDIO_PIPELINE: Pipeline started

```

- Use the volume down button [Vol-] to decrease the volume and the volume up button [Vol+] to increase the volume.

```c
I (621051) Audio_com: [ * ] [Vol+] input key event
I (621053) Audio_com: [ * ] Volume set to 80 %
I (622091) Audio_com: [ * ] [Vol-] input key event
I (622092) Audio_com: [ * ] Volume set to 70 %

```


- Press the [Play] button on **Board 1** and **Board 2** to start the conversation function, which will print the following:

```c
I (713796) Audio_com: [ * ] [Play] input key event
W (713796) Audio_com: START communicating
```

- Press the [Stop] button on **Board 1** and **Board 2** to stop the conversation function, which will print the following:

```c
I (713796) Audio_com: [ * ] [Stop] input key event
W (713796) Audio_com: STOP communicating
```

### Example Logs
A complete log is as follows:

```c
uild:Mar 27 2021
rst:0x1 (POWERON),boot:0x8 (SPI_FAST_FLASH_BOOT)
SPIWP:0xee
mode:DIO, clock div:1
load:0x3fce3810,len:0x178c
load:0x403c9700,len:0x4
load:0x403c9704,len:0xcbc
load:0x403cc700,len:0x2d9c
entry 0x403c9914
I (27) boot: ESP-IDF v5.2.2-224-gdb07674edc 2nd stage bootloader
I (27) boot: compile time Oct 16 2024 17:21:03
I (27) boot: Multicore bootloader
I (31) boot: chip revision: v0.2
I (35) boot.esp32s3: Boot SPI Speed : 80MHz
I (40) boot.esp32s3: SPI Mode       : DIO
I (45) boot.esp32s3: SPI Flash Size : 4MB
I (49) boot: Enabling RNG early entropy source...
I (55) boot: Partition Table:
I (58) boot: ## Label            Usage          Type ST Offset   Length
I (66) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (73) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (81) boot:  2 factory          factory app      00 00 00010000 00300000
I (88) boot: End of partition table
I (92) esp_image: segment 0: paddr=00010020 vaddr=3c0c0020 size=3d66ch (251500) map
I (146) esp_image: segment 1: paddr=0004d694 vaddr=3fc9b700 size=02984h ( 10628) load
I (148) esp_image: segment 2: paddr=00050020 vaddr=42000020 size=bd12ch (774444) map
I (290) esp_image: segment 3: paddr=0010d154 vaddr=3fc9e084 size=03ad4h ( 15060) load
I (294) esp_image: segment 4: paddr=00110c30 vaddr=40374000 size=17638h ( 95800) load
I (327) boot: Loaded app from partition at offset 0x10000
I (327) boot: Disabling RNG early entropy source...
I (339) cpu_start: Multicore app
I (339) octal_psram: vendor id    : 0x0d (AP)
I (339) octal_psram: dev id       : 0x02 (generation 3)
I (342) octal_psram: density      : 0x03 (64 Mbit)
I (348) octal_psram: good-die     : 0x01 (Pass)
I (353) octal_psram: Latency      : 0x01 (Fixed)
I (358) octal_psram: VCC          : 0x01 (3V)
I (363) octal_psram: SRF          : 0x01 (Fast Refresh)
I (369) octal_psram: BurstType    : 0x01 (Hybrid Wrap)
I (375) octal_psram: BurstLen     : 0x01 (32 Byte)
I (380) octal_psram: Readlatency  : 0x02 (10 cycles@Fixed)
I (386) octal_psram: DriveStrength: 0x00 (1/1)
I (392) esp_psram: Found 8MB PSRAM device
I (396) esp_psram: Speed: 40MHz
I (1131) esp_psram: SPI SRAM memory test OK
I (1141) cpu_start: Pro cpu start user code
I (1141) cpu_start: cpu freq: 160000000 Hz
I (1141) cpu_start: Application information:
I (1144) cpu_start: Project name:     Multi-user_Chat_Application
I (1151) cpu_start: App version:      96905d1-dirty
I (1156) cpu_start: Compile time:     Oct 16 2024 17:20:55
I (1163) cpu_start: ELF file SHA256:  f1fa06f33...
I (1168) cpu_start: ESP-IDF:          v5.2.2-224-gdb07674edc
I (1174) cpu_start: Min chip rev:     v0.0
I (1179) cpu_start: Max chip rev:     v0.99 
I (1184) cpu_start: Chip rev:         v0.2
I (1189) heap_init: Initializing. RAM available for dynamic allocation:
I (1196) heap_init: At 3FCA8828 len 00040EE8 (259 KiB): RAM
I (1202) heap_init: At 3FCE9710 len 00005724 (21 KiB): RAM
I (1209) heap_init: At 3FCF0000 len 00008000 (32 KiB): DRAM
I (1215) heap_init: At 600FE010 len 00001FD8 (7 KiB): RTCRAM
I (1221) esp_psram: Adding pool of 8192K of PSRAM memory to heap allocator
I (1230) spi_flash: detected chip: gd
I (1233) spi_flash: flash io: dio
W (1237) spi_flash: Detected size(16384k) larger than the size in the binary image header(4096k). Using the size in the binary image header.
W (1251) i2c: This driver is an old driver, please migrate your application code to adapt `driver/i2c_master.h`
W (1261) ADC: legacy driver is deprecated, please migrate to `esp_adc/adc_oneshot.h`
I (1270) sleep: Configure to isolate all GPIO pins in sleep state
I (1277) sleep: Enable automatic switching of GPIO sleep configuration
I (1284) coexist: coex firmware version: d96c1e51f
I (1289) coexist: coexist rom version e7ae62f
I (1295) main_task: Started on CPU0
I (1305) esp_psram: Reserving pool of 32K of internal memory for DMA/internal allocations
I (1305) main_task: Calling app_main()
I (1315) Multi-user Chat: [ 1 ] Initialize peripherals
I (1315) AUDIO_THREAD: The button_task task allocate stack on external memory
I (1325) AUDIO_THREAD: The esp_periph task allocate stack on internal memory
I (1335) Multi-user Chat: [ 2 ] Start codec chip
I (1335) DRV8311: es8311_codec_init
I (1345) DRV8311: ES8311 in Slave mode
I (1365) gpio: GPIO[48]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0 
W (1375) I2C_BUS: I2C bus has been already created, [port:0]
I (1375) ES7210: ES7210 in Slave mode
I (1395) ES7210: Enable ES7210_INPUT_MIC1
I (1395) ES7210: Enable ES7210_INPUT_MIC2
I (1395) ES7210: Enable ES7210_INPUT_MIC3
W (1405) ES7210: Enable TDM mode. ES7210_SDP_INTERFACE2_REG12: 2
I (1405) ES7210: config fmt 60
I (1415) AUDIO_HAL: Codec mode is 3, Ctrl:1
I (1415) Multi-user Chat: [ 3 ] Create and start input key service
I (1415) AUDIO_THREAD: The input_key_service task allocate stack on internal memory
I (1425) Multi-user Chat: [ 4 ] Create audio pipeline for playback
I (1435) Multi-user Chat: [4.1] Create i2s stream to write data to codec chip and read data from codec chip
I (1445) Multi-user Chat: [4.2] Register all elements to audio pipeline
I (1445) Multi-user Chat: [4.3] Link it together [Micro-ros]-->i2s_stream_writer-->amr_decoder-->[codec_chip]     [codec_chip]-->i2s_stream_reader-->amrnb_encoder-->[Micro-ros]
I (1465) AUDIO_PIPELINE: link el->rb, el:0x3c101d50, tag:i2s_reader, rb:0x3c102b14
I (1475) AUDIO_PIPELINE: link el->rb, el:0x3c1022b4, tag:amr_decoder, rb:0x3c104b5c
I (1485) Multi-user Chat: [ 5 ] Set up  event listener
I (1485) Multi-user Chat: [ 6 ] Start audio_pipeline
W (1495) Multi-user Chat: [ 7 ] Press the [Play] keys to communicate
I (1535) wifi_station_netif: ESP_WIFI_MODE_STA
I (1535) pp: pp rom version: e7ae62f
I (1535) net80211: net80211 rom version: e7ae62f
I (1545) wifi:wifi driver task: 3fcbf690, prio:23, stack:6656, core=0
I (1555) wifi:wifi firmware version: 73edce8
I (1555) wifi:wifi certification version: v7.0
I (1555) wifi:config NVS flash: enabled
I (1555) wifi:config nano formating: disabled
I (1565) wifi:Init data frame dynamic rx buffer num: 32
I (1565) wifi:Init static rx mgmt buffer num: 5
I (1575) wifi:Init management short buffer num: 32
I (1575) wifi:Init static tx buffer num: 16
I (1575) wifi:Init tx cache buffer num: 32
I (1585) wifi:Init static tx FG buffer num: 2
I (1585) wifi:Init static rx buffer size: 1600
I (1595) wifi:Init static rx buffer num: 10
I (1595) wifi:Init dynamic rx buffer num: 32
I (1605) wifi_init: rx ba win: 6
I (1605) wifi_init: tcpip mbox: 32
I (1605) wifi_init: udp mbox: 6
I (1615) wifi_init: tcp mbox: 6
I (1615) wifi_init: tcp tx win: 5760
I (1615) wifi_init: tcp rx win: 5760
I (1625) wifi_init: tcp mss: 1440
I (1625) wifi_init: WiFi IRAM OP enabled
I (1635) wifi_init: WiFi RX IRAM OP enabled
I (1635) phy_init: phy_version 670,b7bc9b9,Apr 30 2024,10:54:13
W (1645) phy_init: failed to load RF calibration data (0xffffffff), falling back to full calibration
I (1735) wifi:mode : sta (74:4d:bd:9c:de:24)
I (1735) wifi:enable tsf
I (1745) wifi_station_netif: wifi_init_sta finished.
I (1755) wifi:new:<1,0>, old:<1,0>, ap:<255,255>, sta:<1,0>, prof:1
I (1755) wifi:state: init -> auth (b0)
I (1805) wifi:state: auth -> assoc (0)
I (1825) wifi:state: assoc -> run (10)
I (1825) wifi:connected with AudioTest, aid = 1, channel 1, BW20, bssid = 24:cf:24:5a:09:e2
I (1825) wifi:security: Open Auth, phy: bgn, rssi: -23
I (1835) wifi:pm start, type: 1

I (1835) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (1845) wifi:set rx beacon pti, rx_bcn_pti: 14, bcn_timeout: 25000, mt_pti: 14, mt_time: 10000
I (1865) wifi:<ba-add>idx:0 (ifx:0, 24:cf:24:5a:09:e2), tid:6, ssn:0, winSize:64
I (1915) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (2855) esp_netif_handlers: sta ip: 192.168.31.219, mask: 255.255.255.0, gw: 192.168.31.1
I (2855) wifi_station_netif: got ip:192.168.31.219
I (2855) wifi_station_netif: connected to ap SSID:AudioTest password:
I (2875) wifi:<ba-add>idx:1 (ifx:0, 24:cf:24:5a:09:e2), tid:0, ssn:2, winSize:64
I (3315) AUDIO_THREAD: The i2s_reader task allocate stack on internal memory
I (3315) AUDIO_ELEMENT: [i2s_reader-0x3c101d50] Element task created
I (3325) AUDIO_THREAD: The amr_encoder task allocate stack on external memory
I (3325) AUDIO_ELEMENT: [amr_encoder-0x3c10215c] Element task created
I (3335) AUDIO_PIPELINE: Func:audio_pipeline_run, Line:359, MEM Total:8454988 Bytes, Inter:139095 Bytes, Dram:139095 Bytes, Dram largest free:41984Bytes

I (3345) AUDIO_ELEMENT: [i2s_reader] AEL_MSG_CMD_RESUME,state:1
I (3355) AUDIO_ELEMENT: [amr_encoder] AEL_MSG_CMD_RESUME,state:1
I (3365) AMRWB_ENCODER: amrwb open
I (3365) AUDIO_PIPELINE: Pipeline started
I (3375) AUDIO_THREAD: The i2s_writer task allocate stack on internal memory
I (3375) AUDIO_ELEMENT: [i2s_writer-0x3c102570] Element task created
I (3385) AUDIO_THREAD: The amr_decoder task allocate stack on external memory
I (3395) AUDIO_ELEMENT: [amr_decoder-0x3c1022b4] Element task created
I (3405) AUDIO_PIPELINE: Func:audio_pipeline_run, Line:359, MEM Total:8434824 Bytes, Inter:134855 Bytes, Dram:134855 Bytes, Dram largest free:37888Bytes

I (3415) AUDIO_ELEMENT: [i2s_writer] AEL_MSG_CMD_RESUME,state:1
I (3425) AUDIO_ELEMENT: [amr_decoder] AEL_MSG_CMD_RESUME,state:1
I (3425) Multi-user Chat: Write AMRWB_HEADER_INFO: #!AMR-
I (3435) Multi-user Chat: Write AMRWB_HEADER_INFO: WB
MR-
I (3445) CODEC_ELEMENT_HELPER: The element is 0x3c1022b4. The reserve data 2 is 0x0.
I (3455) AMR_DECODER: a new song playing
I (3455) AMR_DECODER: current audio is amrwb
I (3545) AUDIO_PIPELINE: Pipeline started

I (621051) Audio_com: [ * ] [Vol+] input key event
I (621053) Audio_com: [ * ] Volume set to 80 %
I (622091) Audio_com: [ * ] [Vol-] input key event
I (622092) Audio_com: [ * ] Volume set to 70 %
I (713796) Audio_com: [ * ] [Play] input key event
W (713796) Audio_com: START communicating
I (713796) Audio_com: [ * ] [Stop] input key event
W (713796) Audio_com: STOP communicating
```

## Troubleshooting

- At this stage, only two-person chat is supported. Additional boards can subscribe to the topic for playback. If you require a multi-person chat, please implement a mixing module on your own.



## Technical Support and Feedback
Please use the following feedback channels:

* For technical queries, go to the [esp32.com](https://esp32.com/viewforum.php?f=20) forum
* For a feature request or bug report, create a [GitHub issue](https://github.com/espressif/esp-adf/issues)

We will get back to you as soon as possible.
