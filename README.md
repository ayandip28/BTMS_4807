#include <Wire.h>
#include <LiquidCrystal_I2C.h>

const int lm35Pin = 34;
const int currentSensorPin = 35;
const int voltageSensorPin = 36;
const int coolingRelayPin = 25;
const int heatingRelayPin = 32;

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(115200);
  lcd.begin(16, 2);
  lcd.backlight();

  pinMode(lm35Pin, INPUT);
  pinMode(currentSensorPin, INPUT);
  pinMode(voltageSensorPin, INPUT);
  pinMode(coolingRelayPin, OUTPUT);
  pinMode(heatingRelayPin, OUTPUT);

  digitalWrite(coolingRelayPin, LOW);
  digitalWrite(heatingRelayPin, LOW);
}

void loop() {
  // Read temperature sensor
  int tempValue = analogRead(lm35Pin);
  float temperature = (tempValue / 4095.0) * 3300;
  temperature = temperature / 10;  // LM35 gives 10mV per degree Celsius

  // Read current sensor
  int currentValue = analogRead(currentSensorPin);
  float current = (currentValue / 4095.0) * 3.3; // Adjust based on the sensor's specifications

  // Read voltage sensor
  int voltageValue = analogRead(voltageSensorPin);
  float voltage = (voltageValue / 4095.0) * 3.3; // Adjust based on the sensor's specifications

  // Calculate power (P = V * I)
  float power = voltage * current;

  // Calculate charging cycles (example calculation)
  static float totalEnergy = 0;
  totalEnergy += power * (1.0 / 3600); // Assuming loop runs every second
  float chargingCycles = totalEnergy / (batteryCapacity * voltage); // batteryCapacity in Ah

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  Serial.print("Current: ");
  Serial.print(current);
  Serial.println(" A");

  Serial.print("Voltage: ");
  Serial.print(voltage);
  Serial.println(" V");

  Serial.print("Power: ");
  Serial.print(power);
  Serial.println(" W");

  Serial.print("Charging Cycles: ");
  Serial.println(chargingCycles);

  // Display temperature on LCD
  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(temperature);
  lcd.print(" C  ");

  // Control relays based on temperature
  if (temperature > 45.0) {
    digitalWrite(coolingRelayPin, HIGH);
    lcd.setCursor(0, 1);
    lcd.print("Cooling ON ");
  } else if (temperature < 30.0) {
    digitalWrite(coolingRelayPin, LOW);
  }

  if (temperature < 10.0) {
    digitalWrite(heatingRelayPin, HIGH);
    lcd.setCursor(0, 1);
    lcd.print("Heating ON ");
  } else if (temperature > 10.0) {
    digitalWrite(heatingRelayPin, LOW);
  }

  // Ensure only one message is shown at a time
  if (temperature <= 45.0 && temperature >= 10.0) {
    lcd.setCursor(0, 1);
    lcd.print("System STABLE ");
  }

  delay(1000); // Wait for a second
}
