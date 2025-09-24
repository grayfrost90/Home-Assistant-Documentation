# Migrating a Room/TRV
These are the steps to take to migrate a room from being Tado controlled to being locally controlled via Home Assistant. This is specific for my setup:

### Prerequisties
- TRVs are discovered with the [HomeKit](https://www.home-assistant.io/integrations/homekit_controller) integration
- [Proximity](https://www.home-assistant.io/integrations/proximity) integration is configured for all members of the household
- [Uptime](https://www.home-assistant.io/integrations/uptime) integration is configured (AHC requires this for some functionality, although I don't think my setup needs it)

### Process
#### Tado App Changes
1) Change the Schedule for the room in the Tado App to be 'Mon, Tue, Wed, Thu, Fri, Sat, Sun'. This should result in the room being Off all day, every day
2) Turn off the Away Temperature. I'm not sure this is needed, but do it for completeness
3) Ensure Geofencing is set to Home. This is a one time thing for the whole of Tado

#### Home Assistant Setup
4) Disable the Zone for the room in the Tado integration 
5) Add the [helpers](room_configurations.md#helpers) for the room
    - Copy all the schedules from the Tado app to the schedule helpers as follows:
        - Mon-Fri, Sat, Sun > Normal
        - Mon-Sun > Holiday
6) Add the [automations](room_configurations.md#automations) for the room
7) Edit the Home Dashboard and update the card for the room to point to the HomeKit TRV for the following sections:
    - Temperature
    - Badge
    - Climate Control
8) Edit the Room Dashboard and update the the following cards to point to the HomeKit TRV:
    - Temperature
    - Humidity
    - Climate
9) Edit the Heating Dashboard and update the climate card with the HomeKit TRV
