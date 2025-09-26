# Heating Control Usage
- [Heating schedules](#heating-schedules)
- [Heating Mode](#heating-mode)
- Away behaviour
- Manual override

## Heating Schedules
Each room has two schedules, Normal and Holiday. Normal are the main schedules, Holiday are run when in Holiday mode, e.g. at Christmas time. To edit these, in Home Assistant do the following (needs Admin rights):
1. Browse to Settings > Devices & services > Helpers
2. Find the schedule to edit, e.g. AHC - Dining Room Schedule Normal, and open it
3. Click the cog in the top right corner, this will open the weekly schedule
4. By clicking on a blue section, the time and `target_temp ` can be adjusted. Alternatively, the times can be edited by dragging the schedule
5. IMPORTANT! each schedule block must have a `target_temp ` set in the additional data section:
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
  
