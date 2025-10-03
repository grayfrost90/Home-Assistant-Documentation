# Heating Control Usage
- [Heating schedules](#heating-schedules)
- [Heating Mode](#heating-mode)
- [Away behaviour](#away-behaviour)
- Manual override

## Heating Schedules
Each room has two schedules, Normal and Holiday. Normal are the main schedules, Holiday are run when in Holiday mode, e.g. at Christmas time. To edit these, in Home Assistant do the following (needs Admin rights):
1. Browse to Settings > Devices & services > Helpers
2. Find the schedule to edit, e.g. AHC - Dining Room Schedule Normal, and open it
3. Click the cog in the top right corner, this will open the weekly schedule
4. By clicking on a blue section, the time and `target_temp ` can be adjusted. Alternatively, the times can be edited by dragging the schedule
5. **IMPORTANT!** each schedule block must have a `target_temp ` set in the additional data section:
   <img width="557" height="682" alt="image" src="https://github.com/user-attachments/assets/f4ad5fda-dc16-4aaa-8624-133f47c1404e" />

   this is used by the automations to set the target temperature for the schedule block

## Heating Mode
The heating mode of the system can be set to three options:
1. Normal
2. Holiday
3. Off

This control can be found in two places:
1. The occupancy screen:
  <img width="911" height="318" alt="image" src="https://github.com/user-attachments/assets/63900573-00fe-4a37-905b-2b32a4f15efc" />

2. The heating screen:
  <img width="877" height="412" alt="image" src="https://github.com/user-attachments/assets/c948439d-b550-413e-8f49-5d14ecce72cc" />
  
## Away Behaviour
There are a few factors that change and affect what happens to the heating when no one is at home. In order for the heating to enter Away Mode, either of the following states need to be true:
1. **Person detection:** If all members of the household are deemed to be away AND the house occupancy is on Auto (note that people need to added to both the household group AND each AHC Room Automation for this to work)
2. **House Occupancy Mode:** If the house occupancy mode is manually set to Away or Vacation
3. **Heating Schedule:** The heating needs to be running either the Normal or Holiday schedules

Upon entering away mode, the heating will do the following:
1) **Proximity:** As the nearest person to home moves away from home, the target temperature of all rooms will drop, eventually reaching 12C, if they are far enough from home
2) **Time:** Regardless of proximity, if all people are away from home for more than 1 day, then the target temperature of all rooms will be set to 12C. This will result in the house not pre-heating on return **(something I need to look at)**
