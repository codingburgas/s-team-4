#include <Servo.h>

Servo barrierServo;

const int trig1 = 9;
const int echo1 = 10;


const int trig2 = 7;
const int echo2 = 8;


const int trigBarrier = 12;
const int echoBarrier = 13;


const int green1 = 2;
const int red1 = 3;

const int green2 = 5;
const int red2 = 6;


const int buzzer = 4;


const int servoPin = 11;

float measureDistance(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000);

  if (duration == 0) return -1;

  return duration * 0.0343 / 2.0;
}

void setup() {
  Serial.begin(9600);

  pinMode(trig1, OUTPUT);
  pinMode(echo1, INPUT);

  pinMode(trig2, OUTPUT);
  pinMode(echo2, INPUT);

  pinMode(trigBarrier, OUTPUT);
  pinMode(echoBarrier, INPUT);

  pinMode(green1, OUTPUT);
  pinMode(red1, OUTPUT);

  pinMode(green2, OUTPUT);
  pinMode(red2, OUTPUT);

  pinMode(buzzer, OUTPUT);

  barrierServo.attach(servoPin);
  barrierServo.write(0);
}

void loop() {

  float d1 = measureDistance(trig1, echo1);
  delay(50);

  float d2 = measureDistance(trig2, echo2);
  delay(50);

  float dBarrier = measureDistance(trigBarrier, echoBarrier);
  delay(50);

  
  if (dBarrier > 0 && dBarrier < 15) {
    barrierServo.write(90);
    delay(3000);
    barrierServo.write(0);
  }

  bool p1_free = (d1 > 50);
  bool p2_free = (d2 > 50);

  bool beep = false;

  if (p1_free) {
    digitalWrite(green1, HIGH);
    digitalWrite(red1, LOW);
  } else {
    digitalWrite(green1, LOW);
    digitalWrite(red1, HIGH);

    if (d1 > 15) {
      beep = true;
    }
  }

  if (p2_free) {
    digitalWrite(green2, HIGH);
    digitalWrite(red2, LOW);
  } else {
    digitalWrite(green2, LOW);
    digitalWrite(red2, HIGH);

    if (d2 > 15) {
      beep = true;
    }
  }


  if (beep) {
    tone(buzzer, 2000);
    delay(200);
    noTone(buzzer);
  } else {
    noTone(buzzer);
  }

  Serial.print(p1_free ? "FREE" : "FULL");
  Serial.print(",");

  Serial.print(p2_free ? "FREE" : "FULL");
  Serial.print(",");

  Serial.println(dBarrier < 15 ? "OPEN" : "CLOSED");

  delay(200);
}
