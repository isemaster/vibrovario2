# VibroVario 2.0

**Тактильный бесшумный вариометр на базе Watchy V3.0 с интегрированным датчиком BMP390**

---

## Описание

VibroVario 2.0 — это продолжение проекта [VibroVario](https://github.com/isemaster/VibroVario), основанное на платформе Watchy V3.0 (ESP32-S3). Ключевое отличие — **интегрированный на плату датчик BMP390** для поверхностного монтажа (SMD), что исключает необходимость пайки проводами.

### Особенности:
- ✅ **BMP390 интегрирован в плату** — заводской SMD монтаж
- ✅ **Управляемое питание датчика** — через GPIO45 для экономии батареи
- ✅ **I2C шина** — SDA=GPIO12, SCL=GPIO11
- ✅ **Адрес BMP390** — 0x77 (SDO подтянут к VDDIO)
- ✅ **Совместимость с VibroVario** — минимальные изменения в прошивке

---

## Аппаратные изменения относительно Watchy V3.0

### Добавлено:
| Компонент | Обозначение | Питание | Примечание |
|-----------|-------------|---------|------------|
| BMP390 | U6 | GPIO45 | Барометр для вариометра |
| Керамика 100nF | C7 | VDD | Декаплинг BMP390 |
| Керамика 1µF | C8 | VDDIO | Фильтр BMP390 |
| Резистор 10kΩ | R10 | SDA | Pull-up I2C (доп.) |
| Резистор 10kΩ | R11 | SCL | Pull-up I2C (доп.) |
| Резистор 4.7kΩ | R12 | CSB | Pull-up для I2C режима |

### Распиновка BMP390:
| Пин BMP390 | Соединение | Примечание |
|------------|------------|------------|
| 1 - GND | GND | Земля |
| 2 - VDDIO | +3.3V (через R/C фильтр) | Питание I/O |
| 3 - SDO | VDDIO | Адрес I2C = 0x77 |
| 4 - INT2 | NC | Не используется |
| 5 - VDD | GPIO45 | Управляемое питание |
| 6 - INT1 | NC | Не используется |
| 7 - SCL | GPIO11 | I2C Clock |
| 8 - SDA | GPIO12 | I2C Data |
| 9 - CSB | +3.3V (через R12) | Pull-up для I2C |
| 10 - GND | GND | Земля |

---

## Файлы проекта KiCAD

```
vibrovario2/
├── vibrovario2.kicad_sch     # Принципиальная схема
├── vibrovario2.kicad_pcb     # Печатная плата
├── vibrovario2.kicad_pro     # Проект KiCAD
├── vibrovario2.lib           # Библиотека компонентов
├── sym-lib-table             # Таблица символов
├── vibrovario2.pretty/       # Футпринты
│   └── BMP390_LGA-10.kicad_mod  # Футпринт BMP390
├── BMP390.kicad_sym          # Символ BMP390
├── GERBER/                   # Gerber файлы (генерируются)
├── docs/
│   └── schematic.pdf         # PDF схемы (генерируется)
└── README.md                 # Документация
```

---

## Инструкция по размещению BMP390 в KiCAD

### Шаг 1: Добавить символ BMP390 в проект
1. Открыть `vibrovario2.kicad_sch` в KiCAD
2. Добавить библиотеку: `Preferences → Manage Symbol Libraries → Add`
3. Выбрать файл `BMP390.kicad_sym`

### Шаг 2: Разместить компонент на схеме
1. Нажать `A` (Add Symbol)
2. Найти `BMP390`
3. Разместить рядом с I2C линией (SDA/SCL)
4. Соединить:
   - VDD → GPIO45 (Net: VARIO_EN)
   - VDDIO → +3V3
   - SDA → GPIO12 (Net: SDA)
   - SCL → GPIO11 (Net: SCL)
   - GND → GND
   - SDO → +3V3 (для адреса 0x77)
   - CSB → +3V3 (через резистор 4.7kΩ)

### Шаг 3: Добавить пассивные компоненты
- C7: 100nF 0402 между VDD и GND (рядом с U6)
- C8: 1µF 0402 между VDDIO и GND (рядом с U6)
- R10, R11: 10kΩ 0402 pull-up на SDA/SCL (если отсутствуют)
- R12: 4.7kΩ 0402 pull-up на CSB

### Шаг 4: Размещение на PCB
1. Открыть `vibrovario2.kicad_pcb`
2. Обновить из схемы: `Tools → Update PCB from Schematic`
3. Разместить BMP390 в свободной зоне (рекомендуется: рядом с BMA423)
4. Развести трассы:
   - SDA/SCL — ширина 0.15-0.2mm
   - VDD — ширина 0.25mm (от GPIO45)
   - GND — через via на слой GND

### Шаг 5: Генерация Gerber
1. `File → Plot`
2. Выбрать слои: F.Cu, B.Cu, F.Paste, B.Paste, F.SilkS, B.SilkS, F.Mask, B.Mask, Edge.Cuts
3. Формат: Gerber
4. Output: папка `GERBER/`

---

## BOM (Bill of Materials)

Полный BOM с компонентами для заказа: **`BOM_vibrovario2.xlsx`**

### Ключевые компоненты:
| Компонент | MPN | Корпус | Источник |
|-----------|-----|--------|----------|
| BMP390 | BMP390 | LGA-10 2x2mm | LCSC, Mouser |
| ESP32-S3FN8 | ESP32-S3FN8 | QFN-56 | LCSC |
| BMA423 | BMA423 | LGA-12 | LCSC |
| PCF8563 | PCF8563BS | SOIC-8 | LCSC |

---

## Прошивка

Совместима с оригинальным VibroVario с минимальными изменениями:

```cpp
// В VibroVario.ino изменить пины для ESP32-S3:
#define SDA_PIN 12      // было 21
#define SCL_PIN 11      // было 22
#define VARIO_POWER 45  // новый пин питания BMP390

// Инициализация:
pinMode(VARIO_POWER, OUTPUT);
digitalWrite(VARIO_POWER, HIGH);  // Включить BMP390
delay(50);  // Время на прогрев
bmp.begin_I2C(0x77);  // Адрес 0x77
```

---

## Производство

### Для заказа PCB:
1. Загрузить Gerber файлы на JLCPCB / PCBWay / etc.
2. Спецификация:
   - 4 слоя
   - Толщина: 1.0mm
   - Медь: 1oz
   - Паяльная маска: зелёная
   - Пленка: белая

### Для PCBA (сборка):
1. Загрузить BOM и позиционный файл (CPL)
2. Уточнить наличие BMP390 (LCSC: C5124834)

---

## Лицензия

MIT License — используйте свободно с указанием авторства.

---

## Благодарности

- [SQFMI Watchy](https://github.com/sqfmi/Watchy) — базовая платформа
- [VibroVario](https://github.com/isemaster/VibroVario) — оригинальный проект вариометра
- Bosch Sensortec — датчик BMP390
