# Global Configurations
## Helpers
There are 4 global helpers that control settings across the whole house:
1. [Home](#home)
2. [Heating Mode](#heating-mode)
3. [Holiday Heating](#holiday-heating)
4. [Main Heating Switch](#main-heating-switch)
5. [Away Temp Adjustment](#away-temp-adjustment)

### Home
This is a template binary sensor that will return the inverse value of the Away helper. This is used in AHC to activate Guest mode if all tracked people are away. Manually setting the house to Home in the occupancy dashboard screen will set this entity to True.

### Heating Mode
An input select helper toggled by a Dashboard selection that controls which schedules the AHC uses or if the heating is off. There are three options:
1. Normal. When activated, AHC will follow the normal schedules. This is typically a work & school day
2. Holiday. When activated, AHC will follow the holiday schedules. This is to be used on days when the kids are home, or we're not at work, but at home, e.g. Christmas
3. Off. This will turn the heating off

<img width="397" height="161" alt="image" src="https://github.com/user-attachments/assets/ea4e403e-e844-4291-966a-b3bb485f8dd1" />

Moving between the options will trigger the Heating Mode automation that will toggle the following helpers on or off:

### Holiday Heating
When on, holiday heating schedules are followed

### Main Heating Switch
Heating will only take place when this switch is on

### Away Temp Adjustment
Set by the [Proximity Update](#proximity-update) automation to adjust the away temperature

## Automations
There are 2 global automations:
1) [Heating Mode](#heating-mode-1)
2) [Proximity Update](#proximity-update)

### Heating Mode
The automation is triggered by a change in the Heating Mode Helper - moving between Normal, Holiday and Off. When triggered the autmation toggles the Holiday Heating and Main Heating Switch Helpers accordingly:
```
actions:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - normal
        sequence:
          - action: input_boolean.turn_off
            metadata: {}
            data: {}
            target:
              entity_id: input_boolean.ahc_holiday_heating
          - action: input_boolean.turn_on
            metadata: {}
            data: {}
            target:
              entity_id: input_boolean.ahc_main_heating_switch
      - conditions:
          - condition: trigger
            id:
              - holiday
        sequence:
          - action: input_boolean.turn_on
            metadata: {}
            data: {}
            target:
              entity_id: input_boolean.ahc_holiday_heating
          - action: input_boolean.turn_on
            metadata: {}
            data: {}
            target:
              entity_id: input_boolean.ahc_main_heating_switch
      - conditions:
          - condition: trigger
            id:
              - "off"
        sequence:
          - action: input_boolean.turn_off
            metadata: {}
            data: {}
            target:
              entity_id: input_boolean.ahc_main_heating_switch
```
### Proximity Update
AHC will drop the target temperature of a room by a set number of degrees if no one is detected to be at home - or if guest mode isn't activated.
Tado has a feature that adjusts the temperature gradually depending on how far away from home you are - the away temperature decreases the further (and maybe longer) you are away from home, and then increases as you get closer to home.

This is a feature I wanted to try and replicate, which this automation attempts to achieve.

Using the [Proximity](https://www.home-assistant.io/integrations/proximity) integration, members of the house are tracked by their distance (meters) from home. As the nearest person moves between set distances from home (Zones, but not to be confused with Home Assistant Zones), this automation changes the value of the [Away Temp Adjustment](#away-temp-adjustment) Helper, and re-runs all AHC automations tagged with 'Heating Control'. AHC is then configured to subtract the value of this helper from the comfort temperature of the room. The values and distances are as follows:

|Zone  |Distance	| Temperature Difference |
|----- |----------|------------------------|
|Zone0 |0         |0                       |
|Zone1 |3000	    |2                       |
|Zone2 |7500	    |3                       |
|Zone3 |15000	    |4                       |
|Zone4 |30000	    |5                       |
|Zone5 |64000	    |6                       |
|Zone6 |64001+	  |25                      |

Setting Zone6 to be 25 will ensure that, regardless of the target temp, the resulting value of `target temp - away temp adjustment` will always be less than 12, and therefore the away temp adjustment in [AHC](/docs/heating/room_configurations.md#advanced-heating-control) will set the temp to 12.
