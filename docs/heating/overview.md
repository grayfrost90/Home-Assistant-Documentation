# Heating Overview using Advanced Heating Control Blueprint
### Overview

The heating setup uses Tado TRVs and has normally all been controlled by Tado. This has worked fine but in autumn 2025 I decided to attempt to migrate the control all locally to Home Assistant for the following reasons:
1. Tado charge an annual fee for advanced features - like Auto Home/Away geolocation switching - which, although is only £30 a year, this has gone up in the past and seems an unnecessary cost when I already pay for Nabu Casa (which I'm happy with)
2. Tado announced rate throttling on their API, which Home Assistant uses, meaning I would have to change the integration to use the (local) HomeKit one to reduce API calls. Even though you get more API calls with the subscription, I have 9 TRVs plus the hot water, which would soon add up
3. If I'm having to setup HomeKit, I may as well look at other ways to achieve my heating schedule because....
4. Tado relies on an internet connection, so if the internet is down, the schedules don't work. Moving all the schedules to Home Assistant would remove this dependency

### New Architecture
<img width="1190" height="738" alt="image" src="https://github.com/user-attachments/assets/5a212a3a-dbbf-4946-bf1d-52038f6518e4" />

The TRVs are controlled by the HomeKit integration in Home Assistant - this is all local control on the LAN. Unfortunately, Hot Water control still needs to go via the Tado Cloud integration as this entity does not exist in the HomeKit integration.

### Configuration
The heating control is split into two main categories:
1. [Global configuration](global_configurations.md) helpers and automations. These adjust settings that are common to all room based automations
2. [Room configurations](room_configurations.md). Controlled via the [Advanced Heating Control](https://github.com/panhans/HomeAssistant/blob/main/blueprints/automation/panhans/advanced_heating_control.yaml) (AHC) automation blueprint, and the helpers/automations to support it. These are replicated for each room with a Tado TRV

### Setting up a new room/migrating from Tado
To move a room from being Tado controlled to local, follow these steps: [Migrating a Room/TRV](migrating.md)
