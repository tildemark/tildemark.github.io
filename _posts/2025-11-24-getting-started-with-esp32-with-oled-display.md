---
title: Getting Started with ESP32 and 1.3" OLED Display
date: 2025-11-25 12:00:00 +0800
categories: [IoT, ESP32]
tags: [esp32, oled, display, i2c, adafruit, platformio]
pin: false
math: true
mermaid: true
image:
  path: https://res.cloudinary.com/dxsshynmg/image/upload/esp32-with-oled_gwgll9.webp
  alt: esp32 with oled display
---

Welcome to this guide on connecting a **1.3-inch OLED display (128x64)** to an **ESP32 microcontroller**. This project demonstrates how to set up the hardware, configure the necessary libraries, and get your display running using either Arduino IDE or PlatformIO.

> **Repo Reference:** You can find the full source code and diagrams in the [GitHub Repository](https://github.com/tildemark/ESP32-OLED-GME12864-78).
{: .prompt-info }

## Hardware Requirements

To follow along, you will need the following components:

* **ESP32 Development Board** (e.g., CH340C, USB Type C)
* **ESP32 Expansion Board** (Optional, but recommended for easier wiring)
* **1.3" OLED Display** (White, 128x64 resolution, usually SH1106 or SH110X driver)
* **Jumper Wires** (Female-to-Female or Male-to-Female depending on your board)
* **Breadboard**

## Wiring the Display

This project uses the **I2C protocol**, which simplifies wiring to just four connections. 

| OLED Pin | ESP32 Pin | Description |
|:--------:|:---------:|:-----------:|
| **GND** | GND       | Ground      |
| **VCC** | 3.3V / 5V | Power       |
| **SDA** | GPIO 21   | Data        |
| **SCL** | GPIO 22   | Clock       |

![Wiring Diagram](https://github.com/tildemark/ESP32-OLED-GME12864-78/raw/main/wiring-diagram.jpg)
_Figure 1: Connection diagram for ESP32 and OLED_

## Software Setup

You can choose between **Arduino IDE** or **VS Code with PlatformIO**.

### Option A: Arduino IDE

1.  Open the Arduino IDE.
2.  Navigate to **Sketch** > **Include Library** > **Manage Libraries**.
3.  Search for and install the following:
    * `Adafruit SH110X`
    * `Adafruit GFX Library`
4.  Copy the code from the repository's `src/main.cpp` into your sketch.

### Option B: PlatformIO (VS Code)

If you are using PlatformIO, ensure your `platformio.ini` file includes the required dependencies.

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps =
    adafruit/Adafruit SH110X @ ^2.1.8
    adafruit/Adafruit GFX Library @ ^1.11.5
````

{: .nolineno file="platformio.ini" }

## The Code

The core logic utilizes the Adafruit libraries to handle the graphics. Here is a conceptual snippet of how the initialization looks in `src/main.cpp`:

```cpp
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>

/* 1.3" OLED usually uses I2C Address 0x3C */
#define i2c_Address 0x3c 

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1   

Adafruit_SH1106G display = Adafruit_SH1106G(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void setup() {
  Serial.begin(115200);
  
  // Initialize the display
  display.begin(i2c_Address, true);
  display.display();
  display.clearDisplay();
  
  // Text settings
  display.setTextSize(1);
  display.setTextColor(SH110X_WHITE);
  display.setCursor(0, 0);
  display.println("Hello ESP32!");
  display.display();
}

void loop() {
  // Main loop logic
}
```

{: file="src/main.cpp" }

> **Note:** The 1.3" OLEDs often use the **SH1106** driver rather than the SSD1306 found in smaller 0.96" displays. The `Adafruit_SH110X` library handles this perfectly.
> {: .prompt-warning }

## Simulation

Don't have the hardware yet? You can simulate this project online using Wokwi.

  * [Wokwi Simulation](https://github.com/tildemark/ESP32-OLED-GME12864-78/wiki/Wokwi-Simulation)
  * [VS Code Extension Guide](https://github.com/tildemark/ESP32-OLED-GME12864-78/wiki/Wokwi-VS-Code-Extension)

## Troubleshooting

If your display remains black:

1.  **Check Wiring:** Ensure SDA and SCL are not swapped.
2.  **I2C Address:** Verify if your display uses `0x3C` or `0x3D` using an I2C scanner sketch.
3.  **Contrast:** Sometimes specific initialization commands are needed for different OLED panels.

## Conclusion

This setup provides a robust foundation for building IoT dashboards, status monitors, or simple games on the ESP32.

Check out the [Demo Video on YouTube](https://www.youtube.com/shorts/dMohdxYCU6c) to see it in action\!

-----

*Found this guide helpful? Star the [repository](https://github.com/tildemark/ESP32-OLED-GME12864-78) to support the project\!*

