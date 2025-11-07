# Heat-stroke-protection-vest-project-code
This project has been developed to prevent heat strokes during hot weather.
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <OneWire.h>
#include <DallasTemperature.h>

// === Pin Definitions ===
#define ONE_WIRE_BUS 2       // DS18B20
#define PULSE_PIN A0         // Pulse sensor
#define RELAY_PIN 8          // Relay control
#define RELAY_PIN1 9          // Relay control

// === LCD Setup ===
LiquidCrystal_I2C lcd(0x27, 16, 2);  // (Address, Columns, Rows)

// === DS18B20 Setup ===
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// === Pulse Variables ===
int Signal;
int threshold = 550; // Adjust this depending on your sensor output
bool pulseDetected = false;
unsigned long lastBeatTime = 0;
int beatCount = 0;
float bpm = 0;

// === Timing ===
unsigned long lastDisplayTime = 0;
unsigned long lastCheckTime = 0;

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
    pinMode(RELAY_PIN1, OUTPUT);

  digitalWrite(RELAY_PIN, LOW);
  digitalWrite(RELAY_PIN1, LOW);

  lcd.init();
  lcd.backlight();

  sensors.begin();

  lcd.setCursor(0, 0);
  lcd.print("Health Monitor");
  delay(2000);
  lcd.clear();
}

void loop() {
  // === Temperature Reading ===
  sensors.requestTemperatures();
  float temperature = sensors.getTempCByIndex(0);

  // === Pulse Reading ===
  Signal = analogRead(PULSE_PIN);
  unsigned long currentTime = millis();

  if (Signal > threshold && !pulseDetected) {
    pulseDetected = true;
    unsigned long beatInterval = currentTime - lastBeatTime;
    lastBeatTime = currentTime;

    if (beatInterval > 300 && beatInterval < 2000) {  // valid heart beat range
      bpm = 60000.0 / beatInterval;
    }
  }

  if (Signal < threshold) pulseDetected = false;

  // === Display Update Every 500ms ===
  if (millis() - lastDisplayTime > 500) {
    lastDisplayTime = millis();

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temperature, 1);
    lcd.print((char)223); // degree symbol
    lcd.print("C");

    lcd.setCursor(0, 1);
    lcd.print("Pulse: ");
    lcd.print((int)bpm);
    lcd.print(" bpm");
  }

  // === Relay Control ===
  if (temperature > 36.7 || bpm > 120 || bpm < 60) {
    digitalWrite(RELAY_PIN, HIGH);  // Turn relay ON
        digitalWrite(RELAY_PIN1, HIGH);  // Turn relay ON

  } else {
    digitalWrite(RELAY_PIN, LOW);   // Turn relay OFF
        digitalWrite(RELAY_PIN1, LOW);   // Turn relay OFF

  }
}
