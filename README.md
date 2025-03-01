# Animal-detection-and-monitoring
#include <Wire.h>
#include <RTClib.h>
#include <SD.h>

#define PIR_PIN 2
#define TRIG_PIN 3
#define ECHO_PIN 4
#define BUZZER_PIN 5
#define LED_PIN 6
#define SD_CS_PIN 10

RTC_DS3231 rtc;
File dataFile;

void setup() {
  pinMode(PIR_PIN, INPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  Serial.begin(9600);
  Wire.begin();

  if (!rtc.begin()) {
    Serial.println("Couldn't find RTC");
    while (1);
  }

  if (!SD.begin(SD_CS_PIN)) {
    Serial.println("SD card initialization failed!");
    return;
  }
}

void loop() {
  // PIR Sensor
  if (digitalRead(PIR_PIN) == HIGH) {
    Serial.println("Motion detected");
    logEvent("Motion detected");
    activateAlert();
  }

  // Ultrasonic Sensor
  long duration, distance;
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = (duration / 2) / 29.1; // Convert to cm

  if (distance > 0 && distance < 100) { // Example threshold
    Serial.print("animal detected at ");
    Serial.print(distance);
    Serial.println(" cm");
    logEvent("animal detected at " + String(distance) + " cm");
    activateAlert();
  }

  delay(1000);
}

void activateAlert() {
  digitalWrite(BUZZER_PIN, HIGH);
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(LED_PIN, LOW);
}

void logEvent(String event) {
  if (SD.exists("datalog.txt")) {
    dataFile = SD.open("datalog.txt", FILE_WRITE);
    if (dataFile) {
      DateTime now = rtc.now();
      dataFile.print(now.timestamp());
      dataFile.print(": ");
      dataFile.println(event);
      dataFile.close();
    }
  }
}

