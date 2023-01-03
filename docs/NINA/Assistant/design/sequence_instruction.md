---
layout: docs
title: Sequence Instruction
---

# Sequence Instruction

At imaging time, the Assistant is implemented by a single new sequence instruction: **_Assistant_**.  The description below assumes some familiarity with how Sequence Containers work in the NINA Advanced Sequencer.  A Sequence Container (e.g. Deep Space Object Sequence) wraps the following logic into a single entity:
- **Triggers**: logic that runs a test after each instruction execution and then runs trigger-specific logic if the test passed.  Examples include Meridian Flip, any of the Autofocus triggers, center after drift, etc.
- **Loop Conditions**: Control iteration for the contained instructions.  Examples include Loop For Time, Loop Until Time, etc.
  - Note that some 'loop conditions' work differently.  Rather than controlling iterations of container instructions, they serve to periodically check some condition and then potentially interrupt and cancel the instructions.  An example is Loop While Safe which will check the Safety Monitor every few seconds and immediately abort if unsafe conditions are detected - without waiting for the currently executing instruction to end.
- **Instructions**: A set of instructions to execute.  These are executed in order for sequential containers.  Examples include unpark, slew/center on a target, switch filters, take an exposure, etc.  When done and the loop end conditions have not been met, the instructions execute again.

The **_Assistant_** instruction extends the base Sequential Instruction Set container.  It operates as follows:
1. Run the [Planning Engine](planning_engine.html) and get the next target to image.  End execution if none are returned.
2. Set the target details on the container (similar to the Deep Space Object Sequence container) to support the sequencer UI.
3. If the start time for the target is in the future, wait for the start time.
4. Set a trigger on the container to abort this target when the hard stop time is reached, taking into account the likely duration of the next instruction to execute.
5. Add a slew/center/rotate instruction to the instruction list.
6. Add the switch filter and exposure instructions for each planned exposure, with appropriate dither handling.
7. Enable monitoring of image save events so that the image grader can run and the acquired images for this project/target can be updated.
8. Execute the container.
9. Loop back to step 1.

If the sequence is stopped in the NINA UI, it will begin again at step 1 when resumed.  Not only is this the simplest approach, it also makes sense since the time and conditions may have changed.

This approach leverages the existing features and sophistication of the NINA Advanced Sequencer to simplify the implementation.  However, it's not unique - other core instructions in NINA decompose an operation into several steps, each implemented by a different existing instruction, trigger, or condition.

A key feature is that the logic for any sequence containers that are _parents_ to the **_Assistant_** container continue to work normally.  For example, any trigger on a parent container will still run after the execution of each **_Assistant_** instruction.  So the triggers you normally use to handle various tasks - autofocus, meridian flip, center after drift, etc - will continue to work.  The **_Assistant_** itself just focuses on getting pointed to the selected target and taking the planned exposures.

The **_Assistant_** instruction has the potential to significantly simplify sequences since the details of the required exposures and the target timing are managed elsewhere or automatically determined.  A perfectly valid sequence might be:
````
- Regular start instructions (connect equipment, cool camera, unpark, calibrate guiding, etc)
  - Sequential Instruction Set container:
    - Triggers:
        autofocus, meridian flip, center after drift, etc
    - Loop Conditions:
        Loop While Safe (if using a Safety Monitor)
    - Instructions:
        Assistant 
- Regular end instructions (warm camera, park, disconnect equipment, etc)
````

Since the Assistant will manage the start/stop times for each selected target based on selected criteria and twilight, there's no real need to manage that explicitly in the sequence.

### What your Sequence Still Needs
The following is a sample of the sequence logic you'll still want.  It's not meant to be comprehensive or all-inclusive.  For example, triggers or instructions provided by other installed plugins will probably work just fine.
- Your standard Sequence Start instructions to prepare the platform for imaging.
- A sequential container with:
  - Your regular triggers to control meridian flipping, autofocus, center after drift, restore guiding, etc, etc.
  - Loop While Safe condition (if using a Safety Monitor)
  - Assistant instruction
- Flat exposures - for now, the plugin doesn't concern itself with flats but see [Advanced Topics / Future Enhancements](advanced_topics.html).
- Your standard Sequence End instructions to prepare the platform for shut down.

### What your Sequence No Longer Needs

With this approach, you no longer need the following in your sequences:
- Deep Space Object Sequence container.  The target is managed dynamically since the Planning Engine returns the target coordinates for slew/center/rotate.
- Looping conditions.  The exposure plans for the selected target will automatically schedule exposures for the duration of the time available on the target.
- Timing conditions.  The start/stop times for each target, as well as the entire night, are managed automatically by the Planning Engine and injected into the Assistant container.
- Dither trigger or instructions.  Dithering will be handled automatically based on your preferences.
