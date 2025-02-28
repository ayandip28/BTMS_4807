#include <Wire.h>
#include <LiquidCrystal_I2C.h>

const int lm35Pin = 34;
const int coolingRelayPin = 25;
const int heatingRelayPin = 32;

// Initialize the LCD (change address if needed)
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(115200);
  lcd.begin();
  lcd.backlight();

  pinMode(lm35Pin, INPUT);
  pinMode(coolingRelayPin, OUTPUT);
  pinMode(heatingRelayPin, OUTPUT);

  digitalWrite(coolingRelayPin, LOW); // Ensure cooling relay is off initially
  digitalWrite(heatingRelayPin, LOW); // Ensure heating relay is off initially
}

void loop() {
  int sensorValue = analogRead(lm35Pin);
  float temperature = (sensorValue / 4095.0) * 3300;
  temperature = temperature / 10; // LM35 gives 10mV per degree Celsius

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(temperature);
  lcd.print(" C  ");

  if (temperature > 45.0) {
    digitalWrite(coolingRelayPin, HIGH); // Turn on the cooling relay
    lcd.setCursor(0, 1);
    lcd.print("Cooling ON ");
  } else if (temperature < 30.0) {
    digitalWrite(coolingRelayPin, LOW); // Turn off the cooling relay
  }

  if (temperature < 10.0) {
    digitalWrite(heatingRelayPin, HIGH); // Turn on the heating relay
    lcd.setCursor(0, 1);
    lcd.print("Heating ON ");
  } else if (temperature > 10.0) {
    digitalWrite(heatingRelayPin, LOW); // Turn off the heating relay
  }

  // Ensure only one message is shown at a time
  if (temperature <= 45.0 && temperature >= 10.0) {
    lcd.setCursor(0, 1);
    lcd.print("System STABLE ");
  }

  delay(1000); // Wait for a second
}
