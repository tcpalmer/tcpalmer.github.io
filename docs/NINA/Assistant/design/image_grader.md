---
layout: docs
title: Image Grader
---

# Image Grader

When you configure the exposure plans for a target, you specify the total number of images you desire for that filter.  The plugin manages two other counts associated with each exposure plan:
- Total acquired images (read only)
- Total number of 'acceptable' acquired images

As each acquired image is saved, the plugin is notified so it can update the image counts for the target's exposure plans.  There are two approaches to managing the number of acceptable images:
- With the image grader disabled, you inspect the acquired images after an imaging session and manually update the Accepted count for the applicable exposure plans.  You _must_ do this - otherwise the planner will continue to request exposures for the target since the number of acceptable images never reaches the number desired.
- With the image grader enabled, you can let it manage the number of acceptable images.  For each image determined to be acceptable, it will increment the Accepted count for that exposure plan.

## Grading Approach
The approach to grading images is TBD but will likely use HFR, star counts, and sky quality metrics (if available).  Algorithmic grading of images is inherently problematic and qualitative and may also depend on deviation from a small set of comparison images.  For these reasons, the grader does not delete images it determines to be unacceptable - it just doesn't increment the Accepted count.  You're still free to adjust the Accepted count manually and/or increase the total number desired.  Ultimately, rejection of images should only occur during postprocessing.
