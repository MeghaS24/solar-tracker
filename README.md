#include <Arduino.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Initialize the LCD (I2C address: 0x27, 16 columns, 2 rows)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pin Assignments
#define LDR1 34
#define LDR2 35
#define LDR3 32
#define LDR4 33

#define LED_MOTOR1 18
#define LED_MOTOR2 19
#define LED_MOTOR3 21
#define LED_MOTOR4 22

#define LED_CHARGING 2
#define LED_IDLE 4

#define RELAY 25
#define IR 26

// Variables to store sensor and motor states
int ldrVal1, ldrVal2, ldrVal3, ldrVal4;
float trapped_energy = 15.0; // Start battery charge at 15%

void setup() {
    // Initialize serial communication for debugging
    Serial.begin(9600);

    // Initialize pins
    pinMode(IR, INPUT);
    pinMode(LED_CHARGING, OUTPUT);
    pinMode(LED_IDLE, OUTPUT);
    pinMode(RELAY, OUTPUT);

    pinMode(LED_MOTOR1, OUTPUT);
    pinMode(LED_MOTOR2, OUTPUT);
    pinMode(LED_MOTOR3, OUTPUT);
    pinMode(LED_MOTOR4, OUTPUT);

    digitalWrite(LED_MOTOR1, LOW);
    digitalWrite(LED_MOTOR2, LOW);
    digitalWrite(LED_MOTOR3, LOW);
    digitalWrite(LED_MOTOR4, LOW);
    digitalWrite(RELAY, LOW);

    // Initialize LCD
    lcd.init();
    lcd.backlight();
    lcd.setCursor(0, 0);
    lcd.print("Solar Tracker");

    // Add an initial delay only if charge is less than 20%
    if (trapped_energy < 20.0) {
        lcd.setCursor(0, 1);
        lcd.print(" Battery Low ...");
        delay(3000); // 3-second delay
    }

    lcd.clear();
}

void loop() {
    auto_mode();
}

void auto_mode() {
    while (1) {
        // Read LDR values
        ldrVal1 = analogRead(LDR1);
        ldrVal2 = analogRead(LDR2);
        ldrVal3 = analogRead(LDR3);
        ldrVal4 = analogRead(LDR4);

        // Invert LDR values to correspond to light intensity
        ldrVal1 = 4095 - ldrVal1;
        ldrVal2 = 4095 - ldrVal2;
        ldrVal3 = 4095 - ldrVal3;
        ldrVal4 = 4095 - ldrVal4;

        // Simulate charging logic
        simulate_charging();

        // Simulate motor control with LED indicators
        control_motors(ldrVal1, ldrVal2, ldrVal3, ldrVal4);

        // Delay for stability
        delay(500);
    }
}

// Function to simulate the charging logic
void simulate_charging() {
    if (trapped_energy < 100.0) {
        trapped_energy += 5.0; // Increment trapped energy
    }

    lcd.setCursor(0, 0);
    lcd.print("Energy: ");
    lcd.print(trapped_energy, 1); // Display trapped energy
    lcd.print("%     ");

    if (trapped_energy >= 100.0) {
        lcd.setCursor(0, 1);
        lcd.print("Battery Full     ");
        digitalWrite(LED_CHARGING, LOW); // Turn off charging LED
        digitalWrite(RELAY, LOW);        // Turn off relay
        digitalWrite(LED_IDLE, LOW);     // Turn off idle LED
    } else if (trapped_energy <= 20.0) {
        lcd.setCursor(0, 1);
        lcd.print("Battery Low      ");
        digitalWrite(LED_IDLE, HIGH);    // Turn on idle LED
        digitalWrite(LED_CHARGING, LOW); // Turn off charging LED
        digitalWrite(RELAY, LOW);        // Turn off relay
    } else {
        lcd.setCursor(0, 1);
        lcd.print("Charging...      ");
        digitalWrite(LED_CHARGING, HIGH); // Turn on charging LED
        digitalWrite(RELAY, HIGH);        // Turn on relay
        digitalWrite(LED_IDLE, LOW);      // Turn off idle LED
    }

    Serial.print("Current Energy: ");
    Serial.print(trapped_energy);
    Serial.println("%");
}

// Function to simulate motor activation with LED indicators
void control_motors(int val1, int val2, int val3, int val4) {
    if (val1 > 3000) {
        Serial.println("Motor 1 activated");
        Serial.print("LDR1: ");
        Serial.println(ldrVal1);
        digitalWrite(LED_MOTOR1, HIGH); // Turn on LED for Motor 1
    } else {
        digitalWrite(LED_MOTOR1, LOW); // Turn off LED for Motor 1
    }
    
    if (val2 > 3000) {
        Serial.println("Motor 2 activated");
        Serial.print("LDR2: ");
        Serial.println(ldrVal2);
        digitalWrite(LED_MOTOR2, HIGH); // Turn on LED for Motor 2
    } else {
        digitalWrite(LED_MOTOR2, LOW); // Turn off LED for Motor 2
    }

    if (val3 > 3000) {
        Serial.println("Motor 3 activated");
        Serial.print("LDR3: ");
        Serial.println(ldrVal3); 
        digitalWrite(LED_MOTOR3, HIGH); // Turn on LED for Motor 3
    } else {
        digitalWrite(LED_MOTOR3, LOW); // Turn off LED for Motor 3
    }

    if (val4 > 3000) {
        Serial.println("Motor 4 activated");
        Serial.print("LDR4: ");
        Serial.println(ldrVal4);
        digitalWrite(LED_MOTOR4, HIGH); // Turn on LED for Motor 4
    } else {
        digitalWrite(LED_MOTOR4, LOW); // Turn off LED for Motor 4
    }
}
