---
layout: single
title: "Garage Door Automation"
categories: diy
tags: automation esp8266
header:
  overlay_image: /assets/images/2019-03-10/garage-door-controller.jpg
  teaser: /assets/images/2019-03-10/garage-door-controller.jpg
excerpt: My first home automation/HomeAssistant project to ensure the garage door isn't left open all night.
toc: true
---
## Background

At our house, my wife parks the car in the garage and I park in the driveway. Using the front door is inconvenient most of the time, so we primarily enter/exit the house through the garage. Depending on who is coming and going when, and who is working or playing outside, we often leave the garage door open much of the day on weekends and during the summer.

Unfortunately, because no one takes ultimate responsibility for closing the garage door at the end of the day, it was quite common to wake up in the morning to find the garage door had been left open all night.

This was my first 'home automation' project, which involved a lot of research and learning about ESP8266 boards, MQTT, installing and playing with Home Assistant, etc. So though it was a pretty simple project, I learned a lot from doing it.

## Goal

Design a solution such that the garage door will close automatically if left open at night.

Other considerations:

* Account for the fact that sometimes we leave the garage door open later at night on purpose (late night oil change or other tasks).
* Turn on and off the lights in the garage automatically based on usage/human presence.
* Remotely monitor and control the state of the garage door, for those occasions when everyone leaves the house but the door was left open accidentally.

## Solution

In a nutshell, after much research, I followed the plans for <a href="https://github.com/marthoc/GarHAge" target="_new">GarHAge</a>, so that all of this could be integrated into <a href="https://www.home-assistant.io/" target="_new">HomeAssistant</a>. I didn't have HA set up yet at the time, so this project required doing that too. HA has since become my central hub for nearly everything.

I installed a <a href="https://www.amazon.com/Magnetic-Switch-Normally-Security-Contact/dp/B086GYJLML>" target="_new">magnetic reed switch like this</a> that would serve to monitor the open/close state of the garage door. You can see where I attached it near the top of the garage door.

<span style="align: center">![Reed Switch Installed](/assets/images/2019-03-10/reed-switch-installed.jpg)</span>

I installed an <a href="https://www.amazon.com/gp/product/B07LCMNXTN" target="_new">ESP8266 NodeMCU board</a> and a relay in a small plastic enclosure. 

<span style="align: center">![Controller](/assets/images/2019-03-10/garage-door-controller.jpg)</span>

Then I mounted the enclosure on top of the garage door opener. It's powered with a simple 5v USB wall wart plugged into the same outlet as the garage door opener. (Note: the picture shows it held on with painter's tape. That was just temporary until I got some velcro. It needed to be secure, as the opener vibrates a lot when operating.)

<span style="align: center">![Controller Installed](/assets/images/2019-03-10/controller-installed.jpg)</span>

Finally, I wired the NodeMCU to the reed switch by running some low voltage speaker wire along the center of the ceiling. Then, I connected a jumper from the NodeMCU to the same connection point on the garage door opener where the wired, wall-mounted remote is connected. 

So, the NodeMCU actually serves two purposes. 1. It tracks the state of the reed switch and reports to Home Assistant whether the garage door is open or closed. 2. It receives commands from Home Assistant and sends a pulse to the garage door opener which opens or closes the door, just like a wired, wall-mounted controller would. 

Going beyond the GarHAge setup:
* I installed a motion sensor in the garage to detect human presence. (I started with my own DIY PIR sensor connected to a NodeMCU board, but later evolved to using a <a href="https://www.itead.cc/sonoff-rf-bridge-433.html" target="_new">Sonoff PIR2</a> motion sensor. It's MUCH more reliable.)
* To add related controls for lights, I'm using a TP-Link HS200 smart switch. I'm not linking to it because I hate it. I have 2 of these and a few other TP-Link smart devices, and they constantly lose WiFi connectivity.  
* Later, as part of my [Home Security system - post coming soon] project, I tapped into the existing open/close door sensor on the door leading from the garage to the family room to track and trigger based on the state of that door.
* Later, as part of my [Google Assistant - post coming soon] integration project, I added voice control.

## How it works

A series of HA automations does/achieves the following:
* When the family room-to-garage door opens, turn on the garage lights.
* When motion is detected in the garage, turn on the garage lights (or at least don't turn them off).
* Beginning at 9pm, if no motion detected in the garage for the previous 10 minutes, assume the garage door was left open and close it automatically, then turn off the lights.
* When no motion is detected in the garage for 10 minutes, the lights are on, but garage doors are closed, assume it's no longer occupied and turn off the lights.
* At all times, when the garage door has been opened and left open for 10 minutes, with no motion detected, make a one-time voice notification via Google Assistant informing us that the garage door is open and encouraging us to close it if it was not intentional.

And finally, because sometimes I ride my bike on the trainer or someone is doing something in the garage that the motion sensor doesn't pick up, I added a 'motion sensor override' *input boolean* sensor. This allows us to turn off the automation that turns off the lights so they'll stay on if we need.