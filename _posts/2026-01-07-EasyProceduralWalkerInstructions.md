---
layout: post
title: UE5 plugin
date: 2023-01-07 19:14:00 +8000
categories: [Coding]
tags: [UE5]
comments: true
math: true
image:
    path: ../assets/img/photos/EasyProceduralWalker_1.png
---

## 1. Introduction

EasyProceduralWalker is a simple UE plugin for multi-legged creature locomotion. Built on TwoBoneIK, it can drive only two bones (three joints) per leg.  
Before using the plugin, you need to know how to set up a TwoBoneIK node for a single leg.

## 2. Usage
Create an “Easy Procedural Walker” node in your AnimGraph, connect its input to the “Mesh Space Ref Pose” node, and connect its output to the “Output Pose” node.
![blueprint](../assets/img/photos/EasyProceduralWalker_2.png)

In the **Details** panel there are three modes:  
- **SetBody**: configure the two-bone-IK parameters for each leg  
- **AnimBP Preview**: use root motion to preview the results  
- **Game Play**: play in the level

![blueprint](../assets/img/photos/EasyProceduralWalker_3.png)

### 2.1 SetBody

**Step 1: In the `EPW Mode [Critical & Important]` tab**  
- Set **Debug Mode** to *SetBody*  
- Set **Current Leg Index** to *0* which means show the first leg's TwoBoneIK debug UI.

![SetBody](../assets/img/gifs/1_setbody.gif)

**Step2: In `EPW Settings` tab**
- Set **Root Bone** and **Body Bone**

![SetBody](../assets/img/gifs/2_1_setbody.gif)

- To add each leg, first adjust the **Current Leg Index**​ to the corresponding number. Then, set its `IKBone`, `Effector Location`, and `Joint Target Location`.
  - **IKBone**: the end bone of the leg (usually the foot)  
  - **Effector Location**: the world-space position of the foot joint  
  - **Joint Target Location**: the world-space point the middle joint aims at

> Open your Skeletal Mesh asset, copy the global position, and paste it into the corresponding joint location.
{: .prompt-tip }

![blueprint](../assets/img/photos/EasyProceduralWalker_5.png)
The final result for the left leg should look like this.
![blueprint](../assets/img/photos/EasyProceduralWalker_6.png)

** Step3. In `EPW Step Settings` tab **
- If the foot has a heel or shoe, enable **Set Default Heel Height** and enter the heel height for each leg in **Default Heel Height**.  
- To make several legs move in the same time slot, enable **Set Timings Manually** and assign a float to each leg.  
  - Example: the value 0.3 puts the leg in group 0 and makes it start moving at 30 % of that group’s cycle (one complete step).  
- **Timings Offset**: by default, only one leg is defined per group.  

> The next group starts only after every leg in the previous group has finished. The integer part of the value sets the group; the fractional part sets when, within that group’s cycle, the leg begins to move.
{: .prompt-info }

**Step 4. In `EPW Trace`**  
- **Show Predict Pos**: display the next target position for each foot and the predicted joint-target location.  
- **Show Line Ray**: toggle line-trace debugging.  
- **Show Sphere Hitter**: toggle sphere-trace debugging.

![SetBody](../assets/img/gifs/2_3_setbody.gif)

### 2.2 AnimBP Preview

Create a test animation asset that moves only the root bone.
![SetBody](../assets/img/gifs/3_1_ABPMove.gif)

Switch to **AnimBP Preview** mode and plug the animation into the **Easy Procedural Walker** node in the AnimGraph.
![SetBody](../assets/img/gifs/3_2_ABPMove.gif)

> In this step, create animations at different speeds and adjust **Step Length** and **Step Height** to suitable values.
{: .prompt-tip }

### 2.3 Game Play

If you move the avatar in the game scene, reconnect the node to **Mesh Space Ref Pose** (or **Input Pose**) and set **Debug Mode** to **Game Play**.
![blueprint](../assets/img/photos/EasyProceduralWalker_8.png)

## 3. Other parameters

1. In the `EPW Settings` tab  
- `Move breath`: The body moves up and down in sync with each step.  
- `Stand breath`: The body gently rises and falls while standing still.  
- `Body Fit slope`: The body automatically tilts to match the slope angle as it moves.

> Robots with two legs generally don’t need to enable `Body Fit slope`; the parameter is more suitable for multi-legged insects.
{: .prompt-tip }

![blueprint](../assets/img/photos/EasyProceduralWalker_9.png)

