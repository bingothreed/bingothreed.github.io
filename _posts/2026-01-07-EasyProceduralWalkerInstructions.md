---
layout: post
title: UE5 plugin
date: 2023-01-07 19:14:00 +8000
categories: [Coding]
tags: [UE5]
comments: true
image:
    path: ../assets/img/photos/EasyProceduralWalker_1.png
    alt: caption
---

# Introduction

EasyProceduralWalker is a simple UE plugin for multi-legged creature locomotion. Built on TwoBoneIK, it is limited to driving only two bones (three joints) per leg. 
Before you use the plugin, you must know how to set a TwoBoneIK node for one leg

# Usage
Create an “Easy Procedural Walker” node in your AnimGraph, and connect to “Mesh Space Ref Pose” node as input and “Output Pose” as output 
![blueprint](../assets/img/photos/EasyProceduralWalker_2.png)