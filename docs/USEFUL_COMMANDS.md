# Useful G-code Commands for Hellbot Magna 2

Quick reference guide for commonly used commands on dual extruder, single nozzle setup.

## Table of Contents
- [Z Offset Adjustment](#z-offset-adjustment)
- [Tool Change & Filament Swap](#tool-change--filament-swap)
- [Temperature Control](#temperature-control)
- [Calibration](#calibration)
- [Movement & Homing](#movement--homing)
- [EEPROM Management](#eeprom-management)
- [Diagnostics](#diagnostics)

---

## Z Offset Adjustment

### View Current Z Offset
```gcode
M503                    ; Show all current settings (look for M851 Z value)
```

### Adjust Z Offset (Fine Tuning)
```gcode
M851 Z-2.12            ; Set Z offset to -2.12mm (negative = closer to bed)
M851 Z-2.00            ; Move nozzle 0.12mm away from bed
M500                   ; Save to EEPROM
M501                   ; Load from EEPROM
```

### Live Adjustment During First Layer
```gcode
; While printing first layer:
M290 Z-0.05            ; Move nozzle 0.05mm closer (babystepping)
M290 Z0.05             ; Move nozzle 0.05mm away (babystepping)
```

### Z Offset Wizard (if enabled)
```gcode
G28                    ; Home all axes first
M851 Z0                ; Reset Z offset to zero
G1 Z0                  ; Move to Z=0 (nozzle should be touching bed)
; Use paper test method, then measure the gap
M851 Z-X.XX            ; Set measured offset
M500                   ; Save
```

---

## Tool Change & Filament Swap

### Basic Tool Selection
```gcode
T0                     ; Select extruder 0 (left/primary)
T1                     ; Select extruder 1 (right/secondary)
```

### Configure Tool Change Filament Swap (M217)
```gcode
M217                   ; Show current filament swap settings

; Adjust swap parameters:
M217 S12               ; Swap length (mm) - distance to retract/load
M217 E0                ; Extra resume length (mm)
M217 P10               ; Extra prime amount (mm) for single nozzle
M217 R3000             ; Retract speed (mm/min)
M217 U1500             ; Unretract/load speed (mm/min)
M217 F276              ; Extra prime speed (mm/min)

M500                   ; Save settings to EEPROM
```

### Manual Filament Load/Unload
```gcode
; Heat up first:
M109 S200              ; Heat and wait for 200°C

; Load filament manually:
G91                    ; Relative positioning
G1 E50 F300            ; Extrude 50mm at 5mm/s
G90                    ; Absolute positioning

; Unload filament manually:
G91                    ; Relative positioning
G1 E-100 F1800         ; Retract 100mm at 30mm/s
G90                    ; Absolute positioning
```

---

## Temperature Control

### Set Temperatures
```gcode
M104 S200              ; Set hotend temp to 200°C (no wait)
M109 S200              ; Set hotend temp to 200°C and WAIT

M140 S60               ; Set bed temp to 60°C (no wait)
M190 S60               ; Set bed temp to 60°C and WAIT

M104 T0 S200           ; Set T0 (extruder 0) to 200°C
M104 T1 S0             ; Turn off T1 (extruder 1)
```

### Standby Temperatures (SINGLENOZZLE)
When using dual extruders with single nozzle, the inactive extruder can have a standby temperature:
```gcode
M104 T0 S210           ; Set T0 active temp to 210°C
M104 T1 S180           ; Set T1 standby temp to 180°C
T0                     ; T0 heats to 210°C, T1 drops to 180°C
T1                     ; T1 heats to 210°C, T0 drops to 180°C
```

### Check Current Temperatures
```gcode
M105                   ; Report temperatures
```

### Cooling
```gcode
M106 S255              ; Fan at 100% (S0-255)
M106 S128              ; Fan at 50%
M107                   ; Fan off
```

---

## Calibration

### E-Steps Calibration
```gcode
M503                   ; Check current E-steps (look for M92 E value)

; To calibrate:
; 1. Mark filament 120mm above extruder
; 2. Heat hotend to printing temp
M109 S200

; 3. Extrude 100mm
G91                    ; Relative mode
G1 E100 F100           ; Extrude 100mm slowly
G90                    ; Absolute mode

; 4. Measure remaining distance from mark
; 5. Calculate: new_steps = current_steps * (100 / actual_extruded)
; Example: if only 95mm extruded:
; new_steps = 93 * (100/95) = 97.89

M92 E97.89             ; Set new E-steps
M500                   ; Save to EEPROM
```

### PID Tuning (Hotend)
```gcode
M106 S255              ; Turn on fan at full speed
M303 E0 S200 C8        ; PID tune extruder 0 at 200°C, 8 cycles
; Wait for completion, then note Kp, Ki, Kd values

M301 P22.20 I1.08 D114.00  ; Set PID values
M500                       ; Save to EEPROM
```

### PID Tuning (Bed)
```gcode
M303 E-1 S60 C8        ; PID tune bed at 60°C, 8 cycles
; Wait for completion, then note Kp, Ki, Kd values

M304 P294.00 I65.00 D332.00  ; Set bed PID values
M500                          ; Save to EEPROM
```

### Bed Leveling (Manual)
```gcode
G28                    ; Home all axes
G29                    ; Auto bed leveling (if BLTouch enabled)
M500                   ; Save mesh to EEPROM
M420 S1                ; Enable bed leveling
```

---

## Movement & Homing

### Homing
```gcode
G28                    ; Home all axes
G28 X Y                ; Home X and Y only
G28 Z                  ; Home Z only
```

### Manual Movement
```gcode
G1 X100 Y100 Z10 F3000 ; Move to X100 Y100 Z10 at 3000mm/min
G1 Z0.2                ; Move to Z=0.2mm (useful for leveling)
G1 X0 Y0               ; Move to front-left corner
```

### Relative vs Absolute
```gcode
G90                    ; Absolute positioning (default)
G91                    ; Relative positioning
```

### Disable Steppers
```gcode
M84                    ; Disable all steppers
M18                    ; Same as M84
M84 X Y                ; Disable only X and Y
```

---

## EEPROM Management

### Save/Load Settings
```gcode
M500                   ; Save current settings to EEPROM
M501                   ; Load settings from EEPROM
M502                   ; Reset to factory defaults (RAM only)
M503                   ; Display current settings
```

### Complete Reset Procedure
```gcode
M502                   ; Load factory defaults to RAM
M500                   ; Save to EEPROM
M501                   ; Reload from EEPROM
```

---

## Diagnostics

### System Information
```gcode
M115                   ; Firmware info and capabilities
M503                   ; Show all configuration settings
```

### Pin State Report
```gcode
M43                    ; Display pin states
M43 P5                 ; Watch pin 5
M119                   ; Endstop status
```

### Temperature Debugging
```gcode
M105                   ; Report temperatures
M155 S1                ; Auto-report temp every 1 second
M155 S0                ; Disable auto-report
```

### Motor Current (if TMC drivers)
```gcode
M906                   ; Report driver currents
M906 X800 Y800 Z800    ; Set X, Y, Z current to 800mA
M906 E800              ; Set E current to 800mA
M500                   ; Save
```

### Acceleration & Jerk
```gcode
M201                   ; Report max acceleration
M201 X1000 Y1000       ; Set max acceleration for X, Y
M204 P1000 T1000       ; Set print/travel acceleration
M205 X10 Y10           ; Set jerk for X, Y
M500                   ; Save
```

---

## Common Workflows

### Pre-Print Checklist
```gcode
G28                    ; Home all axes
M420 S1                ; Enable bed leveling (if saved mesh exists)
M104 S200              ; Start heating nozzle
M140 S60               ; Start heating bed
M190 S60               ; Wait for bed
M109 S200              ; Wait for nozzle
G1 Z0.3                ; Raise nozzle for safety
```

### Post-Print Routine
```gcode
M104 S0                ; Turn off hotend
M140 S0                ; Turn off bed
M107                   ; Turn off fan
G91                    ; Relative positioning
G1 Z10                 ; Raise Z by 10mm
G90                    ; Absolute positioning
G1 X0 Y200             ; Move bed forward for access
M84                    ; Disable steppers
```

### Emergency Stop
```gcode
M112                   ; Emergency stop (halts all operations)
```

### Tool Change Test Sequence
```gcode
G28                    ; Home
M109 S200              ; Heat to 200°C
T0                     ; Select tool 0
G1 E10 F100            ; Extrude 10mm
T1                     ; Switch to tool 1 (triggers filament swap)
G1 E10 F100            ; Extrude 10mm with tool 1
T0                     ; Switch back to tool 0
```

---

## Tips & Notes

1. **Always save after changes**: Use `M500` to save settings to EEPROM
2. **Z Offset Sign**: Negative values (-) move nozzle closer to bed
3. **Tool Change**: With SINGLENOZZLE, only one heater exists (T0), T1 shares it
4. **Babystepping**: M290 changes are temporary (not saved)
5. **Temperature**: Always preheat before extruding (`M109` or `M190`)
6. **Relative mode**: Remember to return to `G90` after using `G91`

---

## Reference

- Marlin Documentation: https://marlinfw.org/meta/gcode/
- G-code Reference: https://reprap.org/wiki/G-code
- Tool Change Settings: `M217` command
- Z Probe Offset: `M851` command