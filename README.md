# VibroVario2 - Watchy V3 with BMP390 Integration

## Overview
This is a modified version of the Watchy V3.0 board with integrated BMP390 barometric pressure sensor for variometer applications.

## Changes from Original Watchy V3.0

### Added Components
1. **BMP390** (U5) - Barometric Pressure Sensor
   - Package: LGA-10 (2.0x2.0x0.75mm)
   - I2C Address: 0x77 (SDO tied to GND)
   - Position: Near ESP32-S3

2. **Decoupling Capacitors** (per BMP390 datasheet)
   - C22: 100nF (0402) - High-frequency decoupling
   - C23: 1uF (0402) - Bulk decoupling

### Pin Connections
| BMP390 Pin | Function | ESP32-S3 GPIO | Net Name |
|------------|----------|---------------|----------|
| 1 | GND | - | GND |
| 2 | VDDIO | - | +3V3 |
| 3 | SDO | - | GND (sets I2C addr 0x77) |
| 4 | INT2 | - | NC |
| 5 | VDD | GPIO45 | BMP390_PWR |
| 6 | INT1 | - | NC |
| 7 | SCL | GPIO11 | SCL |
| 8 | SDA | GPIO12 | SDA |
| 9 | CSB | - | +3V3 (I2C mode) |
| 10 | GND | - | GND |

### Power Control
- BMP390 VDD is connected to GPIO45 for power management
- Set GPIO45 HIGH to enable BMP390
- Set GPIO45 LOW to disable BMP390 (power saving mode)

## Files Structure
```
vibrovario2/
├── vibrovario2.kicad_sch    # Schematic with BMP390
├── vibrovario2.kicad_pcb    # PCB with BMP390 placed
├── vibrovario2.kicad_pro    # Project file
├── BMP390.kicad_sym         # BMP390 symbol library
├── vibrovario2.lib          # Legacy symbol library
├── sym-lib-table            # Symbol library table
├── vibrovario2.pretty/      # Footprint library
│   └── BMP390_LGA-10.kicad_mod
└── README.md
```

## Assembly Notes
1. BMP390 is a small LGA-10 package (2x2mm) - requires precise soldering
2. Recommend using solder paste and hot air reflow for BMP390
3. Decoupling capacitors should be placed close to BMP390 VDD pin
4. All components are SMD for factory assembly

## Firmware Integration
For Arduino/PlatformIO:

```cpp
#include <Wire.h>
#include <Adafruit_BMP3XX.h>

#define BMP390_SDA 12
#define BMP390_SCL 11
#define BMP390_PWR 45

Adafruit_BMP3XX bmp;

void setup() {
  // Enable BMP390 power
  pinMode(BMP390_PWR, OUTPUT);
  digitalWrite(BMP390_PWR, HIGH);
  delay(10);  // Wait for power stabilization

  // Initialize I2C
  Wire.begin(BMP390_SDA, BMP390_SCL);

  // Initialize BMP390
  if (!bmp.begin_I2C(0x77)) {
    Serial.println("BMP390 not found!");
    while (1);
  }

  // Configure for variometer use
  bmp.setTemperatureOversampling(BMP3_OVERSAMPLING_8X);
  bmp.setPressureOversampling(BMP3_OVERSAMPLING_4X);
  bmp.setIIRFilterCoeff(BMP3_IIR_FILTER_COEFF_3);
  bmp.setOutputDataRate(BMP3_ODR_25_HZ);
}

void loop() {
  if (bmp.performReading()) {
    float altitude = bmp.readAltitude(1013.25);
    Serial.printf("Altitude: %.2f m\n", altitude);
  }
  delay(100);
}
```

## BOM Additions
| Reference | Value | Package | Manufacturer | MPN | LCSC |
|-----------|-------|---------|--------------|-----|------|
| U5 | BMP390 | LGA-10 | Bosch | BMP390 | C496091 |
| C22 | 100nF | 0402 | Samsung | CL05B104JB5NNNC | C1525 |
| C23 | 1uF | 0402 | Samsung | CL05A105KA5NQNC | C52923 |

## License
Based on Watchy by SQFMI (https://sqfmi.com/watchy)
Original hardware design: MIT License
