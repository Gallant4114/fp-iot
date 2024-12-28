# Final Project Internet of Things
## AVIS: Adaptive Ventilation IoT System 
***


### Source code (belum fix)
```cpp
#define BLYNK_TEMPLATE_ID "TMPL6-N1Yxye0"
#define BLYNK_TEMPLATE_NAME "AVIS"
#define BLYNK_DEVICE_NAME "AVIS"
#define BLYNK_AUTH_TOKEN "VwO-4r_7g1V5bGBXsgInbBDdQqpOfGmB"

#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp8266.h>
char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "AG1";
char pass[] = "1sampai8";

#include <Wire.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_GFX.h>
#include <DHT.h>
#include <MQ135.h>
#include <Servo.h>

// Konfigurasi OLED
#define OLED_WIDTH 128
#define OLED_HEIGHT 64
#define OLED_ADDR   0x3C
Adafruit_SSD1306 display(OLED_WIDTH, OLED_HEIGHT);

// Konfigurasi DHT11
#define DHTPIN D4
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

// Konfigurasi MQ135
#define PIN_MQ135 A0
MQ135 mq135_sensor(PIN_MQ135);

// Konfigurasi Servo
#define PIN_SERVO D5
Servo servo_motor;

// Konfigurasi Buzzer
#define BUZZER_PIN D6

// Variabel untuk menyimpan data sensor
float humi, temp;
float rzero, correctedRZero, resistance, ppm, correctedPPM;

// Ambang batas PPM untuk membuka/menutup jendela
const float SAFE_AIR_THRESHOLD = 5000.0;

void setup() {
  // Inisialisasi Serial Monitor
  Serial.begin(115200);

  // Inisialisasi OLED
  display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR);
  display.clearDisplay();

  // Inisialisasi DHT11
  dht.begin();

  // Inisialisasi Servo
  servo_motor.attach(PIN_SERVO);
  servo_motor.write(0); // Awal posisi tertutup (0 derajat)

  // Inisialisasi Buzzer
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW);

  delay(1000);
}

void readDHT11() {
  humi = dht.readHumidity();
  temp = dht.readTemperature();

  if (isnan(humi) || isnan(temp)) {
    Serial.println("DHT11 tidak terbaca... !");
    return;
  }

  Serial.print("Suhu: ");
  Serial.print(temp);
  Serial.println(" C");

  Serial.print("Kelembapan: ");
  Serial.print(humi);
  Serial.println(" %RH");
}

void readMQ135() {
  rzero = mq135_sensor.getRZero();
  correctedRZero = mq135_sensor.getCorrectedRZero(temp, humi);
  resistance = mq135_sensor.getResistance();
  ppm = mq135_sensor.getPPM();
  correctedPPM = mq135_sensor.getCorrectedPPM(temp, humi);

  Serial.print("PPM: ");
  Serial.println(correctedPPM);
}

void controlWindow() {
  if (correctedPPM > SAFE_AIR_THRESHOLD) {
    // Udara buruk, tutup jendela
    servo_motor.write(0); // Posisi tertutup (0 derajat)
    Serial.println("Air quality poor. Window closed.");
    digitalWrite(BUZZER_PIN, HIGH);  // Mengaktifkan buzzer
  } else {
    // Udara aman, buka jendela
    servo_motor.write(180); // Posisi terbuka (180 derajat)
    Serial.println("Air quality good. Window opened.");
    digitalWrite(BUZZER_PIN, LOW);   // Mematikan buzzer
  }
}

void displayData() {
  display.clearDisplay();

  // Menampilkan judul "AVIS"
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(0, 0);
  display.print("--- AVIS ---\n");

  // Menampilkan data DHT11 dengan jarak yang cukup
  display.setCursor(0, 10);
  display.print("Temp: ");
  display.print(temp);
  display.print(" C");

  display.setCursor(0, 20);
  display.print("Humi: ");
  display.print(humi);
  display.print(" %");

  // Menampilkan data MQ135
  display.setCursor(0, 30);
  display.print("PPM: ");
  display.print(correctedPPM);

  display.setCursor(0, 40);
  if (correctedPPM > SAFE_AIR_THRESHOLD) {
    display.print("Status: Poor");
  } else {
    display.print("Status: Good");
  }

  display.display();
}

void loop() {
  readDHT11();    // Membaca data dari sensor DHT11
  readMQ135();    // Membaca data dari sensor MQ135
  controlWindow(); // Mengontrol jendela dan buzzer
  displayData();  // Menampilkan data pada OLED

  delay(2000);    // Tunggu 2 detik sebelum pembacaan berikutnya
}
```

## Dokumentasi
1. foto
2. 
