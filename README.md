# Firmware
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <DHT.h>

// Replace these with your network credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Replace with your server URL
const char* serverUrl = "http://your-server.com/endpoint";

// DHT sensor setup
#define DHTPIN D4 // Define the pin where the data line is connected
#define DHTTYPE DHT11 // DHT 11
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  delay(10);

  // Connect to Wi-Fi
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  // Initialize DHT sensor
  dht.begin();
}

void loop() {
  // Wait a few seconds between measurements
  delay(2000);

  // Reading temperature and humidity from DHT11 sensor
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // Check if any reads failed and exit early (to try again)
  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Print the values to the Serial Monitor
  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print(" %\t");
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.println(" *C");

  // Check WiFi connection status
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    // Prepare JSON payload
    String payload = "{\"temperature\":" + String(t) + ",\"humidity\":" + String(h) + "}";

    http.begin(serverUrl); // Specify the URL
    http.addHeader("Content-Type", "application/json");

    int httpResponseCode = http.POST(payload);

    if (httpResponseCode > 0) {
      String response = http.getString(); // Get the response to the request
      Serial.println(httpResponseCode); // Print return code
      Serial.println(response); // Print request answer
    } else {
      Serial.print("Error on sending POST: ");
      Serial.println(httpResponseCode);
    }

    http.end(); // Free resources
  } else {
    Serial.println("Error in WiFi connection");
  }
}
