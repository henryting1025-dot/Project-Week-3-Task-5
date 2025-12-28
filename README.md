# Project-Week-3-Task-5
Obstacle advoidance using HR-SR04 ultrasonic sensor
#include <LiquidCrystal.h>

// === 1. Hardware Pins ===
#define MOTOR_A_EN 3   
#define MOTOR_A_IN1 13 
#define MOTOR_A_IN2 2  

#define MOTOR_B_EN 11  
#define MOTOR_B_IN1 12 
#define MOTOR_B_IN2 1  

// === 2. Ultrasonic Sensor Pins (Using A4/A5) ===
#define TRIG_PIN A4  // Trigger Pin (Output)
#define ECHO_PIN A5  // Echo Pin (Input)

// LCD Settings
LiquidCrystal lcd(8, 9, 4, 5, 6, 7);

// === Runtime Parameters ===
int baseSpeed = 140;   // Speed
int turnSpeed = 180;   // Turning power
int safeDistance = 50; // Safety distance (50cm)

void setup() {
  // Initialize LCD
  lcd.begin(16, 2); 
  lcd.clear();
  lcd.print("Fixed Direction"); 
  lcd.setCursor(0, 1);
  lcd.print("Ready to go...");
  delay(1000);

  // Setup Motor Pins
  pinMode(MOTOR_A_EN, OUTPUT); pinMode(MOTOR_A_IN1, OUTPUT); pinMode(MOTOR_A_IN2, OUTPUT);
  pinMode(MOTOR_B_EN, OUTPUT); pinMode(MOTOR_B_IN1, OUTPUT); pinMode(MOTOR_B_IN2, OUTPUT);

  // Setup Ultrasonic Pins
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // Initial Stop
  stopCar();
  delay(500);
}

void loop() {
  // 1. Get Distance
  long distance = getDistance();

  // 2. Obstacle Avoidance Logic
  if (distance > 1 && distance < safeDistance) {
    
    // === Obstacle Detected ===
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("!! TOO CLOSE !!"); 
    lcd.setCursor(0, 1);
    lcd.print("Backing up..."); 
    
    // Step A: Stop
    stopCar();
    delay(100); 

    // Step B: Back up
    moveBackward(150);
    delay(400);

    // Step C: Turn Right
    turnRight(turnSpeed);
    delay(600); 

    // Step D: Stop turning
    stopCar();
    delay(200);
    
  } else {
    // === No Obstacle Ahead ===
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Scanning...");
    
    // Go Straight
    moveForward(baseSpeed);
  }

  delay(30);
}

// === Helper Functions ===

// Function: Get Distance (cm)
long getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH, 15000); 
  
  if (duration == 0) return 999; 
  return duration * 0.034 / 2;
}

// Motor Control: Move Forward
void moveForward(int speed) {
  // A Forward: HIGH/LOW
  digitalWrite(MOTOR_A_IN1, HIGH); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, speed);

  // B Forward: HIGH/LOW (Reverted to match Task 1)
  // 修正：改回 HIGH/LOW，和 A 一样
  digitalWrite(MOTOR_B_IN1, HIGH); digitalWrite(MOTOR_B_IN2, LOW);
  analogWrite(MOTOR_B_EN, speed);
}

// Motor Control: Move Backward
void moveBackward(int speed) {
  // A Backward: LOW/HIGH
  digitalWrite(MOTOR_A_IN1, LOW); digitalWrite(MOTOR_A_IN2, HIGH);
  analogWrite(MOTOR_A_EN, speed);

  // B Backward: LOW/HIGH 
  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, HIGH);
  analogWrite(MOTOR_B_EN, speed);
}

// Motor Control: Turn Right
void turnRight(int speed) {
  // A Forward (HIGH/LOW)
  digitalWrite(MOTOR_A_IN1, HIGH); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, speed);

  // B Backward (LOW/HIGH)
  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, HIGH);
  analogWrite(MOTOR_B_EN, speed);
}

// Motor Control: Stop
void stopCar() {
  digitalWrite(MOTOR_A_IN1, LOW); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, 0);

  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, LOW);
  analogWrite(MOTOR_B_EN, 0);
}
