# Global Configurations
## Helpers
There are 4 global helpers that control settings across the whole house:
1. Home
2. Heating Mode
3. Holiday Heating
4. Main Heating Switch

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

## Automations
There are 2 global automations:
1) Heating Mode
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

