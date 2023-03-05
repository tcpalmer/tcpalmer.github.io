---
layout: docs
title: Preferences
---

# Preferences

The plugin supports a set of **_Preferences_** that drive behavior.

The model for preferences is still a work in progress but will likely store preferences applicable at the following levels: project and filter.  Thoughts:
- Project
  - MinimumAltitude: value (in degrees) to use if no custom horizon is available.
  - UseCustomHorizon: use the custom horizon defined for the profile.
  - HorizonOffset: value (in degrees) to add to the custom horizon.
  - FilterSwitchFrequency: value to specify how to repeat filter exposures - see below.
  - DitherEvery: value to specify to dither after every N exposures.
  - EnableGrader: enable/disable the simple [image grader](image_grader.html) for this project.
  - MinimumTime: minimum time (in minutes) that a target must be available for imaging to consider inclusion for a night.
  - Rule weights: a set of values (0-1) that are used to weight the impact of each rule used by the [Planning Engine](planning_engine.html).
- Filter
  - TwilightInclude: the maximum level of dusk/dawn brightness suitable for imaging with a filter: civil, nautical, astronomical, nighttime
  - MoonAvoidanceSeparation: value (in degrees) for the Moon avoidance formula.
  - MoonAvoidanceWidth: value (in days) for the Moon avoidance formula.
  - MaximumHumidity: value (percent) that sets the maximum humidity acceptable for the filter.

FilterSwitchFrequency lets you control how the planning will order exposures, basically repeating a filter for the value.  For example, if you have exposures for LRGB filters included in a plan, then a value of 1 yields LRGBLRGB etc, a value of 2 yields LLRRGGBB, etc.  A value of zero is used to indicate that all planned exposures for a filter should be taken before switching to the next (for example if you don't use filter offsets).This is independent of dithering.  If DitherEvery is not zero, then it will dither at that frequency regardless of the FilterSwitchFrequency value.

The [Moon Avoidance](http://bobdenny.com/ar/RefDocs/HelpFiles/ACPScheduler81Help/Constraints.htm) approach is used instead of separate moon angle and illumination criteria since it encompasses both those in a more flexible model.
