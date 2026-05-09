# Hydra Keyboard — Pin Configuration

## Nice!Nano V2 Pin Assignments (Both Sides Identical Hardware)

### Matrix & Encoder Pins

| Arduino | Frequency | Signal | Description |
|---------|-----------|--------|-------------|
| D1 | ✅ HIGH | ENC_B | Encoder pin B |
| D0 | ✅ HIGH | ENC_A | Encoder pin A |
| D9 | ⚠️ LOW | ENC_SW | Encoder switch (push) |
| D21 | ✅ HIGH | MOS | LED MOSFET gate |
| D20 | ✅ HIGH | ROW1 | Matrix row 1 (top) |
| D19 | ✅ HIGH | ROW2 | Matrix row 2 |
| D18 | ✅ HIGH | ROW3 | Matrix row 3 |
| D15 | ✅ HIGH | ROW4 | Matrix row 4 (THUMB) |

| D14 | ✅ HIGH | COL1 | Matrix column 1 |
| D16 | ⚠️ LOW | COL2 | Matrix column 2 |
| D10 | ⚠️ LOW | COL3 | Matrix column 3 |


**Matrix Layout:** 4 rows × 5 cols per side (standard matrix, col2row diodes)

---

### Sensor Pins (Trackpad I2C / Trackball SPI)

| Arduino | Frequency | Signal | I2C (Trackpad) | SPI (Trackball) |
|---------|-----------|--------|--------|----------|
| D6 | ✅ HIGH | SCL/SCK | Trackpad SCL | PMW3610 SCK |
| D7 | ✅ HIGH | SDA/MOSI | Trackpad SDA | PMW3610 MOSI |
| D5 | ✅ HIGH | DR/IRQ | Trackpad DR | PMW3610 MOT |
| D4 | ✅ HIGH | NCS | — | PMW3610 NCS |

**Trackpad I2C:** Address `0x2A` on D6 (SCL) + D7 (SDA)  
**Trackball SPI:** 2MHz on D6 (SCK) + D7 (MOSI) + D5 (IRQ) + D4 (NCS)

---

### OLED Display Pins (SSD1306 128×32)

| Arduino | Frequency | Signal | I2C Config | SPI Config |
|---------|-----------|--------|------------|------------|
| D2 | ✅ HIGH | CS | OLED pin 8 → VDD (JP2 pos 2-3) | OLED pin 8 ← D2 (JP2 pos 1-2) |
| D3 | ✅ HIGH | DC | OLED pin 10 ← D6 (JP3 pos 2-3) | OLED pin 10 ← D3 (JP3 pos 1-2) |
| D6 | ✅ HIGH | SCL/SCLK | OLED pin 11 (JP4 pos 2-3) | OLED pin 11 ← D6 (JP4 pos 1-2) |
| D7 | ✅ HIGH | SDA/SDIN | OLED pin 12 (JP5 pos 2-3) | OLED pin 12 ← D7 (JP5 pos 1-2) |

**Fixed OLED Connections:**
- Pin 1–4: Charge pump caps (C1, C2 = 1µF each)
- Pin 6: VSS → GND
- Pin 7: VDD → 3.3V
- Pin 9: RES# → RC circuit (VDD → R1 10kΩ → junction → GND via C5 1µF)
- Pin 15: — (NC internally)

**OLED VBAT (Pin 5):** BAT54C Schottky OR gate (higher of battery or 3.3V)

---

### OLED Solder Jumper Configuration

| Jumper | Config | I2C (2&3) | SPI (1&2) | OLED Pin | Function |
|--------|--------|-----------|-----------|----------|----------|
| JP2 | pos 2-3 | VDD (3.3V) | D2 (CS) | 8 | VDD / CS |
| JP3 | pos 2-3 | D6 (SCL) | D3 (DC) | 10 | SCL / DC |
| JP4 | pos 2-3 | D7 (SDA) | D6 (SCLK) | 11 | SDA / SCLK |
| JP5 | pos 2-3 | R4 560kΩ → GND | D7 (SDIN) | 12 | IREF / SDIN |
| JP6 | pos 2-3 | C8 2.2µF → GND | R5 2MΩ → GND | 13 | VCOMH / IREF |
| JP7 | pos 2-3 | VDD | C8 2.2µF → GND | 14 | VCC / VCOMH |

**Summary:** 
- **I2C mode:** All jumpers at positions 2-3
- **SPI mode:** All jumpers at positions 1-2

---

### LED Control Circuit (WS2812B + SI2301CDS P-Channel MOSFET)

```
Battery B+ ─────────────────────────► SI2301 Source (pin 2)
                                      SI2301 Drain (pin 3) ──► WS2812B VDD
                                      
D8 ─────────────────────────────────► WS2812B DIN (data)

D21 ──[10Ω]──► SI2301 Gate (pin 1)
        │
     [10kΩ]
        │
    Battery B+ (pull-up)

WS2812B GND ──► GND
```

**LED Power Control:**
- `D21 = LOW` → MOSFET ON → LEDs powered
- `D21 = HIGH` → MOSFET OFF → LEDs fully cut (zero battery drain)

---

### VBAT Supply (BAT54C-E3-08 Schottky OR Circuit)

```
Battery B+ ──┐
            Anode 1 ─┐
                     ├─ Cathode ──► OLED VBAT (pin 5)
3.3V Reg ──┐        ┌─┘
          Anode 2 ──┘

(Whichever is higher wins; battery B+ when plugged in)
```

---

### PMW3610 Trackball Reset

- **NRESET:** RC auto-reset circuit (no GPIO)
- VDD → R1 10kΩ → NRESET junction → GND via C1 1µF
- Firmware resets via SPI register write

---

## Configuration by Side

| Aspect | Left Side | Right Side | Hardware |
|--------|-----------|------------|----------|
| Sensor | Cirque Trackpad (I2C) | PMW3610 Trackball (SPI) | **Identical PCB** |
| OLED Jumpers | All 2-3 (I2C mode) | All 1-2 (SPI mode) | Same hardware |
| Firmware | `hydra_left.overlay` | `hydra_right.overlay` | Overlay selects mode |

---

## Matrix Layout (Both Sides — Japanese Duplex)

```
ROW1 (D20)  → S1   S2   S3   S4   S5
ROW2 (D19)  → S6   S7   S8   S9  S10
ROW3 (D18)  → S11  S12  S13  S14  S15
ROW4 (D15)  → S16  S17  S18  S19  S20  (THUMB)
              ↑    ↑    ↑    ↑    ↑
             D14  D16  D10  TBD  TBD
             COL1 COL2 COL3 COL4 COL5
```

**Total:** 4 rows × 5 cols = 20 keys per side × 2 = 40 keys
**Anti-ghosting:** 1N4148 diodes in col2row configuration

---

## GPIO Frequency Notes

⚠️ **LOW frequency pins** (P1.04, P1.06, P0.09, P0.10): Used for matrix scanning only.  
✅ **HIGH frequency pins:** Used for I2C, SPI, encoder, and OLED control.

- nRF52840 HIGH frequency: up to 32 MHz (suitable for 2MHz SPI, 400kHz I2C)
- nRF52840 LOW frequency: up to 2 MHz (suitable for matrix scanning, not suitable for serial protocols)
