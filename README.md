# Prusa Mini gcode to improve first layer consistency
The Prusa Mini is no doubt a very capable 3D printer by Prusa Research. However, there is one issue that drives many user crazy (to a point they actually think about selling it) - the first layer inconsistencies.

Unlike the PINDA 2 probe that comes with the MK3S printer, the Prusa Mini's MINDA probe is not equipped with a thermistor so the printer can't compensate for  temperature fluctuations. There are two workarounds that help mitigate this issue to some degree.

## Option 1 - Keep the MINDA probe away from bed as long as possible
This alternative start/end gcode was sent to me by [Olof Ogland](https://www.olofogland.se), the designer of [Bondtech extruder upgrade for the Prusa Mini](https://www.bondtech.se/en/product/prusa-mini/) and [Bondtech Heat-break Kit for Prusa Mini](https://www.bondtech.se/en/product/bondtech-heat-break-for-prusa-mini/). 

It seems to work well if you're able to work in a room with stable ambient temperature.

**WARNING:** The placeholders used in the gcode below work with PrusaSlicer only. If you're using Cura or S3D, you will need to update it accordingly.

### Start Gcode
```M115 U3.2.1 ; tell printer latest fw version
G28 ; home all without mesh bed level
G90 ; use absolute coordinates
M83 ; extruder relative mode
G0 X180 Y180 Z180 F1500 ; this is a good MINDA cooling position and to have place to clean bed
M104 S176; set extruder temp
M140 S[first_layer_bed_temperature] ; set bed temp
M109 S176 ; wait for extruder temp
M190 S[first_layer_bed_temperature] ; wait for bed temp

G1 E-2 F1000 ; Retract a bit to prevent oozing
G29 ; mesh bed leveling
G0 Z30 F1500 ; raise for heating
M104 S[first_layer_temperature] ; set extruder temp
M109 S[first_layer_temperature] ; wait for extruder temp

; Start position
G1 Z0.2 F1500 ; go down to priming height
G1 Y-2.0 X179 F2400

; intro line
G1 X170 F1000
G1 X110.0 E8.0 F900
G1 X40.0 E10.0 F700
G1 Z2 F1000 ; lift nozzle
```

### End Gcode
```G1 E-1 F2100 ; retract
{if layer_z < max_print_height}G1 Z{z_offset+min(layer_z+5, max_print_height)}{endif} F720 ; Move print head up
G1 X178 Y180 F4200 ; park print head
G4 ; wait
M104 S0 ; turn off temperature
M140 S0 ; turn off heatbed
M107 ; turn off fan
M221 S100 ; reset flow
M84 ; disable motors
```

## Option 2 - Pre-heat cycle 
After a while of using Option 1 and getting somewhat mixed results due to inconsistent ambient temperatures, I decided to try the other workaround that pre-heats the MINDA probe for a couple of minutes before each print.

It should be noted that before the first print of the day, the pre-heat period should be made a little longer (15 to 20 minutes works for me). Any subsequent print, provided the break between the prints is only a few minutes, you only need the pre-set 3 minute pre-heat cycle.

### Start G-code
```
G90 ; use absolute coordinates
M83 ; extruder relative mode
G28 ; home all
G1 X100 Y100 F4000
G1 Z1.5 F50 ; park position
M104 S170 ; set extruder temp for bed levelling
M140 S[first_layer_bed_temperature] ; set bed temp
M109 R170 ; wait for bed levelling temp
M190 S[first_layer_bed_temperature] ; wait for bed temp
G4 S180 ; wait 180 secs when heated up
G28 ; home all without mesh bed level
G29 ; mesh bed levelling 
M104 S[first_layer_temperature] ; set extruder temp
G92 E0.0
G1 Y-2.0 X179 F2400
G1 Z3 F720
M109 S[first_layer_temperature] ; wait for extruder temp

; intro line
G1 X170 F1000
G1 Z0.2 F720
G1 X110.0 E8.0 F900
G1 X40.0 E10.0 F700
G92 E0.0
```
