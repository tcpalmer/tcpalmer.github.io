---
layout: docs
title: Planning Engine
---

# Planning Engine

The Planning Engine executes at the request of the [Assistant Sequence Instruction](sequence_instruction.html).  Based on the current time, location, and the state of the projects/targets/preferences in the plugin database (which is per profile), the engine executes and determines the best target to image at that point.  When imaging for one target ends, the Sequence Instruction again invokes the Planning Engine to select the next target.

Engine execution consists of a number of passes that refine the set of potential targets, ultimately selecting one and determining the best sequence of filters/exposures.

## Pass 1
Pass 1 determines the set of targets that could _potentially_ be imaged at the time.  An active target is one that is incomplete, is in the Active state, and the current date is within the project begin/end dates.
- For each active project:
  - For each unfinished target, based on date and location: 
    - Find most inclusive twilight over all incomplete exposure plans 
    - Is it visible (above horizon) for that begin/end twilight period? 
    - Is the time-on-target > minimum imaging time? 
    - If meridian window is in use, is the time in the window > minimum imaging time? 
    - If yes to all, add to potential target list

Potential targets have hard start/stop times determined by twilight, horizon, or meridian window.

## Pass 2
Pass 2 applies the Moon Avoidance formula for each filter of each potential target, removing those exposures that fail the check.
- For each potential target:
  - For each incomplete exposure plan:
      - If enabled, determine moon avoidance - acceptable?  Moon position can be calculated based on the midpoint of start/end times for this target.
      - If yes, add to list of exposure plans for this target

If all exposure plans for a target were culled, remove it from the potential targets list.  Revise the hard start/stop times based on the final set of exposure plans since it may shift with different twilight preferences per remaining filters.


## Pass 3
Pass 3 determines the best single target to image at the current time.
- Order the list of potential targets by start time.
- Walk the timeline (programmatically) from the earliest start time to the latest end time.  At any point along that timeline, you could have a situation where two or more targets could potentially be active.  When those points are hit, make a decision on whether to continue with the current target or switch to another.  A set of rules will be applied that determine an overall score for each potential target. Sample rules:
    - Project Priority: score based on low/normal/high priority of the project
    - Time Limit: assign a higher score to targets that are setting 'soon' or 'near' the western meridian window limit.  This helps to avoid missing opportunities to image a target before it sets for the year.
    - Season Limit: assign a higher score to targets that have shorter remaining imaging seasons.
    - Meridian Flip: assign a lower score to targets that will require an immediate MF.  Related: if a target is east of the meridian but 'close' (check NINA profile MF settings), don't switch to it until it's well past the meridian.
    - Percent Complete: assign a higher score to targets that are closer to 100% complete to wrap them up.
    - Mosaic completion priority: assign a higher score to mosaic targets that are closer to 100% complete to wrap them up.
    - Mosaic balance priority: assign a higher score to mosaic targets that are closer to 0% complete to balance exposures across frames.  (Obviously in conflict with Mosaic completion priority so only one should be used.)

Each rule will have an associated weight maintained in the preferences.  The final score for a target is just the sum of the weighted rule scores, with the highest scoring target winning.

It should be straightforward to add additional rules in the future with this approach.  Plus, the weighting system makes it easy to completely disable a rule by setting its weight to zero.

## Pass 4
Pass 4 refines the order of filters/exposures for the selected target:
- If the time span on the target includes twilight, order so that exposures with more restrictive twilight are not scheduled during those periods.  For example a NB filter set to image at astro twilight could be imaged during that time at dusk and dawn, but a WB filter could not and would require nighttime darkness before imaging.
- Consideration may also be given to prioritizing filters with lower percent complete so that overall acquisition on a target is balanced.

The final planning engine output consists of the selected target with associated ordered exposure plans.  The target as a whole and each exposure plan has a hard start time and a hard stop time.  These are used by the [Assistant Sequence Instruction](sequence_instruction.html) to establish the time limits for imaging the target:
- If the hard start time is in the future, the Sequence Instruction will effectively sleep the sequence until that time.
- Since imaging time is never simply the number of exposures times exposure time, few targets will actually complete the plan before the hard stop time.  To prevent exceeding the planned time, the Sequence Instruction will programmatically add a trigger to abort the target at the hard stop time.
- Will also have to handle the case where a wait may be required in the middle of a set of exposures.  For example, if only a couple of NB images remain and are scheduled for astro twilight, we may have to delay until nighttime before starting on WB images.
