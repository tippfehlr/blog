---
title: "Recovering failed prints"
description: "If your print has just stopped for some reason, it might be recovered!"
date: 2023-09-23T20:00:00+01:00
draft: true
toc: false
cover: images/2023-09-24-articulated-night-spirit.png
coverCaption: "This is the model that lead to this write-up."
tags:
  - 3dprinting
---

- Empty filament (and no sensor)
- Klipper restart/lost connection
- Power outage
- etc.

> **KEEP THE BED HEATED!** \
> Otherwise the print might come loose and you have to start again.

> **ONLY MOVE THE NOZZLE WHILE IT'S HOT!** \
> The nozzle might stick to the print, ripping it lose.

## 1. Recovering bed leveling mesh

> If you’ve enabled bed mesh fadeout, you don’t need the bed mesh above `fade_end`.

If you saved your mesh, then you only have to load it.

Otherwise, check the console if the probe data is still available. If yes, then save these values to the end of your printer.cfg:

```
#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	  -0.056875, -0.142500, -0.141250, -0.232500
#*# 	  -0.201250, -0.255625, -0.164375, -0.196250
#*# 	  -0.108125, -0.231250, -0.232500, -0.350000
#*# 	  -0.119375, -0.286250, -0.288125, -0.416875
```

The `z_offset` from `[bltouch]` has to be subtracted from the probe values. If there are multiple values for each point, take the average.
This is important so you end up with the same mesh that the start of the print used too.

> If you can’t get your leveling mesh, it might be worth to try continuing without it

## 2. Recovering nozzle position

For this method, the x and y coordinates aren’t important. (I also had no way of measuring or retrieving them).

1. Measure the height of the (already printed!) model _correctly_.
2. If you believe you have the height of the currently printed layer, _measure again_ :)

You pretty much **need** a caliper for this. Ideally, your accuracy has to be within 1–2 layers. \
Use the end of the caliper to measure pockets.

## 3. Modifying the g-code file

0. If this is your only copy of the file, make a **backup**. Reslicing may produce a different result.
1. Search for the next layer (the one that would have been printed next). For this, add your layer height to your measurement. **Make sure that you are at that layer and not only retracting to that Z value**.
2. Delete everything before that line.

```bash
ed -s file.gcode <<< $'1,999999d\nwq'
```

```js
let i = 1;
```

_999999 is the last line to delete. (use **bash**, does not work in fish)_

3. Add to the beginning of the file:

- temperatures
- Set the Z value to the current position of the nozzle. Alternatively, home if and where possible (first method might be better) (`SET_KINEMATIC_POSITION X=0 Y=0 Z=0`)
- Also set the extruder position (`G92 E000.000`)
- enable the fan
- enable the bed mesh

## 4. resuming

> Be sure that there is no filament oozing from the nozzle. if there is, remove it while the nozzle is heated. You can freely move the head as long as the coordinates are set.

4. Double-check your print file, your print height, etc. This might be your only chance to rescue the print.

5. _RESUME :D_ (and hope it works)
