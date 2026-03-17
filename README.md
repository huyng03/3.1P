# 3.1P
#include <WiFiNINA.h>

// Replace with your Wi-Fi credentials and IFTTT key
char ssid[] = "Unit 1110";
char pass[] = "93fdc790e201af09";
const char* IFTTT_KEY = "your_ifttt_webhook_key"; // replace this with your actual key

WiFiClient client;

const char* host = "maker.ifttt.com";  // ✅ Correct host
const int port = 80;

const int lightSensorPin = A0;
const int lightThreshold = 500;  // Adjust this value based on your LDR readings

bool sunlightPresent = false;

void setup() {
  Serial.begin(9600);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(2000);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
}

void loop() {
  int lightValue = analogRead(lightSensorPin);
  Serial.print("Light Value: ");
  Serial.println(lightValue);

  bool currentSunlight = lightValue > lightThreshold;

  if (currentSunlight != sunlightPresent) {
    sunlightPresent = currentSunlight;
    String event = sunlightPresent ? "sunlightStart" : "sunlightEnd"; // Use your actual IFTTT event names
    sendIFTTT(event);
  }

  delay(60000); // Check every 60 seconds
}

void sendIFTTT(String event) {
  if (client.connect(host, port)) {
    String url = "/trigger/" + event + "/with/key/" + String(IFTTT_KEY);
    
    client.println("GET " + url + " HTTP/1.1");
    client.println("Host: " + String(host));
    client.println("Connection: close");
    client.println();

    Serial.println("IFTTT event sent: " + event);
    delay(500);
    client.stop();
  } else {
    Serial.println("Connection to IFTTT failed.");
  }
}
