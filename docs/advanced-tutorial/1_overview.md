---
layout: default
title: Overview
parent: Advanced Tutorial
---

#### Table of contents

1. [Setting up](#setting-up): how to import resource files
  + Planning an experiment.
2. [Creating trial template](#creating-trial-template): how to reuse PennController code
  + Using a CSV (comma-separated values) file to organize experimental items.
3. [Customizing the experimental trial](#customizing-the-experimental-trial): how to increase the complexity of a trial
  + Selecting an image with a mouse click.
  + Creating a timeout to end a trial early.
  + Adding a delay to the start of a trial.
  + Manipulating multimedia content visually with CSS styles
  + Saving experiment results before the end of an experiment
4. [Counterbalancing](#counterbalancing): how to include counterbalancing in the experimental design 
  + Shuffling image position.
  + Manipulating trial sequence to randomize experimental item order.
  + Modifying the experiment URL to manually control group assignment.
5. [Collecting participant info](#collecting-participant-info): how to elicit and log participant responses
  + Creating an obligatory checkbox for a consent form.
  + Creating a global variable to record participant IDs.
6. [Examining-data](#examining-data): how to use R to read in a results file
  + Using the [tidyverse](https://www.tidyverse.org/){:target="_blank"} to transform data (optional).

Each section ends with a <span class="text-delta"><a href="#">Back to top</a></span> link, and you can hover to the left of a section title for a shortcut link to that section.

---

## {{ page.title }}

In the **Advanced Tutorial**, you'll learn how to make a picture matching experiment with the following structure:

1. Consent form with:
    + Checkbox to indicate consent
    + Button to continue
2. Instructions screen with:
    + Text input box for participants to enter their ID
    + Button to start the experiment
2. Four experimental trials:
    1. One-second delay
    2. A sentence plays as audio and unfolds as text on the screen.
    3. Two images are printed to the screen next to each other.
    4. Participant clicks on an image or presses a key to select an image.
    5. Trial timeout (trial ends if the participant does not select an image before audio playback finishes).
3. Exit screen

<div class="dotted-grey-dk-000 px-4" markdown="1">
Preview the **AdvancedTutorial** experiment:

<p class="text-delta collapsible-block-title">
  <a href="https://expt.pcibex.net/ibexexps/angelicapan/AdvancedTutorial/experiment.html" target="_blank">Click to take the experiment</a>
</p> 

{% capture content %}
```js
// This is the AdvancedTutorial experiment.
// Type code below this line.

// Remove command prefix
PennController.ResetPrefix(null)

// Turn off debugger
DebugOff()

// Control trial sequence
Sequence("consent", "instructions", randomize("experimental-trial"), "send", "completion_screen")

// Consent form
newTrial("consent",
    newHtml("consent_form", "consent.html")
        .cssContainer({"width":"720px"})
        .checkboxWarning("You must consent before continuing.")
        .print()
    ,
    newButton("continue", "Click here to continue")
        .center()
        .print()
        .wait(getHtml("consent_form").test.complete()
                  .failure(getHtml("consent_form").warn())
        )
)

// Instructions
newTrial("instructions",
    defaultText
        .cssContainer({"margin-bottom":"1em"})
        .center()
        .print()
    ,
    newText("instructions-1", "Welcome!")
    ,
    newText("instructions-2", "In this experiment, you will hear and read a sentence, and see two images.")
    ,
    newText("instructions-3", "<b>Select the image that better matches the sentence:</b>")
    ,
    newText("instructions-4", "Press the <b>F</b> key to select the image on the left.<br>Press the <b>J</b> key to select the image on the right.<br>You can also click on an image to select it.")
    ,
    newText("instructions-5", "If you do not select an image by the time the audio finishes playing,<br>the experiment will skip to the next sentence.")
    ,
    newText("instructions-6", "Please enter your ID and then click the button below to start the experiment.")
    ,
    newTextInput("input_ID")
        .cssContainer({"margin-bottom":"1em"})
        .center()
        .print()
    ,
    newButton("wait", "Click to start the experiment")
        .center()
        .print()
        .wait()
    ,
    newVar("ID")
        .global()
        .set(getTextInput("input_ID"))
)

// Experimental trial
Template("items.csv", row =>
    newTrial("experimental-trial",
        newTimer("break", 1000)
            .start()
            .wait()
        ,
        newAudio("audio", row.audio)
            .play()
        ,
        newTimer("timeout", row.duration)
            .start()
        ,
        newText("sentence", row.sentence)
            .center()
            .unfold(row.duration)
        ,
        newImage("plural", row.plural_image)
            .size(200, 200)
        ,
        newImage("singular", row.singular_image)
            .size(200, 200)
        ,
        newCanvas("side-by-side", 450,200)
            .add(  0, 0, getImage("plural"))
            .add(250, 0, getImage("singular"))
            .center()
            .print()
            .log()
        ,
        newSelector("selection")
            .add(getImage("plural"), getImage("singular"))
            .shuffle()
            .keys("F", "J")
            .log()
            .callback(getTimer("timeout").stop())
        ,
        getTimer("timeout")
            .wait()
        ,
        getAudio("audio")
            .stop()
    )
    .log("group", row.group)
    .log("item", row.item)
    .log("condition", row.inflection)
    .log("ID", getVar("ID"))
)

// Send results manually
SendResults("send")

// Completion screen
newTrial("completion_screen",
    newText("thanks", "Thank you for participating! You may now exit the window.")
        .center()
        .print()
    ,
    newButton("wait", "")
        .wait()
)
```
{% endcapture %}
{% include collapsible-block.html content=content summary="Click to see the final experiment script" inner-border=true %}
</div>

#### Instructions

Follow the tutorial by completing the tasks in the <span class="label label-purple">instructions</span> blocks:

{% capture instructions %}
Code blocks inside an instruction indicate `main.js`, the [experiment script](#editing-an-experiment).

```javascript
// This is a comment.
```
{% endcapture %}
{% include instructions.html text=instructions%}