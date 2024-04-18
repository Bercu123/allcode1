#include "esp_camera.h"
#include <WiFi.h>

// Definirea numelor rețelei WiFi și a parolei
const char* ssid = "TP-Link_E606";
const char* password = "44332211";

// Adresa IP și portul serverului web
IPAddress serverIP(192, 168, 1, 100); // Adresa IP a serverului
WiFiServer server(80); // Portul HTTP

// Inițializarea camerei
camera_config_t config;

void setup() {
  Serial.begin(115200);

  // Conectarea la rețeaua WiFi
  Serial.println();
  Serial.println("Conectare la rețeaua WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("Conectat la rețeaua WiFi");

  // Inițializarea camerei
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d1 = 13; // Exemplu: Pinul D1 conectat la pinul 13
config.pin_d2 = 14; // Exemplu: Pinul D2 conectat la pinul 14
config.pin_d3 = 15; // Exemplu: Pinul D3 conectat la pinul 15
config.pin_d4 = 16; // Exemplu: Pinul D4 conectat la pinul 16
config.pin_d5 = 17; // Exemplu: Pinul D5 conectat la pinul 17
config.pin_d6 = 18; // Exemplu: Pinul D6 conectat la pinul 18
config.pin_d7 = 19; // Exemplu: Pinul D7 conectat la pinul 19
config.pin_xclk = 21; // Exemplu: Pinul XCLK conectat la pinul 21
config.pin_pclk = 22; // Exemplu: Pinul PCLK conectat la pinul 22
config.pin_vsync = 23; // Exemplu: Pinul VSYNC conectat la pinul 23
config.pin_href = 25; // Exemplu: Pinul HREF conectat la pinul 25
config.pin_sscb_sda = 26; // Exemplu: Pinul SIOD (SSCB_SDA) conectat la pinul 26
config.pin_sscb_scl = 27; // Exemplu: Pinul SIOC (SSCB_SCL) conectat la pinul 27
config.pin_pwdn = 32; // Exemplu: Pinul PWDN conectat la pinul 32
config.pin_reset = 33; // Exemplu: Pinul RESET conectat la pinul 33

  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  
  if(psramFound()){
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_SVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }

  // Inițializarea camerei
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  // Pornirea serverului web
  server.begin();
  Serial.println("Serverul web pornit");
}

void loop() {
  // Așteptarea unei conexiuni client
  WiFiClient client = server.available();
  if (!client) {
    return;
  }

  // Trimite antetul HTTP pentru video
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: multipart/x-mixed-replace; boundary=frame");
  client.println();

  while (client.connected()) {
    camera_fb_t * fb = esp_camera_fb_get();
    if (!fb) {
      Serial.println("Camera capture failed");
      break;
    }
    
    // Trimiterea imaginii către client
    client.println("--frame");
    client.println("Content-Type: image/jpeg");
    client.println("Content-Length: " + String(fb->len));
    client.println();
    client.write(fb->buf, fb->len);
    client.println();

    esp_camera_fb_return(fb);

    // Așteptare pentru a evita supraîncălzirea serverului
    delay(100);
  }

  // Deconectarea clientului
  client.stop();
}
