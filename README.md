This code implements a Wi-Fi-enabled water level monitoring system using an ultrasonic sensor and a buzzer. Here's a breakdown of how it works:

  ○ Wi-Fi Setup: The ESP32 connects to a Wi-Fi network using credentials provided in the code. Once connected, it hosts a web server that serves a simple webpage showing the water level and whether an alert is triggered.

  ○ Ultrasonic Sensor: The sensor measures the distance to the water surface in a tank. It sends multiple pulses, averages the readings, and calculates the water level by subtracting the distance to water from the total tank height.

  ○ Water Level Monitoring: If the water level falls below a specified threshold (MIN_WATER_LEVEL), the system activates a buzzer to alert the user. Otherwise, the buzzer remains off.

  ○ Web Interface: The server listens for HTTP requests and provides live water level data. The webpage updates the displayed water level every second and shows an alert message if the water level drops too low.

This system is useful for remotely monitoring water levels in a tank and receiving alerts when water is running low.
