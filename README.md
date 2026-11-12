
# Smart Safety Monitor (STM32)

A small safety monitor I'm building on a NUCLEO-F103RB as my embedded
systems project. It watches three values (temperature, gas, vibration)
and decides whether the machine is normal, in warning, or critical.
If anything goes critical, the system stays latched until someone
sends a RESET command.

I'm a fresher, so I built it step by step: first the logic in plain C
tested on my PC, then moved it into a CubeIDE project.

## Current status
- Firmware builds with 0 errors and 0 warnings (about 17.7 KB flash, 2.2 KB RAM)
- Logic tested on PC with assert tests
- Not tested on the board with real sensors yet (hardware not arrived)

## What it does
- Three channels (TEMP, GAS, VIB), each with warning and critical limits
- Hysteresis, so the alarm does not flicker near a limit
- Critical goes back to warning first, never straight to normal
- Latch: after a critical event it stays critical until RESET.
  RESET is refused unless all channels are normal and no sensor has a fault
- A reading outside the valid range is treated as a sensor fault (critical)
- LED blink pattern changes with the state, done with HAL_GetTick (no HAL_Delay)
- Independent watchdog, about 1 second
- UART commands, interrupt based receive
- Internal temperature sensor read through ADC once per second

## UART commands (115200, 8N1)
- `SET TEMP|GAS|VIB value` : inject a value for testing
- `STATUS` : show all channels, faults, latch
- `RESET` : clear the latch (only if safe)
- `HELP`
## Limits I used
- TEMP: warning 60 (back at 55), critical 80 (back at 75)
- GAS: warning 400 (back at 350), critical 700 (back at 650)
- VIB: warning 30 (back at 20), critical 70 (back at 60)

## How the code is split
- safety_logic.c: state machine for one channel
- safety_system.c: three channels, latch, reset rule
- cmd_parser.c: reads UART lines and turns them into commands
- fault.c: valid range check
- alarm.c: LED pattern
- filter.c: moving average for the temperature reading
- main.c: hardware setup and the main loop

## Build
Open in STM32CubeIDE, build, flash with the on-board ST-LINK,
open the virtual COM port at 115200 and type HELP.

## Known problems
- The internal temperature sensor is chip temperature and its offset
  differs between chips, so it is only to test the ADC path.
- If a new command line arrives before the last one is handled, it is dropped.
- After SET TEMP, temperature stays in manual mode until the board is reset.

## Next
- Real sensors: MQ-2 gas, vibration sensor, DS18B20, relay
- Event log
- Hardware test results and a demo video

- ## Author
Birendra Kumar Sah
