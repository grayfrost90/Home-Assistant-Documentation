# Room Configurations
Each room requires a set of helpers and automations for the heating to work. This is primarily driven through the Advanced Heating Control (ADC) Blueprint.

## Helpers
There are 3 helpers that a room requires:
1) [Comfort Temp](#comfort-temp), named `AHC - <Room> Comfort Temp`. Used by ADC to set the target temperature for the room
2) [Schedule Normal](#schedule-normalholiday), named `AHC - <Room> Schedule Normal`. A Schedule helper that tells ADC when to turn the heating on, if the [Heating Mode](global_configurations.md#heating-mode) is set to Normal
3) [Schedule Holiday](#schedule-normalholiday) named `AHC - <Room> Schedule Holiday`. A Schedule helper that tells ADC when to turn the heating on, if the [Heating Mode](global_configurations.md#heating-mode) is set to Schedule

### Comfort Temp
This is an input_number helper and should be configured as follows:

|Field|Input|
|---|---|
|Name|`AHC - <Room> Comfort Temp`|
|Icon|`mdi:thermometer`|
|Minimum value|`12`|
|Maxinmum value|`25`|
|Display mode|`Input field`|
|Step size|`0.5`|
|Unit of measurement|`°C`|

<img width="484" height="650" alt="image" src="https://github.com/user-attachments/assets/100fedc6-d3bf-420d-83bc-5f6e12ab7b28" />


### Schedule Normal/Holiday
Two schedules for heating as explained. These define the times, and importantly temperature, when the heating is turned on. It's important that each time block has Additional data consisting of `target_temp: temp` where `temp` is the desired temperature for that time:

<img width="394" height="574" alt="image" src="https://github.com/user-attachments/assets/cd239547-83cd-4b64-967d-b15e886f89c5" />

This additional data is used later in the [target temp](#target-temp) automation

## Automations
Each Room requires the following automations:
1) [Target Temp](#target-temp), named `AHC - <Room> Target Temp`. Used to adjust the comfort temp helper that ADC uses to set the room temperature to.
2) [Advanced Heating Control](#advanced-heating-control), named `AHC - <Room> Heating`. The main component that controls the climate entities, this is configured from a Blueprint

### Target Temp
This automation allows for different comfort temperatures to be set for different times of day in a heating schedule. This allows for more control and mimics the original Tado functionality.

There are three triggers in the automation:
1) The `target_temp` attribute (additonal data) of the Room Normal Schedule changes
2) The `target_temp` attribute (additonal data) of the Room Holiday Schedule changes
3) The [Holiday Heating](global_configurations.md#holiday-heating) Helper gets toggled

```
triggers:
  - trigger: state
    entity_id:
      - schedule.ahc_study_schedule_normal
    attribute: target_temp
    id: normal
  - trigger: state
    entity_id:
      - schedule.ahc_study_schedule_holiday
    id: holiday
    attribute: target_temp
  - trigger: state
    entity_id:
      - input_boolean.ahc_holiday_heating
```
Add in a condition that checks if the schedules are on. This stops errors in the logs whereby the `target_temp` attribute disappears when the schedule is off:

<img width="852" height="286" alt="image" src="https://github.com/user-attachments/assets/c0ef71f2-111f-443f-a6d7-94e22129885e" />

The actions that take place depend on the state of the Holiday Heating Helper
1) If the Holiday Heating switch is on, then Set the Comfort temp to be the value of the `target_temp` in the Holiday Schedule
2) If the Holiday Heating switch is of, then Set the Comfort temp to be the value of the `target_temp` in the Normal Schedule

```
actions:
  - choose:
      - conditions:
          - condition: state
            entity_id: input_boolean.ahc_holiday_heating
            state: "off"
        sequence:
          - action: input_number.set_value
            metadata: {}
            data:
              value: >-
                {{ state_attr('schedule.ahc_study_schedule_normal',
                'target_temp') }}
            target:
              entity_id: input_number.ahc_study_comfort_temp
      - conditions:
          - condition: state
            entity_id: input_boolean.ahc_holiday_heating
            state: "on"
        sequence:
          - action: input_number.set_value
            metadata: {}
            data:
              value: >-
                {{ state_attr('schedule.ahc_study_schedule_holiday',
                'target_temp') }}
            target:
              entity_id: input_number.ahc_study_comfort_temp
```
This keeps the temperature set to the value in the correct schedule

## Advanced Heating Control
Each room with a TRV needs an ADC Blueprint Automation configured as follows:
### Thermostats
<img width="1023" height="631" alt="image" src="https://github.com/user-attachments/assets/3a4f878b-0254-414d-b200-dbd09b4f1d5d" />

1. Pick the TRV for the room, all other settings are default. Note that the Room Temperature Sensor must be blank as the Tado TRVs (configured with HomeKit) can't do calibration.

<img width="1034" height="490" alt="image" src="https://github.com/user-attachments/assets/9e66f351-b9a5-464e-941d-202d63691ab6" />

2. In the Temperatures section, set the Eco Temperature to 12 and pick the Comfort Temperature Helper associated with the room.

<img width="1033" height="613" alt="image" src="https://github.com/user-attachments/assets/694de986-27fe-4391-830e-97521473c5d3" />

3. Under Persons, select all members of the household - note that this is now controlled separately to the occupancy for the rest of the house.
4. Set the Guest Mode to use the Home entity

<img width="1028" height="419" alt="image" src="https://github.com/user-attachments/assets/75bc0eaa-1f33-4889-815f-dee5a12b08ad" />

5. Add the two schedules into the Schedules section. It's important that the Normal Schedule is at the top of the list and then the Holiday one, as the next field determines this....
6. Set the Holiday Heating entity in the Scheduler Selector dropdown. This is where the schedule is picked. If Holiday Schedule is Off, then the First Schedule is picked, if it's on, the second Schedule is picked

<img width="1031" height="519" alt="image" src="https://github.com/user-attachments/assets/76c01840-8ce2-4245-be16-020e0119a238" />

7. Under the Away Mode section, toggle the Schdeuler Away Mode to `On`. Notice how there is no Away Temperature Offset configured? This is because this is handled by YAML. Switch to YAML view for the automation and add the following code:
```
    input_away_offset: >
      {% set offset = states('input_number.ahc_away_temp_adjustment') | int %}
      {% set temperature = states(input_temperature_comfort) | float %}
      {% if temperature - offset <= 12 %}
        {{ temperature - 12 }}
      {% else %}
        {{ offset }}
      {% endif %}
```
This code loads the state of the `away_temp_adjustment` helper and the comfort temperature of the room, already defined in the blueprint - `input_temperature_comfort`. It then compares the two, subtracting the offset from the comfort temp. If the result of this is greater than 12 (the minimum temp I want in a room) then it return the offset to ADC. If it's smaller than 12, then it returns the difference to ADC, so that the resulting temperature will be 12C.

<img width="1032" height="262" alt="image" src="https://github.com/user-attachments/assets/ff11130c-5cda-4f9d-b8ed-6181abdc24e5" />

8. Set the Frost Protection. This will force the temperature to be 12C if away for more than a day. Helpful if we're away but within the Proximity distances less than 64km.
   
<img width="1031" height="822" alt="image" src="https://github.com/user-attachments/assets/f4128363-ea44-4368-9bd4-bb6ffa50013d" />

9. Finally, in the On/Off Automation Options, enter the Main Heating Switch in the Winter Mode / Automation Toggle. This is the global switch to control the heating. 
10. Once the Automation is saved, ensure it is tagged with the `Heating` tag and the `Heating Control` tag. The second one is important so that the proximity schedule fires the ADC automations to adjust the temperature based on the distance of people from home.






