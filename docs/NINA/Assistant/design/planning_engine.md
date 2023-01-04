---
layout: docs
title: Planning Engine
---

# Planning Engine

The Planning Engine executes at the request of the [Assistant Sequence Instruction](sequence_instruction.html).  Based on the current time, location, and the state of the projects/targets/preferences in the plugin database (which is per profile), the engine executes and determines the best target to image at that point.  When imaging for one target ends, the Sequence Instruction again invokes the Planning Engine to select the next target.  The current target (or null) is passed in so that the engine knows the current state.

Engine execution consists of a number of passes that refine the set of potential targets, ultimately selecting one and determining the best sequence of filters/exposures.

## Pass 1
Pass 1 determines the set of targets that could _potentially_ be imaged at the current time.  An active target is one that is incomplete, is in the Active state, and the current date is within the project begin/end dates.
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
- Beginning at the earliest start time, walk the timeline (programmatically) forward in time.  If only a single target is available at that point, then it is the selected target (scoring step not needed).
- Otherwise, there are two or more targets available so a decision has to be made on whether to continue with the current target or switch to another.  A set of rules will be applied that determine an overall score for each potential target. Sample rules:

    - Project Priority: score based on low/normal/high priority of the project
    - Time Limit: assign a higher score to targets that are setting 'soon' or 'near' the western meridian window limit.  This helps to avoid missing opportunities to image a target before it sets for the year.
    - Season Limit: assign a higher score to targets that have shorter remaining imaging seasons.
    - Meridian Flip: assign a lower score to targets that will require an immediate MF.  Related: if a target is east of the meridian but 'close' (check NINA profile MF settings), don't switch to it until it's well past the meridian.
    - Percent Complete: assign a higher score to targets that are closer to 100% complete to wrap them up.
    - Switch penalty: assign a higher score to the current target (if it's stop time is still in the future) and lower scores to others.  This assigns some cost to the time to switch to another target (slew/center) and ideally prevents target thrashing.
    - Mosaic completion priority: assign a higher score to mosaic targets that are closer to 100% complete to wrap them up.
    - Mosaic balance priority: assign a higher score to mosaic targets that are closer to 0% complete to balance exposures across frames.  (Obviously in conflict with Mosaic completion priority so only one should be used.)

Each rule will have an associated weight maintained in the preferences.  The final score for a target is just the sum of the weighted rule scores, with the highest scoring target winning.  It should be straightforward to add additional rules in the future with this approach.  Plus, the weighting system makes it easy to completely disable a rule by setting its weight to zero.

### Setting the Hard Stop Time
Regardless of whether only a single target was available or whether it was selected by scoring, the hard stop time of that target must be set - in essence, determining the time that the engine will be called again to select the next target.  The only way to properly do this is to continue the selection process:
- If only this single target was possible, then find the next point in time when another target becomes available and set the hard stop to that time.
- If multiple targets were available, setting the hard stop time is trickier.  One simple approach is just to let the target image for some minimum amount of time.
- Of course, if the target has an earlier hard stop time due to horizon or twilight, that time must be used instead.

It's important to note that the cost of engine invocation is low (performance-wise).  Also, if the engine selects the same target again, the slew/center would be skipped so there's really no penalty in letting the engine decide again.  And at that future decision point, it may well select different exposure plans based on what was previously acquired.

Another consideration for hard stop time is a twilight change event which might change the exposure plans possible.

## Pass 4
Pass 4 refines the order of filters/exposures for the selected target:
- If the time span on the target includes twilight, order so that exposures with more restrictive twilight are not scheduled during those periods.  For example a NB filter set to image at astro twilight could be imaged during that time at dusk and dawn, but a WB filter could not and would require nighttime darkness before imaging.
- Consideration may also be given to prioritizing filters with lower percent complete so that overall acquisition on a target is balanced.

The final planning engine output consists of the selected target with associated ordered exposure plans.  The target as a whole and each exposure plan has a hard start time and a hard stop time.  These are used by the [Assistant Sequence Instruction](sequence_instruction.html) to establish the time limits for imaging the target:
- If the hard start time is in the future, the Sequence Instruction will effectively sleep the sequence until that time.
- Since imaging time is never simply the number of exposures times exposure time, few targets will actually complete the plan before the hard stop time.  To prevent exceeding the planned time, the Sequence Instruction will programmatically add a trigger to abort the target at the hard stop time.
- Will also have to handle the case where a wait may be required in the middle of a set of exposures.  For example, if only a couple of NB images remain and are scheduled for astro twilight, we may have to delay until nighttime before starting on WB images.

## Potential Issues
- For mosaics or other set of close targets, the selection process will probably be biased towards imaging those whose coordinates rise above the horizon first (if twilight isn't the limiting factor).  Once that target is selected, it may well continue to be selected to avoid target thrashing.  This might not be a bad thing, but it does have the potential to counteract any rules regarding mosaic frame priority.
