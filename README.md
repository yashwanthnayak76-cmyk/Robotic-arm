# Robotic-arm
This my first Git repositry
The code to control 3D robotic hand 
#include <Servo.h>

// Define servo objects for each finger
Servo thumbServo;
Servo indexServo;
Servo middleServo;
Servo ringServo;
Servo pinkyServo;

// Define servo pins
const int THUMB_PIN = 3;
const int INDEX_PIN = 5;
const int MIDDLE_PIN = 6;
const int RING_PIN = 9;
const int PINKY_PIN = 10;

// Define flex sensor pins (optional - for glove control)
const int THUMB_FLEX = A0;
const int INDEX_FLEX = A1;
const int MIDDLE_FLEX = A2;
const int RING_FLEX = A3;
const int PINKY_FLEX = A4;

// Calibration values
int flexMin[5] = {200, 200, 200, 200, 200};  // Minimum flex sensor values
int flexMax[5] = {600, 600, 600, 600, 600};  // Maximum flex sensor values
int servoMin = 0;    // Minimum servo angle
int servoMax = 180;  // Maximum servo angle

void setup() {
  Serial.begin(9600);
  
  // Attach servos to pins
  thumbServo.attach(THUMB_PIN);
  indexServo.attach(INDEX_PIN);
  middleServo.attach(MIDDLE_PIN);
  ringServo.attach(RING_PIN);
  pinkyServo.attach(PINKY_PIN);
  
  // Initialize all fingers to open position
  setAllFingers(0);
  
  Serial.println("Robotic Hand Initialized");
  Serial.println("Commands:");
  Serial.println("O - Open all fingers");
  Serial.println("C - Close all fingers");
  Serial.println("W - Wave gesture");
  Serial.println("P - Point gesture");
  Serial.println("G - Grab gesture");
  Serial.println("R - Rock gesture");
}

void loop() {
  // Check for serial commands
  if (Serial.available() > 0) {
    char command = Serial.read();
    executeCommand(command);
  }
  
  // Optional: Read flex sensors for glove control
  // Uncomment if using flex sensors
  /*
  readFlexSensors();
  */
  
  delay(50);
}

void executeCommand(char cmd) {
  switch(toupper(cmd)) {
    case 'O':
      openHand();
      break;
    case 'C':
      closeHand();
      break;
    case 'W':
      waveGesture();
      break;
    case 'P':
      pointGesture();
      break;
    case 'G':
      grabGesture();
      break;
    case 'R':
      rockGesture();
      break;
    default:
      Serial.println("Unknown command");
  }
}

void setAllFingers(int angle) {
  thumbServo.write(angle);
  indexServo.write(angle);
  middleServo.write(angle);
  ringServo.write(angle);
  pinkyServo.write(angle);
}

void openHand() {
  Serial.println("Opening hand...");
  setAllFingers(0);
}

void closeHand() {
  Serial.println("Closing hand...");
  setAllFingers(180);
}

void waveGesture() {
  Serial.println("Waving...");
  for(int i = 0; i < 3; i++) {
    setAllFingers(0);
    delay(500);
    setAllFingers(90);
    delay(500);
  }
  setAllFingers(0);
}

void pointGesture() {
  Serial.println("Pointing...");
  thumbServo.write(180);
  indexServo.write(0);
  middleServo.write(180);
  ringServo.write(180);
  pinkyServo.write(180);
}

void grabGesture() {
  Serial.println("Grabbing...");
  for(int i = 0; i <= 180; i += 5) {
    setAllFingers(i);
    delay(20);
  }
  delay(1000);
  for(int i = 180; i >= 0; i -= 5) {
    setAllFingers(i);
    delay(20);
  }
}

void rockGesture() {
  Serial.println("Rock on!");
  thumbServo.write(180);
  indexServo.write(0);
  middleServo.write(180);
  ringServo.write(180);
  pinkyServo.write(0);
}

void readFlexSensors() {
  // Read and map flex sensor values to servo positions
  int thumbFlex = analogRead(THUMB_FLEX);
  int indexFlex = analogRead(INDEX_FLEX);
  int middleFlex = analogRead(MIDDLE_FLEX);
  int ringFlex = analogRead(RING_FLEX);
  int pinkyFlex = analogRead(PINKY_FLEX);
  
  // Map sensor values to servo angles
  int thumbAngle = map(thumbFlex, flexMin[0], flexMax[0], servoMin, servoMax);
  int indexAngle = map(indexFlex, flexMin[1], flexMax[1], servoMin, servoMax);
  int middleAngle = map(middleFlex, flexMin[2], flexMax[2], servoMin, servoMax);
  int ringAngle = map(ringFlex, flexMin[3], flexMax[3], servoMin, servoMax);
  int pinkyAngle = map(pinkyFlex, flexMin[4], flexMax[4], servoMin, servoMax);
  
  // Constrain angles to valid range
  thumbAngle = constrain(thumbAngle, servoMin, servoMax);
  indexAngle = constrain(indexAngle, servoMin, servoMax);
  middleAngle = constrain(middleAngle, servoMin, servoMax);
  ringAngle = constrain(ringAngle, servoMin, servoMax);
  pinkyAngle = constrain(pinkyAngle, servoMin, servoMax);
  
  // Set servo positions
  thumbServo.write(thumbAngle);
  indexServo.write(indexAngle);
  middleServo.write(middleAngle);
  ringServo.write(ringAngle);
  pinkyServo.write(pinkyAngle);
}

// Calibration function for flex sensors
void calibrateFlexSensors() {
  Serial.println("Calibrating flex sensors...");
  Serial.println("Make a fist and press any key");
  
  while(!Serial.available()) {}
  Serial.read();
  
  flexMin[0] = analogRead(THUMB_FLEX);
  flexMin[1] = analogRead(INDEX_FLEX);
  flexMin[2] = analogRead(MIDDLE_FLEX);
  flexMin[3] = analogRead(RING_FLEX);
  flexMin[4] = analogRead(PINKY_FLEX);
  
  Serial.println("Open your hand and press any key");
  
  while(!Serial.available()) {}
  Serial.read();
  
  flexMax[0] = analogRead(THUMB_FLEX);
  flexMax[1] = analogRead(INDEX_FLEX);
  flexMax[2] = analogRead(MIDDLE_FLEX);
  flexMax[3] = analogRead(RING_FLEX);
  flexMax[4] = analogRead(PINKY_FLEX);
  
  Serial.println("Calibration complete!");
}

