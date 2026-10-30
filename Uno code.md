# Laser_Beam_Detection

#include <Servo.h>

const int LDR_ZONE1 = A0;
const int LDR_ZONE2 = A1;
const int LDR_ZONE3 = A2;

const int SERVO_PIN = 9;

Servo pointerServo;
int currentAngle = 0;
int targetAngle = 0; // Added to track where the servo needs to go

const int ALERT_PIN = 8;       // Uno D8 -> Nano D4
const int ZONE2_SIGNAL = 7;    // Uno D7 -> Nano D6
const int ZONE3_SIGNAL = 5;    // Uno D5 -> Nano D7

const int THRESHOLD_ZONE1 = 1000;
const int THRESHOLD_ZONE2 = 960;
const int THRESHOLD_ZONE3 = 1000;

void setup() {
  Serial.begin(9600);

  pinMode(ALERT_PIN, OUTPUT);
  pinMode(ZONE2_SIGNAL, OUTPUT);
  pinMode(ZONE3_SIGNAL, OUTPUT);

  digitalWrite(ALERT_PIN, LOW);
  digitalWrite(ZONE2_SIGNAL, LOW);
  digitalWrite(ZONE3_SIGNAL, LOW);

  pointerServo.attach(SERVO_PIN);
  pointerServo.write(0); 
  currentAngle = 0;
  targetAngle = 0;
}

void loop() {
  int zone1 = analogRead(LDR_ZONE1);
  int zone2 = analogRead(LDR_ZONE2);
  int zone3 = analogRead(LDR_ZONE3);

  bool alert1 = zone1 >= THRESHOLD_ZONE1;
  bool alert2 = zone2 >= THRESHOLD_ZONE2;
  bool alert3 = zone3 >= THRESHOLD_ZONE3;

  // Send signals to the Nano for the LED indicators
  digitalWrite(ALERT_PIN, alert1 ? HIGH : LOW);
  digitalWrite(ZONE2_SIGNAL, alert2 ? HIGH : LOW);
  digitalWrite(ZONE3_SIGNAL, alert3 ? HIGH : LOW);

  // Determine the target angle based on which zone is broken
  if (alert1) {
    targetAngle = 30;   
  } 
  else if (alert2) {
    targetAngle = 90;   
  } 
  else if (alert3) {
    targetAngle = 120;  
  } 
  else {
    targetAngle = 0;    // Return home when clear
  }

  // Smoothly glide the servo toward the target angle step by step
  if (currentAngle < targetAngle) {
    currentAngle++;
    pointerServo.write(currentAngle);
    delay(5); // Adjust this delay to change speed (higher = slower)
  } 
  else if (currentAngle > targetAngle) {
    currentAngle--;
    pointerServo.write(currentAngle);
    delay(5); // Adjust this delay to change speed (higher = slower)
  }

  // Serial Monitor Debugging
  Serial.print("Zone 1: ");
  Serial.print(zone1);
  Serial.print(alert1 ? " [ALERT]" : " [CLEAR]");

  Serial.print(" | Zone 2: ");
  Serial.print(zone2);
  Serial.print(alert2 ? " [ALERT]" : " [CLEAR]");
  
  Serial.print(" | Zone 3: ");
  Serial.print(zone3);
  Serial.print(alert3 ? " [ALERT]" : " [CLEAR]");
  
  Serial.print(" | Target: ");
  Serial.print(targetAngle);
  Serial.print(" | Current Angle: ");
  Serial.println(currentAngle);

  delay(5); // Small loop delay
}
