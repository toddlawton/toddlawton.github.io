---
layout: post
title: Use NodeJS and Arduino to build a weather display
---

<img class="float-right" src="https://cloud.githubusercontent.com/assets/5748440/4144068/ac68fe0a-33d7-11e4-88a1-762d0fdd8fea.gif"/>

In this short guide I'll explain how I built my own desktop weather display on an [Arduino Uno](http://arduino.cc/en/Main/ArduinoBoardUno) running Node and [Johnny-Five](https://github.com/rwaldron/johnny-five). This could serve as a nice desktop ornament that you can glance at when you wake up in the morning, and the principles in this guide will serve well in future projects that require some hardware interaction with an API. 

This project will use is the [OpenWeatherMap API](http://openweathermap.org/API).

-----

You'll need the following things before starting:

* A 16x2 LCD display, I used the [LCD-WH1602B](http://arduino.cc/documents/datasheets/LCD-WH1602B-TMI.pdf)
* An Arduino board, many will work with this project, but I used an [Arduino Uno](http://arduino.cc/en/Main/ArduinoBoardUno)
* Different coloured jumper wires
* A potentiometer

-----

### So, why Arduino?

Since any microcontroller board that is capable of running Node and has digital I/O pins will run this, I consider this project to be hardware agnostic to a certain extent. I chose the Arduino Uno as my first board because they have a great community, they're easily extendable, and the boards are made in Italy under ethical conditions for a reasonable price. 

I started out by doing a few small projects in C++ in the Arduino IDE so I could understand a little bit better how these boards work before adding layers of abstraction with JS and its hardware libraries.
