---
layout: docs
title: Assistant Design
---

The **Assistant** NINA Plugin is designed to provide a higher level of imaging automation than typically achievable today with NINA.  Specifically, it maintains a database of imaging projects describing DSO targets and associated exposure plans.  Based on various criteria and preferences, it can decide at any given time what project/target should be actively imaging.  A user will enter the desired projects, targets, and preferences into a UI exposed by the plugin.  At runtime, a single new instruction for the NINA Advanced Sequencer will interact with the planning engine to determine the best target for imaging at each point throughout a night.  The instruction will manage the slew/center to the target, switching filters, taking exposures, and dithering - all while transparently interacting with the surrounding NINA triggers and conditions.

The targets for a project also maintain the number of images you would like to acquire per filter, the number acquired, and the number of those that are 'acceptable'.  Projects remain active until the number of acceptable images equals or exceeds the number desired.  The plugin also includes a simple **_image grader_** that can proxy for 'acceptable' based on HFR, star counts, etc.  No images are deleted by the grader - it merely adjusts the acceptable count so that the planning engine will keep requesting exposures for that filter until acceptable equals desired.  The grader can be disabled and the acceptable count can be managed manually to support external image grading approaches.

# Key Concepts
- A **_Project_** models a single imaging effort.  Projects contain metadata as well as one (single frame projects) or many **_Targets_** (mosaics).
- A **_Target_** models a single DSO or other (non-solar system) object of interest.  It contains the RA and Dec coordinates as well as a frame rotation angle and a ROI (region of interest).  Targets also reference a set of one or more **_FilterPlans_** that provide the exposure details for each applicable filter.
- A **_FilterPlan_** provides the details that will drive a set of exposures for a single filter via the NINA Advanced Sequencer (filter name, exposure length, gain, offset, binning).
- **_Preferences_** let a user specify various criteria that drive the selection of appropriate targets as well as the details for exposures.
- The **_Planning Engine_** reads the project database and, based on the current time, location, and preferences, decides which of multiple potential targets it should be imaging.

# Topics
- [Preferences](preferences.html)
- [Planning Engine](planning_engine.html)
- [Assistant Sequence Instruction](sequence_instruction.html)
- [Plugin UI](plugin_ui.html)
- [Image Grader](image_grader.html)
- [Database](database.html)
- [Advanced Topics / Future Enhancements](advanced_topics.html)
