---
layout: docs
title: Preferences
---

# Preferences

The plugin supports a set of **_Preferences_** that drive behavior.

The model for preferences is still a work in progress but will likely store preferences applicable at the following levels: project and filter.  Thoughts:
- Project
  - UseCustomHorizon: use the custom horizon defined for the profile.
  - HorizonOffset: value (in degrees) to add to the custom horizon.
  - MinimumAltitude: value (in degrees) to use if no custom horizon is available.
  - EnableGrader: enable/disable the simple [image grader](image_grader.html) for this project.
  - MinimumTime: minimum time (in minutes) that a target must be available for imaging to consider inclusion for a night.
  - Rule weights: a set of values (0-1) that are used to weight the impact of each rule used by the [Planning Engine](planning_engine.html).
- Filter
  - TwilightInclude: the maximum level of dusk/dawn brightness suitable for imaging with a filter: civil, nautical, astronomical, nighttime
  - MoonAvoidanceSeparation: value (in degrees) for the Moon avoidance formula.
  - MoonAvoidanceWidth: value (in days) for the Moon avoidance formula.
  - MaximumHumidity: value (percent) that sets the maximum humidity acceptable for the filter.

The [Moon Avoidance](http://bobdenny.com/ar/RefDocs/HelpFiles/ACPScheduler81Help/Constraints.htm) approach is used instead of separate moon angle and illumination criteria since it encompasses both those in a more flexible model.

## Other Potential Preferences
- We'll need some way to indicate the preferred order of taking exposures for different filters.  For example those without filter offsets would likely prefer to take a large number for each filter before switching to minimize the cost of autofocus events.  Those with offsets may prefer orders like RRGGBBLL.
- The above will also involve a preference to indicate when, in a sequence of exposures, to dither.  If imaging without filter offsets, you would likely still want to dither after a few exposures of the same filter.  With offsets, you might only dither after a RRGGBBLL sequence for example.

