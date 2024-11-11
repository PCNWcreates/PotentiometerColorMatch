#include <Adafruit_NeoPixel.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Constants
#define LED_PIN 6
#define NUM_LEDS 10
#define POT_PIN A0
#define BRIGHTNESS 128 // 50% of 255
#define FULL_BRIGHTNESS 255 // 100% of 255
#define DEAD_ZONE 10 // Dead zone for potentiometer value to prevent drifting

// Create objects
Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);
LiquidCrystal_I2C lcd(0x27, 20, 4); // Adjust the address (0x27) according to your LCD

// Color names and values
const char* colorNames[NUM_LEDS] = {"Red", "Orange", "Yellow", "Green", "Blue", "Purple", "Teal", "Pink", "Lime", "Magenta"};
uint32_t colorValues[NUM_LEDS] = {
    strip.Color(255, 0, 0),    // Red
    strip.Color(255, 165, 0),  // Orange
    strip.Color(255, 255, 0),  // Yellow
    strip.Color(0, 255, 0),    // Green
    strip.Color(0, 0, 255),    // Blue
    strip.Color(128, 0, 128),  // Purple
    strip.Color(0, 128, 128),  // Teal
    strip.Color(255, 20, 147), // Pink
    strip.Color(50, 205, 50),  // Lime
    strip.Color(255, 0, 255)   // Magenta
};

// Variables to hold the last known state
int lastColorIndex = -1;
int lastPotValue = 0;
int maxPotValue = 0;

void setup() {
  strip.begin();
  strip.show(); // Initialize all pixels to 'off'

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Selected Color:");

  // Find the maximum potentiometer value at startup
  for (int i = 0; i < 100; i++) {
    int potValue = analogRead(POT_PIN);
    if (potValue > maxPotValue) {
      maxPotValue = potValue;
    }
    delay(10);
  }
}

void loop() {
  int potValue = analogRead(POT_PIN);
  // Adjust the map range to fit the potentiometer's actual range
  int colorIndex = map(potValue, 0, maxPotValue, 0, NUM_LEDS - 1);

  // Constrain the colorIndex to ensure it stays within the limits of 'Red' and 'Magenta'
  colorIndex = constrain(colorIndex, 0, NUM_LEDS - 1);

  // Only update if the potentiometer value has changed significantly
  if (abs(potValue - lastPotValue) > DEAD_ZONE) {
    lastPotValue = potValue;
    lastColorIndex = colorIndex;
    updateLCD(colorIndex);
    updateLEDs(colorIndex);
  }

  delay(100); // Small delay to reduce flicker
}

void updateLCD(int index) {
  static int currentIndex = -1;
  if (currentIndex != index) {
    currentIndex = index;
    lcd.setCursor(0, 1);
    lcd.print("                "); // Clear previous color name
    lcd.setCursor(0, 1);
    lcd.print(colorNames[index]); // Display the selected color name
  }
}

void updateLEDs(int index) {
  for (int i = 0; i < NUM_LEDS; i++) {
    if (i == index) {
      strip.setPixelColor(i, colorValues[i]); // Full brightness for selected color
      strip.setBrightness(FULL_BRIGHTNESS);
    } else {
      uint8_t r = (colorValues[i] >> 16) & 0xFF;
      uint8_t g = (colorValues[i] >> 8) & 0xFF;
      uint8_t b = colorValues[i] & 0xFF;
      // Set the other LEDs to 50% brightness
      strip.setPixelColor(i, strip.Color(r * BRIGHTNESS / 255, g * BRIGHTNESS / 255, b * BRIGHTNESS / 255));
    }
  }
  strip.show();
}
