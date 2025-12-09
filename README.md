# Embedded-Capstone---Group-20
# Smart Home Energy Monitoring System  
### STM32F446RE · Custom RTOS Scheduler · INA219 Sensors · ESP32 Wi-Fi Dashboard

This project implements a complete energy monitoring prototype using an STM32 running a lightweight RTOS-style scheduler, dual INA219 current sensors, an ESP32 Wi-Fi dashboard, and an active-low relay driver for a fan load. The system continuously measures voltage, current, and power, and streams that data to a real-time web interface.

---

## Features
- Real-time voltage, current, and power monitoring  
- STM32F446RE running cooperative scheduler (1 ms tick)  
- INA219 at 0x40 (lamp) and 0x41 (fan)  
- UART link between STM32 and ESP32  
- ESP32 SoftAP + embedded dashboard  
- Live JSON endpoints + 24-hour history logging  
- Relay-controlled fan load (PA8, active-low)

---

## Hardware Overview
| Component | Function |
|----------|----------|
| STM32F446RE | RTOS core, sensing, relay control |
| INA219 (0x40) | Lamp sensing |
| INA219 (0x41) | Fan sensing |
| ESP32 | Wi-Fi AP + Dashboard |
| Relay Driver | Switches fan (active-low input) |
| Lamp Load | Monitored only |
| Fan Load | Controlled load |

---

## System Architecture (Text Diagram)

Lamp Load → INA219 (0x40) → STM32 RTOS Core → UART → ESP32 → Web Dashboard  
Fan Load → INA219 (0x41) → STM32 RTOS Core → Relay Driver → Fan Power  

The STM32 performs sensing and relay control.  
The ESP32 handles UI, charting, AP mode, and network logic.

---

## Project Structure

```
/stm32-firmware
    main.c
    rtos_scheduler.c
    rtos_tasks.c
    ina219.c
    gpio.c
    i2c.c
    usart.c
    tim.c

/esp32-dashboard
    dashboard.ino
    index.html
    script.js
    style.css
```

---

## RTOS Summary

A cooperative real-time scheduler is implemented using TIM2 generating a 1 ms tick. Each task has:

- A function pointer  
- A period (ms)  
- An elapsed counter  

Tasks execute when `elapsed >= period`.

### Task Table

| Task | Period | Description |
|------|--------|-------------|
| Task_Sense | 50 ms | Reads INA219 sensors & computes power |
| Task_Comms | 1000 ms | Sends formatted `DATA` line to ESP32 |
| Task_Receive | 20 ms | Reads UART commands to toggle relay |

This design enables predictable timing without full RTOS complexity.

---

## ESP32 Dashboard

The ESP32 hosts a Wi-Fi AP (`EnergyMonitor-Group20`) and web server.  
Responsibilities:

- Parse STM32 UART packets  
- Serve dashboard UI (HTML/CSS/JS)  
- Provide JSON endpoints `/data/latest` & `/data/history`  
- Forward ON/OFF commands to STM32  

---

## Build & Run Instructions

### Flash STM32
1. Open STM32CubeIDE  
2. Build → Run  
3. Confirm console output shows  
   `Device found at 0x40`  
   `Device found at 0x41`  
4. Confirm UART packets appear:  
   `DATA,5.38,0.10,0.538,0.88,0.00,0.000`

### Flash ESP32
1. Open dashboard.ino in Arduino IDE  
2. Select ESP32 Dev Module  
3. Upload  
4. Connect to: `EnergyMonitor-Group20`  
5. Access dashboard at `192.168.4.1`

---

## Testing & Validation
- INA219 devices detected via I2C scan  
- STM32 sends correct formatted data lines  
- ESP32 parses data and updates UI live  
- Fan relay toggles correctly when commanded  
- Dashboard graph updates over time  

---

## Trade-Offs & Limitations
- Only one relay output implemented  
- INA219 low-current readings can drift  
- AP mode prevents simultaneous internet use  
- History stored in RAM (volatile)  
- Cooperative scheduler = no preemption  

---

## Future Improvements
- Add additional controlled loads  
- Use MQTT/Cloud backend  
- Add SD card logging  
- Add OTA updates  
- Improve dashboard graph resolution  

---

## License
MIT License — free academic & personal use.

---
Custom RTOS-powered Smart Home Energy System integrating real-time monitoring, automation, and wireless control to optimize household power use.
