#include <Wire.h>
#include <LiquidCrystal_I2C.h> // Include the I2C library for the LCD
#include <NewPing.h>           // Library for Ultrasonic sensor

// Define pins for the ultrasonic sensor
#define trigPin 12
#define echoPin 13

// Buzzer pin
#define buzzer 8

// Maximum distance of the water tank (in cm) and the distance threshold for the buzzer
const int maxTankDepth = 100; // Set this to the height of your water tank
const int buzzerThreshold = 80; // Buzzer should activate when the tank is 90% full

// LCD setup for I2C (address 0x27)
LiquidCrystal_I2C lcd(0x27, 16, 2); // Update with the correct address if different

// Setup NewPing library (trig pin, echo pin, max distance)
NewPing sonar(trigPin, echoPin, maxTankDepth);

void setup() {
  // Initialize the LCD with columns, rows, and character size
  lcd.begin(16, 2); // Adjust if necessary
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Water Level");
  lcd.setCursor(0, 1);
  lcd.print("Measurement");
 
  delay(2000);  // Wait 2 seconds before starting measurement

  // Set up the pin modes
  pinMode(buzzer, OUTPUT);
  digitalWrite(buzzer, LOW); // Ensure buzzer is off at the start
 
  // Begin serial communication (optional, for debugging)
  Serial.begin(9600);
}

void loop() {
  // Get the distance measured by the ultrasonic sensor
  int distance = sonar.ping_cm();
 
  // Calculate the water level as a percentage (0% = empty, 100% = full)
  int waterLevel = map(distance, 0, maxTankDepth, 100, 0);
 
  // Ensure the water level is within a valid range (0 to 100%)
  waterLevel = constrain(waterLevel, 0, 100);

  // Print the water level to the Serial Monitor for debugging
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.print(" cm, Water Level: ");
  Serial.print(waterLevel);
  Serial.println(" %");
 
  // Display the water level on the LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Water Level: ");
  lcd.print(waterLevel);
  lcd.print("%");
 
  // Display a simple bar indicator for the water level
  lcd.setCursor(0, 1);
  int bars = map(waterLevel, 0, 100, 0, 16); // 16 character spaces on the LCD
  for (int i = 0; i < bars; i++) {
    lcd.print("|");
  }

  // Check if the water level is 90% or higher, and sound the buzzer for 3 seconds
  if (waterLevel >= buzzerThreshold) {
    digitalWrite(buzzer, HIGH);  // Activate buzzer
    delay(3000);                 // Keep it on for 3 seconds
    digitalWrite(buzzer, LOW);   // Turn off buzzer
  }

  // Wait before taking the next measurement (adjust as needed)
  delay(1000);
}


