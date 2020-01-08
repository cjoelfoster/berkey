# Auto Filler for Berkey Water Filtration System

## Goal
Automate filling of a Berkey water filtration system

## Constraints
1. Cannot require manual intervention
2. Cannot overflow upper or lower reservoir
3. Must account for time-dependent filtration process

## Description of the System
Berkey water filtration system is a single-stage filtration device that contains an upper reservoir (to be filtered) and a lower reservoir (of filtered water) seperated by a filter. The lower reservoir also incorporates a dispenser.


[==========]
|          |
|   upper  |
| reservoir|
|          |
------------
|  filter  |
------------
|          |
|   lower  | \
| reservoir|={}  <-- dispenser
|          |  
============

Fig. 1: Simplified model of the Berkey water filtration system

## Option 1

### Description
The fill line ties off the cold water line to the kitchen tap and routes to the valve. When the lower limit switch trips showing that the lower reservoir of purified water is empty, the valve is switched to the ON position until the full limit on the upper reservoir trips at which time the valve is switched back to the OFF position (default).

### Schematic

water:
[cold water line to tap]------->[fill valve]------->[upper reservoir]------->[filter]------->[lower reservoir/dispenser]

logic:
[fill valve (ON/OFF)]<---------(DO)[micro     ](DI)<------------[upper limit switch (Upper_Reservoir_is_Full)]
                                   [controller](DI)<------------[lower limit switch (Lower_Reservoir_is_Empty)]



### Hardware
- 1/4in. plastic refrigeration line (<6ft)
- Connector  T (1/2 - 1/2 - 1/4; splice into the 1/2in. cold water line)
- Small uC with 3 digital IO ports
- 2 limit switches
- Digitally controlled flow valve (ON/OFF)

### Software

Logic:
If upper reservoir is full, do not open valve, or close it if open (e.g. upper limit switch digital input = 1 or TRUE).
If lower reservoir is not empty, do not open valve, or close it if open (e.g. lower limit switch digital input = 1 or TRUE).
If upper reservoir is empty, and lower reservoir is empty, open the valve if closed (e.g. both limit switches = 0 or FALSE).

Four states:
1. lower reservoir full, upper empty (default--purified water available for use)
    UPPER = FALSE
    LOWER = TRUE
    VALVE = OFF
2. lower reservoir full, upper full (disallowed--would lead to an overflow of the lower reservoir)
    UPPER = TRUE
    LOWER = TRUE
    VALVE = OFF
3. lower reservoir empty, upper empty (fill--time to fill up the upper reservoir)
    UPPER = FALSE
    LOWER = FALSE
    VALVE = ON
4. lower reservoir empty, upper full (stop filling--turn off the water to keep from overflowing the upper reservoir)
    UPPER = TRUE
    LOWER = FALSE
    VALVE = OFF
    
Pseudo-code:
```
Initialize:

    set VALVE = OFF
    set upper_is_full = TRUE
    set lower_is_full = FALSE

Loop:
    set upper_is_full = UPPER-LIMIT-SWITCH-INPUT
    set lower_is_full = LOWER-LIMIT-SWITCH-INPUT
    
    if(upper_is_full = TRUE)
        set VALVE = OFF
        /* do not permit overflow of upper reservoir */
    
    if(upper_is_full = FALSE)
        if(lower_is_full = TRUE)
            set VALVE = OFF
            /* do not keep filling upper reservoir once lower reservoir has water in it again */
    
    if(upper_is_full = FALSE)
        if(lower_is_full = FALSE)
           set VALVE = TRUE
           /* lower and upper reservoirs are empty, time to fill! */
    
    wait 1 second
```
  
