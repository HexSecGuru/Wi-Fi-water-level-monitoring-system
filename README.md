# Smart Water Level Monitor

This project demonstrates how to monitor and manage water levels in a tank using an ultrasonic sensor and an ESP32. It features real-time water level monitoring via a web interface, where users can see the current water level and receive an alert if the water level falls below a minimum threshold.

## Components

- **ESP32 Development Board**
- **HC-SR04 Ultrasonic Sensor**
  - `TRIG_PIN`: Pin 23 (Trigger)
  - `ECHO_PIN`: Pin 22 (Echo)
- **Buzzer**
  - `BUZZER_PIN`: Pin 21 (Buzzer)
- **Wi-Fi Network** (For ESP32 to connect to and serve the web page)

## Circuit Diagram

- Connect the **TRIG_PIN** to **GPIO 23** of the ESP32.
- Connect the **ECHO_PIN** to **GPIO 22**.
- Connect the **BUZZER_PIN** to **GPIO 21**.
- Power the ultrasonic sensor with 5V and ground.

## Features

- **Real-Time Water Level Measurement**: Measures the water level in a tank using the ultrasonic sensor.
- **Web Interface**: View live updates of the water level through a web page served by the ESP32.
- **Threshold Alert**: A buzzer will activate if the water level falls below a specified minimum threshold.
- **Wi-Fi Connectivity**: Connect the ESP32 to a Wi-Fi network and access the water level monitor remotely.

## Functionality

- **Water Level Measurement**: The system uses an ultrasonic sensor to calculate the distance to the water level, then derives the current water level from the tank height and distance.
  
- **Proximity Alert**: If the water level falls below the minimum threshold, the buzzer will sound to alert the user.
  
- **Web Interface**: A simple web page is hosted on the ESP32 that displays the live water level and any alerts.
  
- **Wi-Fi Connectivity**: The ESP32 connects to a Wi-Fi network, allowing remote access to the water level data from any device.

## Code Explanation

The following Arduino sketch captures the main functionality of the system:

```cpp
#include <WiFi.h>
#include <WebServer.h>

#define TRIG_PIN 23
#define ECHO_PIN 22
#define BUZZER_PIN 21
#define TANK_HEIGHT 016.0  // Height of the tank in centimeters
#define MIN_WATER_LEVEL 010.0  // Minimum water level before the buzzer sounds

// Replace these with your network credentials
const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

WebServer server(80);  // Create a web server object that listens for HTTP requests on port 80

float waterLevel = 0.0;

void setup() {
  Serial.begin(115200);
  
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(TRIG_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);  // Make sure the buzzer is off initially

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to Wi-Fi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected to Wi-Fi");

  // Print the ESP32's IP address
  Serial.print("ESP32 IP Address: ");
  Serial.println(WiFi.localIP());

  // Define the handling of root URL "/"
  server.on("/", handleRoot);

  // Define the handling of the "/level" URL for live water level updates
  server.on("/level", handleLevel);

  // Start the server
  server.begin();
}

void loop() {
  // Send a pulse
  float distanceToWater = 0;
  for(int i = 0; i < 5; i++) {
      digitalWrite(TRIG_PIN, HIGH);
      delayMicroseconds(10);
      digitalWrite(TRIG_PIN, LOW);
      long duration = pulseIn(ECHO_PIN, HIGH);
      distanceToWater += (duration / 2.0) * 0.0344;
      delay(10);
  }
  distanceToWater /= 5;

  // Calculate the water level
  waterLevel = TANK_HEIGHT - distanceToWater;

  // Check if the reading is valid
  if (distanceToWater > TANK_HEIGHT) {
      Serial.println("Warning: Distance to water is greater than tank height.");
      return;
  }

  // Print the distance and water level
  Serial.print("Distance to Water: ");
  Serial.print(distanceToWater);
  Serial.println(" cm");

  Serial.print("Water Level: ");
  Serial.print(waterLevel);
  Serial.println(" cm");

  // Check if the water level is below the minimum threshold
  if (waterLevel <= MIN_WATER_LEVEL) {
    digitalWrite(BUZZER_PIN, HIGH);  // Turn on the buzzer
    Serial.println("Alert: Water level is below the minimum threshold!");
  } else {
    digitalWrite(BUZZER_PIN, LOW);  // Turn off the buzzer
  }

  // Handle any incoming client requests
  server.handleClient();

  delay(1000); // Update every second
}

// Handle root URL "/"
void handleRoot() {
  String html = "<html><head><title>Water Level Monitor</title></head>";
  html += "<body><h1>Water Level Monitor</h1>";
  html += "<p>Water Level: <span id='level'>Loading...</span> cm</p>";
  html += "<p id='alert'></p>";
  html += "<script>";
  html += "function getData() {";
  html += "fetch('/level').then(response => response.text()).then(data => {";
  html += "document.getElementById('level').innerText = data.split(',')[0];";
  html += "if (data.split(',')[1] == '1') {";
  html += "document.getElementById('alert').innerText = 'Alert: Water level is below the minimum threshold!';";
  html += "document.getElementById('alert').style.color = 'red';";
  html += "} else {";
  html += "document.getElementById('alert').innerText = '';";
  html += "}});";
  html += "}";
  html += "setInterval(getData, 1000);";  // Update every second
  html += "</script></body></html>";
  server.send(200, "text/html", html);
}

// Handle "/level" URL for live updates
void handleLevel() {
  String level = String(waterLevel) + "," + (waterLevel <= MIN_WATER_LEVEL ? "1" : "0");
  server.send(200, "text/plain", level);
}
