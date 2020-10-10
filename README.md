# Prusa Mini MINDA gcode to ensure first layer consistency
This alternative start/end gcode was sent to me by Olof Ogland (https://www.olofogland.se), the designer of Bondtech extruder upgrade for the Prusa Mini. Its goal is to make the first layer z-height somewhat more consistent. It's been working great so far, and a lot better than the pre-heat method I used to use before.

## Start Gcode:
``M115 U3.2.1 ; tell printer latest fw version
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
G1 Z2 F1000 ; lift nozzle``

## End Gcode
``G1 E-1 F2100 ; retract
{if layer_z < max_print_height}G1 Z{z_offset+min(layer_z+5, max_print_height)}{endif} F720 ; Move print head up
G1 X178 Y180 F4200 ; park print head
G4 ; wait
M104 S0 ; turn off temperature
M140 S0 ; turn off heatbed
M107 ; turn off fan
M221 S100 ; reset flow
M84 ; disable motors``
