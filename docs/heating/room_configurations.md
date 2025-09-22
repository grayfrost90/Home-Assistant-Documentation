# Room Configurations
Each room requires a set of helpers and automations for the heating to work. This is primarily driven through the Advanced Heating Control (ADC) Blueprint.

## Helpers
There are 3 helpers that a room requires:
1) [Comfort Temp](#comfort-temp). Used by ADC to set the target temperature for the room
2) [Schedule Normal](#schedule-normalholiday). A Schedule helper that tells ADC when to turn the heating on, if the [Heating Mode](global_configurations.md#heating-mode) is set to Normal
3) [Schedule Holiday](#schedule-normalholiday). A Schedule helper that tells ADC when to turn the heating on, if the [Heating Mode](global_configurations.md#heating-mode) is set to Schedule

### Comfort Temp
This is an input_number helper and should be configured as follows:
<img width="547" height="586" alt="image" src="https://github.com/user-attachments/assets/bc46b1b8-e04e-4cd6-91f9-78599221cb77" />

### Schedule Normal/Holiday
Two schedules for heating as explained. These define the times, and importantly temperature, when the heating is turned on. It's important that each time block has Additional data consisting of `target_temp: temp` where `temp` is the desired temperature for that time:

<img width="394" height="574" alt="image" src="https://github.com/user-attachments/assets/cd239547-83cd-4b64-967d-b15e886f89c5" />

This additional data is used later in the [target temp](#target-temp) automation

## Automations
Each Room requires the following automations:
1) [Target Temp](#target-temp). Used to adjust the comfort temp helper that ADC uses to set the room temperature to.
2) [Advanced Heating Control](#advanced-heating-control). The main component that controls the climate entities, this is configured from a Blueprint

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
### Advanced Heating Control
