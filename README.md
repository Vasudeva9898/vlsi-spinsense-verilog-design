SpinSense: Motor Speed Monitoring using Hall Effect Sensor and FPGA

SpinSense is a real-time motor speed monitoring system developed using the EDGE Artix 7 FPGA, a Hall effect sensor, a DC motor, and a buzzer. This project demonstrates RPM (Revolutions Per Minute) measurement using magnetic pulse detection and FPGA-based pulse counting logic.

Features

- Accurate RPM monitoring using Hall effect sensor
- FPGA-based hardware logic for pulse counting and timing
- Buzzer alert when RPM exceeds a defined threshold
- Real-time and reliable system without the need for a microcontroller
  
Components Used

- **FPGA**: EDGE Artix 7 Development Board
- **Sensor**: Hall Effect Sensor (digital output)
- **Motor**: DC Motor with attached magnet
- **Output**: Buzzer
- **Power Supply**

How It Works

1. A magnet is attached to the shaft of a rotating DC motor.
2. Every time the shaft rotates, the magnet passes near the Hall effect sensor.
3. The Hall effect sensor sends a digital pulse to the FPGA.
4. The FPGA counts these pulses over 1-second intervals.
5. RPM is calculated using the formula:  
   `RPM = Pulse Count × 60`
6. If RPM exceeds the threshold, a buzzer is triggered.
