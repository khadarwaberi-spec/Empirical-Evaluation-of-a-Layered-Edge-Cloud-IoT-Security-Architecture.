#include <WiFi.h>
#include <HTTPClient.h>
#include <Keypad.h>
#include <Adafruit_Fingerprint.h>
#include <HardwareSerial.h>

const char* ssid = "ITOFFICE";
const char* password = "Citycot@16204";
const char* serverName = "http://your-server-api.com/api/security";

String deviceID = "ESP32_SEC_NODE_01";

HardwareSerial mySerial(2);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

const int relayPin = 5;
const int buzzerPin = 4;

const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {13, 12, 14, 27};
byte colPins[COLS] = {26, 25, 33, 32};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

String correctPassword = "1234";
String inputPassword = "";

void sendToCloud(String eventType) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    http.begin(serverName);
    http.addHeader("Content-Type", "application/json");

    String jsonData = "{";
    jsonData += "\"device_id\":\"" + deviceID + "\",";
    jsonData += "\"event\":\"" + eventType + "\"";
    jsonData += "}";

    http.POST(jsonData);
    http.end();
  }
}

uint8_t getFingerprintID() {
  if (finger.getImage() != FINGERPRINT_OK) return 0;
  if (finger.image2Tz() != FINGERPRINT_OK) return 0;
  if (finger.fingerSearch() != FINGERPRINT_OK) return 0;

  return finger.fingerID;
}

void setup() {
  Serial.begin(115200);

  pinMode(relayPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  digitalWrite(relayPin, LOW);
  digitalWrite(buzzerPin, LOW);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  mySerial.begin(57600, SERIAL_8N1, 16, 17);
  finger.begin(57600);
}

void loop() {

  char key = keypad.getKey();

  if (key) {
    if (key == '#') {
      if (inputPassword == correctPassword) {
        digitalWrite(relayPin, HIGH);
        sendToCloud("access_granted_keypad");
        delay(3000);
        digitalWrite(relayPin, LOW);
      } else {
        digitalWrite(buzzerPin, HIGH);
        sendToCloud("access_denied_keypad");
        delay(2000);
        digitalWrite(buzzerPin, LOW);
      }
      inputPassword = "";
    } else if (key == '*') {
      inputPassword = "";
    } else {
      inputPassword += key;
    }
  }

  uint8_t id = getFingerprintID();

  if (id != 0) {
    digitalWrite(relayPin, HIGH);
    sendToCloud("access_granted_biometric");
    delay(3000);
    digitalWrite(relayPin, LOW);
  }
}
