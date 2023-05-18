# ESPHome-modbus-solar-inverter-Sunways

## Description:
This manual describes how to connect the device to Home Assistant via Modbus protocol using ESP8266 and RS485/TTL converter.
In this case it is about connecting a Solar Inverter Sunways.

Many thanks to [peca2345](https://github.com/peca2345) who helped me with the connection and code.

## Info:
- shielded cable recommended, otherwise the error "Modbus CRC Check Failed!" may appear in the log.
- Sunways modbus datasheet: [Interter Sunways](https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/files/11507484/hybrid_inverter_modbus_rtu_protocol_en0332.pdf)
- Sunways User manual: [User Manual EN.pdf](https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/files/11507661/STH.4.12KTL-HT.User.Manual.EN.pdf)
- you need to find out what the Inverter modbus address is - default is 247
- you also need to find out the serial port speed - default 9600
- in ESPHome use the sensor class only for addresses that are read-only
- for addresses that are read/write use the "number" class (you can then change their values in lovelace)
- for each register you want to have in HA you have to create a separate sensor in ESPHome
- you can write the address to the sensor in decimal or hex

## Components:
- ESP8266 (NodeMCU, Wemos D1...)
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-uart-na-rs-485--max485/) 

## Connection:
- ESP D1 (GPIO5) -> TXD TTL
- ESP D2 (GPIO4) -> RXD TTL
- ESP VCC (3V3) -> VCC TTL
- ESP GND -> GND TTL

- RS485 A -> A Inverter
- RS485 B -> B Inverter

## Schematic ESP8266 - NodeMCU:
![Schema](https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/assets/35574450/563bdae1-68e7-4028-8bbe-27a8e6a13f70)
<img width="554" alt="image" src="https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/assets/35574450/bc1b19f7-97f4-490d-a0da-1afb2a9d78af">

## Lovelace:
<img width="375" alt="image" src="https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/assets/35574450/77d7a180-340d-4570-b3bd-0cc1a9412778">

<img width="375" alt="image" src="https://github.com/budisek/ESPHome-modbus-solar-inverter-Sunways/assets/35574450/7fc45319-5bcd-4c9d-bfd0-57bfbc785f15">

## ESPHome code:
(code is still in development, some values from inverter are not yet included)

```
uart:
  id: mod_bus
  tx_pin: GPIO5
  rx_pin: GPIO4 # D6 for Wemos
  baud_rate: 9600
  stop_bits: 1

modbus:
  send_wait_time: 100ms
  id: modbus_sunways1
  

modbus_controller:
  - id: sunways1
    ## the Modbus device addr
    address: 247
    modbus_id: modbus_sunways1
    setup_priority: -10
    update_interval: 5s

text_sensor:
  - platform: template
    id: battery_mode_text
    name: "Battery Mode Text"    

  - platform: template
    id: working_mode_text
    name: "Working Mode Text"
   
sensor:
  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Phase A Power on Meter"
    address: 10994
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "power"
    accuracy_decimals: 1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Phase B Power on Meter"
    address: 10996
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "power"
    accuracy_decimals: 1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Phase C Power on Meter"
    address: 10998
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "power"
    accuracy_decimals: 1   

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Power on Meter"
    address: 11000
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "power"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Grid-Injection Energy on Meter"
    address: 11002
    unit_of_measurement: "kW" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"
    accuracy_decimals: 1     

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Purchasing Energy from Grid on Meter"
    address: 11004
    unit_of_measurement: "kW" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Current Load Power"
    address: 11016
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "power"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total PV Generation on that day"
    address: 11018
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    accuracy_decimals: 1 


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total PV Generation from Installation"
    address: 11020
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"
    filters:
    - multiply: 0.1


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total PV Generation Time from Installation"
    address: 11022
    unit_of_measurement: "H" 
    register_type: holding
    value_type: U_DWORD
    device_class: "duration"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV Input Total Power"
    address: 11028
    unit_of_measurement: "W" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"


# temperature
  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Temperature Sensor 1"
    address: 11032
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    device_class: "temperature"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Temperature Sensor 2"
    address: 11033
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    device_class: "temperature"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

#PV

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV1 Voltage"
    address: 11038
    unit_of_measurement: "V" 
    register_type: holding
    value_type: U_WORD
    device_class: "voltage"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV1 Current"
    address: 11039
    unit_of_measurement: "A" 
    register_type: holding
    value_type: U_WORD
    device_class: "current"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV2 Voltage"
    address: 11040
    unit_of_measurement: "V" 
    register_type: holding
    value_type: U_WORD
    device_class: "voltage"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV2 Current"
    address: 11041
    unit_of_measurement: "A" 
    register_type: holding
    value_type: U_WORD
    device_class: "current"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV1 Input Power"
    address: 11062
    unit_of_measurement: "W" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV2 Input Power"
    address: 11064
    unit_of_measurement: "W" 
    register_type: holding
    value_type: U_DWORD
    device_class: "power"

#stats

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Grid Injection Energy on that day[Meter]"
    address: 41000
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Grid Purchasing Energy on that day[Meter]"
    address: 41001
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Backup Output Energy on that day"
    address: 41002
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Battery Charge Energy on that day"
    address: 41003
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    state_class: "total_increasing"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Battery Discharge Energy on that day"
    address: 41004
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    state_class: "total_increasing"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "PV Generation Energy on that day"
    address: 41005
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    state_class: "total_increasing"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Loading Energy on that day"
    address: 41006
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Energy Purchased from Grid on that day"
    address: 41008
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_WORD
    device_class: "energy"
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Energy injected to grid"
    address: 41102
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Energy Purchased from Grid from Meter"
    address: 41104
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Output Energy on backup port"
    address: 41106
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Energy Charged to Battery"
    address: 41108
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Energy Discharged from Battery"
    address: 41110
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total PV Generation"
    address: 41112
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Loading Energy consumed at grid side"
    address: 41114
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Total Energy Purchased from Grid at inverter side"
    address: 41118
    unit_of_measurement: "kWh" 
    register_type: holding
    value_type: U_DWORD
    device_class: "energy"
    filters:
    - multiply: 0.1    

#battery

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Battery DC Voltage"
    address: 40254
    unit_of_measurement: "V" 
    register_type: holding
    value_type: U_WORD
    device_class: "voltage"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Battery DC Current"
    address: 40255
    unit_of_measurement: "A" 
    register_type: holding
    value_type: S_WORD
    device_class: "current"
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: sunways1
    id: battery_mode
    internal: true
    name: "Battery Mode"
    address: 40256
    register_type: holding
    value_type: U_WORD
    on_value:
      then:
        - lambda: |-
            int state = id(battery_mode).state;
            std::string text;
            if (state == 0) {
              text = "Discharging";
            } else if (state == 1) {
              text = "Charging";
            } else {
              text = "UNKNOWN";
            }
            id(battery_mode_text).publish_state(text);

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Battery Power"
    address: 40258
    unit_of_measurement: "W" 
    register_type: holding
    value_type: S_DWORD
    device_class: "Power"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "BMS Charge Imax"
    address: 42005
    unit_of_measurement: "A" 
    register_type: holding
    value_type: U_WORD
    device_class: "current"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "BMS Discharge Imax"
    address: 42006
    unit_of_measurement: "A" 
    register_type: holding
    value_type: U_WORD
    device_class: "current"

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "SOC"
    address: 43000
    unit_of_measurement: "%" 
    register_type: holding
    value_type: U_WORD
    device_class: "battery"
    filters:
    - multiply: 0.01

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "SOH"
    address: 43001
    unit_of_measurement: "%" 
    register_type: holding
    value_type: U_WORD
    device_class: "battery"
    filters:
    - multiply: 0.01

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "BMS Status"
    address: 43002
    register_type: holding
    value_type: U_WORD

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "BMS Pack Temperature"
    address: 43003
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    device_class: "temperature"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Max Cell Temperature"
    address: 43009
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    device_class: "temperature"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Min Cell Temperature"
    address: 43011
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    device_class: "temperature"
    accuracy_decimals: 1 
    filters:
    - multiply: 0.1

number:
  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Grid Injection Power Limit Switch"
    address: 25100
    # register_type: holding
    value_type: U_WORD
    entity_category: config
    min_value: 0
    max_value: 1
    step: 1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Grid Injection Power Limit Setting"
    address: 25103
    # register_type: holding
    value_type: U_WORD
    entity_category: config
    min_value: 0
    max_value: 1000
    step: 1

  - platform: modbus_controller
    modbus_controller_id: sunways1
    id: working_mode
    name: "Working Mode Setting"
    address: 50000
    # register_type: holding
    value_type: U_WORD
    entity_category: config
    step: 1
    on_value:
      then:
        - lambda: |-
            int state = id(working_mode).state;
            std::string text;
            if (state == 257) {text = "General";} 
            else if (state == 258) {text = "Economic";}
            else if (state == 259) {text = "UPS";} 
            else if (state == 260) {text = "Off-Grid";} 
            else {text = "UNKNOWN";}
            id(working_mode_text).publish_state(text);


  - platform: modbus_controller
    modbus_controller_id: sunways1
    name: "Off-grid Frequency Setting"
    address: 50005
    # register_type: holding
    value_type: U_WORD
    device_class: frequency
    unit_of_measurement: Hz

```
