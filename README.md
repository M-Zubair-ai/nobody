   // Motor Driver Pins 
const int AIN1 = 2; 
const int AIN2 = 4; 
const int PWMA = 5; 
const int BIN1 = 7; 
const int BIN2 = 8; 
const int PWMB = 6; 
const int STBY = 9; 
 
// IR Sensor Pins 
const int IR1 = A0;  // Left-most 
const int IR2 = A1; 
const int IR3 = 3;   // Center 
const int IR4 = 10; 
const int IR5 = 11;  // Right-most 
 
// Ultrasonic Sensor Pins 
const int trigPin = A3; 
const int echoPin = A2; 
 
// Speed setting (0 - 255) 
const int MOTOR_SPEED = 100; 
 
// Distance threshold in cm 
const float OBSTACLE_DISTANCE = 20.0; 
 
void setup() { 
  pinMode(AIN1, OUTPUT); 
  pinMode(AIN2, OUTPUT); 
  pinMode(PWMA, OUTPUT); 
  pinMode(BIN1, OUTPUT); 
  pinMode(BIN2, OUTPUT); 
  pinMode(PWMB, OUTPUT); 
  pinMode(STBY, OUTPUT); 
 
  pinMode(IR1, INPUT); 
  pinMode(IR2, INPUT); 
  pinMode(IR3, INPUT); 
  pinMode(IR4, INPUT); 
  pinMode(IR5, INPUT); 
 
  pinMode(trigPin, OUTPUT); 
  pinMode(echoPin, INPUT); 
 
  Serial.begin(9600); 
} 
 
void loop() { 
  float distance = readUltrasonicDistance(); 
 
  if (distance > 0 && distance < OBSTACLE_DISTANCE) { 
    // Object detected, turn 180° until line detected on middle IR sensor 
    turn180UntilLine(); 
  } else { 
    // Normal line following behavior 
    lineFollow(); 
  } 
} 
 
// Function to read distance from ultrasonic sensor (in cm) 
float readUltrasonicDistance() { 
  digitalWrite(trigPin, LOW); 
  delayMicroseconds(2); 
 
  digitalWrite(trigPin, HIGH); 
  delayMicroseconds(10); 
  digitalWrite(trigPin, LOW); 
 
  long duration = pulseIn(echoPin, HIGH, 30000); // timeout 30ms 
  if (duration == 0) return -1; // no echo 
 
  float distanceCm = (duration * 0.0343) / 2; 
  return distanceCm; 
} 
 
void turn180UntilLine() { 
  Serial.println("Object detected! Turning 180 degrees..."); 
  digitalWrite(STBY, HIGH); // Enable motors 
 
  // Turn in place: left motor forward, right motor backward 
  digitalWrite(AIN1, HIGH); 
  digitalWrite(AIN2, LOW); 
  analogWrite(PWMA, MOTOR_SPEED); 
 
  digitalWrite(BIN1, LOW); 
  digitalWrite(BIN2, HIGH); 
  analogWrite(PWMB, MOTOR_SPEED); 
 
  // Keep turning until middle sensor (IR3) detects black (LOW) 
  while (digitalRead(IR3) == HIGH) { 
    // Just keep turning 
    delay(10); 
  } 
 
  stopMotors(); 
  Serial.println("Black line detected, resuming line following."); 
} 
 
void lineFollow() { 
  int s1 = digitalRead(IR1); 
  int s2 = digitalRead(IR2); 
  int s3 = digitalRead(IR3); 
  int s4 = digitalRead(IR4); 
  int s5 = digitalRead(IR5); 
 
  digitalWrite(STBY, HIGH); // Enable motors 
 
  // Follow the black line: LOW = black, HIGH = white 
  if (s3 == LOW && s1 == HIGH && s2 == HIGH && s4 == HIGH && s5 == HIGH) { 
    // Only center sensor on black → go straight 
    forward(); 
  }  
  else if (s1 == LOW || s2 == LOW) { 
    // Far left or mid-left sensors see black → turn right 
    turnRight(); 
  } 
  else if (s4 == LOW || s5 == LOW) { 
    // Far right or mid-right sensors see black → turn left 
    turnLeft(); 
  }  
  else { 
    // No sensor sees black → stop 
    stopMotors(); 
  } 
} 
 
void forward() { 
  digitalWrite(AIN1, HIGH); 
  digitalWrite(AIN2, LOW); 
  analogWrite(PWMA, MOTOR_SPEED); 
 
  digitalWrite(BIN1, HIGH); 
  digitalWrite(BIN2, LOW); 
  analogWrite(PWMB, MOTOR_SPEED); 
} 
 
void turnLeft() { 
  // Left motor ON, right motor OFF → turn left 
  digitalWrite(AIN1, HIGH); 
  digitalWrite(AIN2, LOW); 
  analogWrite(PWMA, MOTOR_SPEED); 
 
  digitalWrite(BIN1, LOW); 
  digitalWrite(BIN2, LOW); 
  analogWrite(PWMB, 0); 
} 
 
void turnRight() { 
  // Right motor ON, left motor OFF → turn right 
  digitalWrite(AIN1, LOW); 
  digitalWrite(AIN2, LOW); 
  analogWrite(PWMA, 0); 
 
  digitalWrite(BIN1, HIGH); 
  digitalWrite(BIN2, LOW); 
  analogWrite(PWMB, MOTOR_SPEED); 
} 
 
void stopMotors() { 
  digitalWrite(AIN1, LOW); 
  digitalWrite(AIN2, LOW); 
  analogWrite(PWMA, 0); 
 
  digitalWrite(BIN1, LOW); 
  digitalWrite(BIN2, LOW); 
  analogWrite(PWMB, 0); 
 
  digitalWrite(STBY, LOW); // Disable motors 
}
