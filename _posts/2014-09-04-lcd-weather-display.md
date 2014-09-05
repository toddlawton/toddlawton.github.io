---
layout: post
title: Use NodeJS and Arduino to build a weather display
---

<img class="float-right" alt="The finished LCD display in action" src="https://cloud.githubusercontent.com/assets/5748440/4144068/ac68fe0a-33d7-11e4-88a1-762d0fdd8fea.gif"/>

In this tutorial I'll explain how to build your own LCD weather display on an [Arduino Uno](http://arduino.cc/en/Main/ArduinoBoardUno) running Node and [Johnny-Five](https://github.com/rwaldron/johnny-five). This could serve as a nice desktop ornament that you can glance at when you wake up in the morning, and the principles in this guide will serve well in future projects that require hardware interaction with an API. 

This project will use is the pretty awesome [OpenWeatherMap API](http://openweathermap.org/API).

-----

### Gather up:

* An Arduino board, many will work with this project, but I used an [Arduino Uno](http://arduino.cc/en/Main/ArduinoBoardUno)
* A 16x2 LCD display, I used the [LCD-WH1602B](http://arduino.cc/documents/datasheets/LCD-WH1602B-TMI.pdf)
* A bunch of jumper wires
* 1 potentiometer
* 1 220-ohm resistor

-----

### So, why Arduino?

Since any microcontroller board that is capable of running Node and has digital I/O pins will run this, I consider this project to be hardware agnostic to a certain extent. I chose Arduino foremost because they have a great community, they're easily extendable, and the boards are made in Italy under ethical conditions for a reasonable price. 

<img class="float-right" alt="Arduino Uno Version 3" src="http://arduino.cc/en/uploads/Main/ArduinoUno_R3_Front_450px.jpg" />

I started out by doing a few small projects in C++ in the Arduino IDE so I could understand a little bit better how these boards work before adding layers of abstraction with JS and its hardware libraries.

Once I got a taste of Arduino's capabilities on Node, it was hard to look back. There are however by default some drawbacks to Node. When you upload a C++ sketch to Arduino it is able to boot from that sketch each time it's powered on. A Node application will need to be fired externally on boot, but there are ways around that which I aim to explore in future articles.

-----

### First step: Wire it up

<img src="https://cloud.githubusercontent.com/assets/5748440/4158768/c1eaa2fe-3492-11e4-9c7c-1767bffd2e79.jpg" alt="Arduino Wiring Diagram" class="float-right">

In this guide I'm assuming you've already made a few small things with an Arduino such as flashing an LED, using a light sensor, etc. I won't cover any of the wiring basics, however I've included a wiring diagram to make things faster to get started. 

If you'd like more information on setting up an LCD, you can visit Arduino's tutorial [here](http://arduino.cc/en/Tutorial/LiquidCrystal)

In the JS file for the project I've included a comment indicating which pins on the LCD need to be connected to which pins of your Arduino. These can be changed in the code if you'd prefer a different pin configuration.


```
// LCD pin name  RS  EN  DB4 DB5 DB6 DB7
// Arduino pin # 12  11   5   4  3  2
```

 -----

### Second step: Get your Arduino ready for Node

An Arduino will not be able to run a Node application until you sprinkle on some magic fairy dust called Firmata.

1. Connect your Arduino to your computer.
2. Open up the Arduino IDE.
3. Open the sketch under: File -> Examples -> Firmata -> StandardFirmata
4. Hit Upload

-----

### Third step: Set up your project folder

1. In your terminal mkdir a folder named *lcd-weather-display* and cd into it.
2. Run ```npm install request``` and ```npm install johnny-five```.
3. Make a new file called *lcd-weather-display.js* and open it in your favorite code editor.

-----

### Fourth step: Let's code!

#### Setup

Add variables and dependencies.

```
	// For interfacing with Arduino
var five = require("johnny-five"),

	// For making simple HTTP requests in Node
    request = require("request"), 
    
	// Initialize a new Johnny-Five board
    board = new five.Board(), 

	// The # of rows on the LCD screen
    numberRows = 2,

    // The # of columns on the LCD screen
    numberCols = 16,

    // The city name to query
    currentCity = "Montreal",

    // The country code for above city
    countryCode = "CA",

	// "C" for Celsius, "F" for Fahrenheit
    temperatureFormat = "C";
```

Add a ready function for your Arduino, and initialize the lcd.

```

// Johnny-Five comes with jQuery-esque
// ready functions

board.on("ready", function() {

	// Initialize a new LCD through 
	// Johnny-Five's built in LCD library

	var lcd = new five.LCD({

		// LCD pin name  RS  EN  DB4 DB5 DB6 DB7
		// Arduino pin # 12  11   5   4  3  2
		pins: [ 12, 11, 5, 4, 3, 2 ],
		rows: numberRows,
		cols: numberCols

	});


	// Once the lcd is fired up...

	lcd.on("ready", function() {
		// Do awesome lcd things
	});

});
```

#### LCD functions

Once your lcd is ready you can control it really easily using functions like:
<br>
``` lcd.clear(); // Remove everything from the screen ``` <br>
``` lcd.setCursor(0,0); // Set the column, row to print from ``` <br>
``` lcd.print("Fetching Weather"); // Print accepts normal strings ``` <br>

You can also make your own characters on screen and add them to the library which is pretty darn useful. I did this to make a better looking degree symbol.

If you'd like to practice making your own, you can build one visually with [this tool](http://www.quinapalus.com/hd44780udg.html). When you're finished creating, you can copy the decimal output and add it as an array to the `createChar` function.

It would look like this: <br>
```lcd.createChar("characterName", [4,10,4,0,0,0,0]);```<br>
```lcd.useChar("characterName");```<br>
```lcd.print(":characterName:");```<br>
**Important:** Don't forget to add a colon before and after when doing a `.print` of your custom character.

When the lcd is ready you can clear the screen, set the cursor at (0,0) and print a "Fetching Weather" message before sending off the request to the API. Chances are this will flash really quickly, but visual feedback like this are great when on a slow connection.

#### API request

Once the lcd is ready, we'll send a GET request to OpenWeatherMap.

```
lcd.on("ready", function() {

	// Add visual feedback that the LCD
	// is ready and about to fetch data.

	lcd.clear().setCursor(0,0).print("Fetching Weather");


	// Fire a request to the API, 
	// passing in your variables currentCity
	// and countryCode <-must be lowercase

	request("http://api.openweathermap.org/data/2.5/weather?q="+currentCity+","+countryCode.toLowerCase()+"", function(error, response, body) {
		
		// The contents of this function
		// will fire on a successful GET.

	});
});
```

Upon inspection of the JSON response that we receive, we'll notice that the temperatures are all in Kelvins (oh noes!). Since most people are accustomed to reading the temperature in Celsius or Fahrenheit, let's include utility functions to take care of the conversion. We can place these outside of the board's ready function.

```
function kelvinToCelsius(input) {
	input -= 273.15;
	return parseInt(input);
}

function kelvinToFahrenheit(input) {
	input = (input - 273.15) * 1.8000 + 32.00;
	return parseInt(input);
}
```

The weather data in the response can now be stored in variables to be sent to the LCD screen.
When you've got all of the data you need, call a function to execute the display loop. As a classic StarCraft fan, I called my function fireItUp().

We'll pass any Kelvin measurements through our utility functions depending on the temperature format variable set at the top, or throw an error if the format is invalid.

```
lcd.on("ready", function() {
	lcd.clear().setCursor(0,0).print("Fetching Weather");
	request("http://api.openweathermap.org/data/2.5/weather?q="+currentCity+","+countryCode.toLowerCase()+"", function(error, response, body) {

		// Send a message back to the
		// terminal if great success (yakshemash).

		console.log("Request Successful");

		weatherData = JSON.parse(body);
		
		city = weatherData.name;
		
		if (temperatureFormat === "C") {
			
			// Store the current temperature, 
			// High and Low for the day.

			temperature = kelvinToCelsius(weatherData.main.temp);
			temperatureLow = kelvinToCelsius(weatherData.main.temp_min);
			temperatureHigh = kelvinToCelsius(weatherData.main.temp_max);

		} else if (temperatureFormat === "F") {
			
			temperature = kelvinToFahrenheit(weatherData.main.temp);
			temperatureLow = kelvinToFahrenheit(weatherData.main.temp_min);
			temperatureHigh = kelvinToFahrenheit(weatherData.main.temp_max);

		} else {

			throw "Invalid Temperature Format!";

		}

		// This is a literal description
		// such as mist, partly cloudy, etc.

		description = weatherData.weather[0].description;

		fireItUp(); // Send weather data to the screen
	});
});
```

Almost done. Whew! This is the part of the application that really handles what we're seeing on the LCD. 

As you'll notice in the code below, we start off by creating our custom degree symbol mentioned earlier, and printing it to the LCD with the current temperature, as well as the current city on row 1.

Then we start an interval on our Arduino which controls the second line, that cycles through the weather description, high and low for the day, every 2000 milliseconds.

The second line message is decided by a very simple switch case statement with a counter that resets after the numberMessages is hit.

```
function fireItUp() {

		// Here we see our custom degree
		// symbol come to life.

		// Print the city name and current temp
		// on row 1 of the LCD.

		var degCharMap = [4,10,4,0,0,0,0]; // Make a custom degree symbol
		lcd.createChar("deg", degCharMap); // Assign it as a usuable char in the lcd
		lcd.useChar("deg"); // Use dat sheet
		lcd.clear().setCursor(0,0).print(city); // Write city name
		lcd.setCursor(numberCols-4, 0).print(temperature).setCursor(numberCols-2, 0).print(":deg:").setCursor(numberCols-1, 0).print(temperatureFormat); // Write current temperature

		// The second row will cycle through the 
		// description, high and low temperatures.

		var secondLine = 0, // 0 = Description, 1 = High Temp, 2 = Low Temp
			numberMessages = 3;

		// This loop function is rather self explanatory,
		// it's basically just a setInterval that works
		// on your Arduino.	

		board.loop( 2000, function() {


			// It's a good practice in this case to clear
			// the entire line so that no leftover
			// charactrs are present on the next draw.

			lcd.setCursor(0, numberRows-1).print("                "); // Clear the second line

			switch(secondLine) {
				case 0:
					lcd.setCursor(0, numberRows-1).print(description); // Describe current weather
					break;
				case 1:
					lcd.setCursor(0, numberRows-1).print("High of "+temperatureHigh+":deg:"+temperatureFormat); // Display high temperature
					break;
				case 2:
					lcd.setCursor(0, numberRows-1).print("Low of "+temperatureLow+":deg:"+temperatureFormat); // Display low temperature
					break;
			}

			secondLine++;

			if (secondLine > numberMessages-1) {
				secondLine = 0;
			}
		});
}
```

I hope this tutorial has been helpful, and that you push this even further! 

You can find the code for this little project on GitHub [right here](https://github.com/toddlawton/arduino-nodebots/tree/master/node/lcd-weather-display). 

If you'd like to follow me on twitter, I'm [@toddlawton](http://twitter.com/toddlawton)