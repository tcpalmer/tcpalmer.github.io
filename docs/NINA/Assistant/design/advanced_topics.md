---
layout: docs
title: Advanced Topics
---

# Advanced Topics / Future Enhancements

## Running Multiple NINA Instances
Some imagers have multiple OTAs, filter wheels, and cameras on a single mount.  This poses challenges for dithering since cameras can't be exposing during a dither operation.

The existing Synchronization plugin provides instructions that can support this scenario and synchronize operations across instances.  Typically, a 'master' NINA instance will connect to the mount and the guider.  Both the master and a 'slave' instance will connect to separate cameras and will each be taking exposures.  When the master determines that it's time to dither, it can wait for the slave to complete an exposure, have it pause, and then execute it.

The **_Assistant_** plugin can possibly operate in such a scenario but details are TBD.  Potential approaches:
- Assuming each NINA instance is using a separate profile, then each will also have a separate Assistant database with separate projects, targets, etc.  You could manually duplicate identical projects, targets, and filter plans but set the total count of desired images in each to half the actual total desired.  Each instance would then contribute to the grand total desired for each filter.
- If it is possible to specify that a given NINA profile shares the Assistant database with another profile, then each NINA instance would just work on the same project/target and potentially acquire the total desired in half the time.

There are complexities that must be solved to use either of these approaches.  For example, care must be taken to ensure that the Planning Engine for each instance is selecting the exact same target and filter plans in each instance at the same time.

## Flats
After using the Assistant to image various targets, it would know the set of filters that were actually used throughout the night and the information could be saved in the database.  Perhaps another instruction could be added to take the applicable flat exposures.  It could work with existing instructions in a sequence like the following:
- Open Flat Panel Cover
- Assistant version of Trained Flat Exposure
- Close Flat Panel Cover

This new instruction would read the list of filters used for the most recent session and then generate flats for each using the existing logic of the Trained Flat Exposure instruction.

In the case of synced instances of NINA, the set of filters used would be the same but each camera would have to take separate sets of flats.
