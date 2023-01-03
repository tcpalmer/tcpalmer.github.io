---
layout: docs
title: Plugin UI
---

# Plugin UI
The plugin will provide user interfaces at two points.

## Plugin Page: Project Management

The **_Assistant_** plugin page will provide the UI to manage Projects, Targets, ExposurePlans, and Preferences.
- Ability to create, edit, copy, delete Projects.
- Ability to add Targets to a Project.  Target coordinates can be manually entered or loaded via one of the following sources:
  - NINA DSO catalog lookup
  - A Target saved in the Advanced Sequencer (which typically originated in the Framing Assistant)
  - Connection to planetarium software
- Ability to add ExposurePlans to a Target and edit them.  The total desired and accepted counts can be manually edited.
- Ability to edit Preferences.

Other thoughts:
- For active and completed projects, you should be able to see a summary of project status including:
  - Total time on each target
  - Total counts for the ExposurePlans for each target: desired, acquired, accepted
  - Total imaging time (hh:mm) for each ExposurePlan/filter
  - Sum totals for all targets for the project

## Sequence Instruction: Current Target and Progress
The [Assistant Sequence Instruction](sequence_instruction.html) will have a UI in the Advanced Sequencer similar to the existing Deep Space Object Sequence container.  It will show the coordinates for the active Target plus the standard nighttime altitude chart.  It will not show the typical container slots for Triggers, Loop Conditions, and Instructions since those will be managed programmatically by the Assistant instruction.

Some indication of the progress made on the current target must be displayed.  This could be as simple as a log showing exposures or a more comprehensive display - perhaps similar to the existing Exposure Info table in the Deep Space Object Sequence container but showing the exposure counts as managed by the plugin.

UI for the mini display in the Sequence view on the Imaging screen is TBD.
