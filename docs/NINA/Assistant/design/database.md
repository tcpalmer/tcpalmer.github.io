---
layout: docs
title: Database
---

# Database

SQLlite will be used for the Assistant database:
- Simple, robust, minimal SQL database that NINA already uses (DSO catalog).
- [DB Browser for SQLite](https://sqlitebrowser.org/) for database inspection.
- NINA provides an example of how you can perform database migration if schema changes with new plugin versions.
- Supports multiple concurrent processes having access to same database file.  One process will lock it for the duration of a write operation but should work fine.
- Database will be saved to %localappdata%\NINA\Plugins\AssistantPlugin\.

## Databases and NINA Profiles
Conceptually, there is a separate database for each NINA profile since projects will likely depend on the equipment configured for that profile.

However, we could have a single physical database but with the profile ID used to segregate projects/targets/etc.  This would have the following advantages:
- It's conceptually simpler and easier to back up.
- It would make it easier to copy projects/targets/prefs/etc from one profile set to another.
- It might make it easier to have options for users with multiple synced instances of NINA so that the instances could share projects.

## Model

At a high level, the database model supports the following:
- One or more **_Projects_**, each modeling a single imaging effort.  Projects contain metadata as well as one (single frame projects) or many **_Targets_** (mosaics).
- A **_Target_** models a single DSO or other (non-solar system) object of interest.  It contains the RA and Dec coordinates as well as a frame rotation angle and a ROI (region of interest).  Targets also reference a set of one or more **_FilterPlans_** that provide the exposure details for each applicable filter.
- An **_FilterPlan_** provides the details that will drive a set of exposures for a single filter via the NINA Advanced Sequencer (filter name, exposure length, gain, offset, binning).  Each FilterPlan also supports the following total counts:
  - Desired: total number of exposures desired for this filter.
  - Acquired: total number of exposures actually taken for this filter.
  - Accepted: number of _accepted_ exposures taken for this filter.  Definition of accepted exposures is discussed in the [Image Grader](image_grader.html).
- **_Preferences_**
  - Each preference row in the table only stores a JSON string which is deserialized to yield the actual preference model object.  This is done to keep the preference model flexible since this is certain to be an area of very active early development.
  - [Preference](preferences.html) modeling and persistence are very much TBD.
- **_AcquiredImages_** persist the metadata associated with each saved image, primarily for use by the [Image Grader](image_grader.html).

## Tables

### Project
- Name: string
- Description: string
- State: Draft, Active, Inactive, Closed
- Priority: Low, Normal, High
- ProfileId: string id of the associated NINA profile
- CreateDate: date/time
- BeginDate: date/time
- EndDate: date/time
- ActiveDate: date/time
- InactiveDate: date/time
- 1-Many -> Target

### Target
- Name: string
- RA: real
- Dec: real
- Epoch: JNOW or J2000
- Rotation: real
- ROI: percentage (only applies if the camera supports subsampling)
- 1-Many -> FilterPlan

### ExposurePlan
- FilterName: string (_must_ match a filter name configured into NINA for this profile)
- Exposure: real
- Gain: int
- Offset: int
- Binning: int
- ReadoutMode: int
- Desired: int
- Acquired: int
- Accepted: int

It remains to be seen how we would support different parameters like gain/offset/binning for the _same_ filter under different circumstances.

### ProjectPreferences
- Id
- Preferences: JSON string

### FilterPreferences
- Id
- ProfileId: string id of the associated NINA profile
- FilterName: string
- Preferences: JSON string

### AcquiredImage
- Id
- ProjectId
- TargetId
- AcquiredDate
- FilterName
- Metadata: JSON string


## Other Persistence Needs
We might also need to persist the following information:
- Have a table that records each unique filter used throughout an imaging session.  This could be used by a new Assistant Flat instruction to automatically take the flats needed for that night.  Would also need rotation angle if applicable.
- Have a table that records what the planner did (and how well) on any given night: targets imaged, time on target/exposure time, time spent in slew/centers, how many MFs, etc.  Ideally, should be able to determine productive time ratio: total exposure time/total other time.
