#include <BluetoothSerial.h>
#include <DHT.h>

// === CONFIGURARE PINI ===
#define ECG_PIN 35
#define PULS_PIN 34
#define DHTPIN 4
#define DHTTYPE DHT11

BluetoothSerial SerialBT;
DHT dht(DHTPIN, DHTTYPE);

// === ECG ===
const int numSamples = 10;
int ecgBuffer[numSamples];
int ecgIndex = 0;

// === Puls ===
int pulsPrag = 1900;
unsigned long timpAnterior = 0;
bool detectatPuls = false;
int bpm = 0;

// === Timp transmitere BT ===
unsigned long lastSend = 0;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("ESP32-Wearable");
  dht.begin();

  for (int i = 0; i < numSamples; i++) {
    ecgBuffer[i] = 0;
  }

  Serial.println("🟢 Sistem pornit: citire senzori + Bluetooth activ");
}

void loop() {
  // === 1. Citim DHT11 ===
  float temperatura = dht.readTemperature();
  float umiditate = dht.readHumidity();

  if (isnan(temperatura) || isnan(umiditate)) {
    temperatura = 0;
    umiditate = 0;
  }

  // === 2. Citim puls optic și calcul BPM ===
  int valoarePuls = analogRead(PULS_PIN);
  unsigned long timpCurent = millis();

  if (valoarePuls > pulsPrag && !detectatPuls) {
    unsigned long interval = timpCurent - timpAnterior;
    timpAnterior = timpCurent;
    if (interval > 300 && interval < 2000) {
      bpm = 60000 / interval;
    }
    detectatPuls = true;
  }
  if (valoarePuls < pulsPrag - 50 && detectatPuls) {
    detectatPuls = false;
  }

  // === 3. Citim și filtrăm semnalul ECG ===
  int ecgRaw = analogRead(ECG_PIN);
  ecgBuffer[ecgIndex] = ecgRaw;
  ecgIndex = (ecgIndex + 1) % numSamples;

  int ecgSum = 0;
  for (int i = 0; i < numSamples; i++) {
    ecgSum += ecgBuffer[i];
  }
  int ecgFiltrat = ecgSum / numSamples;

  // === 4. Debug pe Serial Monitor === 
  Serial.print("🌡️ Temp: "); Serial.print(temperatura); Serial.print("°C | ");
  Serial.print("💧 Umid: "); Serial.print(umiditate); Serial.print("% | ");
  Serial.print("❤️ Puls: "); Serial.print(bpm); Serial.print(" BPM | ");
  Serial.print("🧠 ECG filtrat: "); Serial.println(ecgFiltrat);
  Serial.println(ecgRaw);

  // === 5. Transmitere prin Bluetooth ===
  if (millis() - lastSend > 200) {
  String json = "{";
  json += "\"temperature\":" + String(temperatura, 1) + ",";
  json += "\"humidity\":" + String(umiditate, 1) + ",";
  json += "\"heart_rate\":" + String(bpm) + ",";
  json += "\"ekg_signal\":" + String(ecgFiltrat);
  json += "}";

  SerialBT.println(json);      // trimite la aplicație
  Serial.println(json);        // vezi și local
  lastSend = millis();
}

  delay(20);  // ajustabil
}
