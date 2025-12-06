# Home Assistant HVAC Requirements 12/5

# Goal

The goal of this automation is that all rooms are comfortable, condenser mode switching does not exceed 15 times per day, and little if any manual adjustment is needed by the user each day.

# General

* The automation should run automatically and not need manually restarting after an overall home assistant instance restart  
* The system should regularly check if rooms are out of deviation, at least once per minute.  
* Our goal is that rooms are comfortable 

# Sensors

* The automation should use the Sonoff temperature sensors as the authoritative truth for the room temperatures

# Setpoints & Deviations

* Each room should have a target temperature setpoint defined  
* Each room should have a default deviation of plus or minus 2 degrees from its setpoint  
* At night, bedroom deviation remains at 2 degrees but the deviation of all other rooms changes to 5 degrees

# Ranking, Targeting, & Priority

* All rooms should have a unique ranking 1-4, with 1 being the top ranking  
* If a room is out of deviation for more than 20 minutes it becomes a targeted room. There can be more than 1 targeted room at a time. There can also be zero targeted rooms if none are out of deviation.  
* The top-ranked room should be prioritized for conditioning if it is out of deviation and even if lower-ranked rooms are also out of deviation  
* There can be a max of one priority room at a time, and there can be zero priority rooms if there are no targeted rooms (i.e. if there are no rooms out of deviation)

# Condenser Mode

* All units are only on one mode at a time (heat or cool)  
* If the priority room is being cooled, also cool all other rooms that are out of setpoint  
* If the priority room is being heated, also heat all other rooms that are out of setpoint  
* If a room is at setpoint and condenser mode is heat, set target temp to 60 degrees to effectively put that room idle  
* If a room is at setpoint and condenser mode is cool, set target temp to 80 degrees to effectively put that room idle  
* Condenser should run for a minimum of 45 minutes in one mode before a mode switch is allowed

# Fan Settings

* Fan speed should be set by deviation amount. If \>3 degree deviation, set fan to HIGH. If 1.5-3 degree deviation, set to medium. If at setpoint, set to low.
* Exception: First floor always runs on HIGH when being conditioned (due to its larger size of 600 sq ft).

# Configurability

The below system variables should be configurable:

* Room setpoints  
* Room rankings  
* Room deviations  
* Night start and end  
* Night deviation for non-bedroom rooms  
* Deviation delay  
* Condenser mode switch minimum

# Dashboard

* All configurations can be set  
* Key values can be seen  
  * Current priority room  
  * Current condenser mode  
  * Condenser mode changes over time  
  * Current room temperatures  
  * Room temperatures over time  
  * Condenser mode limits

# Testing

* There should be an automated test that tests all of the logical conditions in this document without relying on the actual hardware. It should run in seconds.  
* The test should be able to be run from the HVAC dashboard in Home Assistant with one click of a button  
* Results for each test should show in the Dashboard in a card