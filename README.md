# 12 or 24+ Channel Dimming Controller for Meanwell LED Power Supplies (works with many other manufacturers aswell)

## Disclaimer
**By using the information you agree to the following disclaimer:** The author of this information assumes no liability for the accuracy of the info, or  damage or injury caused by using these instructions or any product created using methods or instructions herein. Building and using this controller requires technical knowledge and experience with electrical circuits and components. If you are unsure about any aspect of this project it is strongly recommended that you seek assistance from a knowledgeable or licensed individual or consult relevant resources before proceeding.  Always follow <a href="https://www.google.com/search?q=hobby+electronics+safety">electronics safety guidelines</a>.

## Table of Contents

## Intro
This guide provides information to build a <a href="https://www.home-assistant.io/">Home Assistant</a> compatible light dimmer that will work with most <a href="https://led.meanwell.com/productSeries.aspx">Meanwell LED drivers (aka LED power supplies or ballasts)</a> using just a few electronic parts.  This should work with other manufacturer LED drivers which use PWM dimming, so long as the driver provides source current on the dim circuit.  I think this can also be modified to work with 0-10v DC additive dimming by connecting a 10v power source to the signal board, but I haven't tried it.  Feel free to reach out to me if you have any questions about that.

## Background
Most LED grow and aquarium lights offer some form of dimming, either with dimming knobs, controllers, or the option to connect a dimming knob or controller (usually promoted by the manufacturer/reseller of the light).  Just like many other products, having your own easy-to-build and cost-effective option that works with Home Assistant offers a huge improvement in functionality, as you can for example create lighting profiles for different stages of growth or times of day, and build automatic rules to adjust your lights depending on things such as room/water temperatures, reducing light output if a person is detected in the room and more.  With Home Assistant the limit to how you automate your control is mostly limited to your imagination.

Over the years I've searched for lighting controllers, only to find expensive products that are limited in their functionality. The options that are available for use with Home Assistant or other SmartHome systems usually lack the ability to control a large number channels or drivers, so the cost can quickly rise depending on how complex the lighting system is.  This controller being able to control up to 24 or more _channels_ means you can have a large number of LED drivers being controlled by a single, cheap solution.  

<p align="right"><a href="#readme">...back to top</a></p>

## Applications & Use Cases
The most likely use for this controller are indoor gardens or aquarium (reef) tanks that use LED power supplies, although it can be used for decor lighting applications.  The advancement of LED lighting has led to many of these types of systems being put into use.  From do-it-yourself hobbyist lighting to the relative market explosion of off-the-shelf options, this controller is a nice upgrade to many of these systems already in use.

A further use case would be a LED fixture that has multiple channels or colors that can be individually controlled.  Controlling different colors of LEDs is much more common in aquarium applications, although there is good benefit from being able to control different colored LED channels with grow lighting. (Examples include Warm White, Cool White, 660nm Red, Far Red, Blue, UV).  Some specific examples:

### Example use case 1: Tuneable 3000k-5500k grow light for all growth stages: 
Cost effective tuneable grow light based off white leds only. Using only two channels this configuration allows you can adjust the overall temperature of the grow light from 3000k warm (late stage growth) to 5500k cool (early stage growth) simply by changing the output of each channel.

- Channel 1: Warm White 3000K
- Channel 2: Cool White 5500K

### Example use case 2: 8-Channel aquarium light
Control over multiple channels to optimize the aesthetics of a reef tank, or adjust the spectrum mix for optimal aquatic life.

- Channel 1: Cool White
- Channel 2: Royal Blue
- Channel 3: Blue
- Channel 4: Red
- Channel 5: Green
- Channel 6: UV
- Channel 7: Violet
- Channel 8: Warm White  

<p align="right"><a href="#readme">...back to top</a></p>

## Concept
LED drivers (aka power supplies) with dimming wires are designed to be connected to a dimming controller. In the case of Meanwell LED drivers with 3-in-1 dimming, this could be either a <a href="https://www.google.com/search?q=0-10v+Additive+DC+voltage+dimmer">0-10v Additive DC voltage dimmer</a>, <a href="https://www.google.com/search?q=10v+PWM+dimmer">10v PWM dimmer</a> or just a basic <a href="https://www.google.com/search?q=100+ohm+potentiometer">100ohm mechanical potentiometer.</a>  This project makes use of PWM dimming.

A MeanWell power supply that supports external dimming will have two wires (or two terminal points) where a positive (DIM +) and negative (DIM -) wire can be connected to a dimming controller.  In the case of MeanWell PWM dimming (which is the focus of this project), the LED driver creates a small current which runs through the wires. When the wires are disconected from eachother, no current flows and the light output will be 100% brightness.  When the wires are connected, the current flows through and signals the driver to dim the light output to 0% brightness.  PWM works off the principle of opening and closing this circuit very fast (over 1000 times a second), creating the effect of either more or less current to be allowed to pass through the power supply's DIM circuit.  Although you dont need to know how this works in detail to build this project, you can <a href="https://www.google.com/search?q=how+does+pwm+work">read more about how PWM works here.</a>

This project uses an Adafruit PWM LED Driver board to control the dim signals we sent to a MeanWell LED driver.  The original use for the Adafruit board is to drive LEDs directly, but the LEDs we are utilizing require much higher powered drivers (the MeanWell LED Drivers), hence we can use the Adafruit board to send the DIM signals to the higher powered driver.  It is a little confusing that we are using a low-powered LED Driver board to send DIM signals to a higher powered one, it was only coincidence that the Adafruit board is also an LED driver, you can ignore that.

<img src="/images/Home-Assistant-MeanWell-LED-Driver-Dimming-Controller-Schematic.png">  
<img src="/images/Home-Assistant-MeanWell-LED-Driver-Dimming-Controller-Schematic-Adafruit-PWM-Driver-Closeup.png">
<img src="/images/Home-Assistant-MeanWell-LED-Driver-Dimming-Controller-completed.png">  

<p align="right"><a href="#readme">...back to top</a></p>

## Pre-requisites

#### Infrastructure
- <a href="https://www.google.com/search?q=what+are+the+different+ways+to+install+home+assistant%3F">Home Assistant</a> running on a NAS, Raspberry Pi, Computer/Server etc with the <a href="https://www.google.com/search?q=how+to+install+esphome+on+Home+Assistant">ESPHome Add-on Installed</a>
- Wi-fi network for the ESP device to connect to.

#### Skills needed
- <a href="https://www.google.com/search?q=working+with+electronic+hobby+tools">Working with hobby electronic tools</a>.  
- <a href="https://www.google.com/search?q=how+to+solder+pin+headers+into+a+breakout+board" target="_blank">Soldering pin headers into a breakout board</a>.
- <a href="https://www.google.com/search?q=how+to+wire+microcontrollers+to+breakout+boards">Wiring ESP32/ESP8266 MCUs to breakout boards</a>.  

#### Tools Required
_Although having top-of-the-line tools makes the job easier, I rely on economical options for my toolkit. While working with budget tools may not always be optimal, they are adequate for small and occasional projects and have served me well._  
- [Soldering iron](https://www.amazon.ca/s?k=Soldering+iron) ([brass wool](https://www.amazon.ca/s?k=brass+wool+soldering) recommended instead of using a wet solder sponge).  
- [Wire stripper](https://www.amazon.ca/s?k=wire+stripper)  
- [Multimeter](https://www.amazon.ca/s?k=Multimeter&rh=n%3A3006902011%2Cp_36%3A12035760011&dc&ds=v1%3AVJ07OZbfZbPTVCeOY0%2FOTVw2%2FDdTF6NoXoZ9MvBP20c&qid=1675112740&rnid=12035759011&ref=sr_nr_p_36_1) (for testing connections) - _if you plan on using this a lot, it would be better to get a somewhat decent one. Cheap multimeters will have weak connectors that can break easily, or the mode selector dial might not hold up over a long time. I'm happy with the cheapest option available on Amazon, I've only had to replace mine once. I'm just careful with it._  

##### Optional stuff:
- <a href="https://www.amazon.ca/s?k=breadboard">Breadboards</a> - _I used two small breadboards for this project.  They came in a 3-pack, and you can take off the +/- power rails and then snap the main pieces together.  They came with a sticky foam on the back with a peel off sticker.  Using this I didn't need to mount or glue anything into the project box, and it lets me be able to change parts up in the future if I ever wanted to rebuild it._
- [Hot glue gun](https://www.amazon.ca/s?k=Hot+glue+gun) - _I've seen videos of people hot glueing circuit boards directly into project boxes.  I'm not a huge fan of that for most of the stuff I build, but for small projects that you don't think you'll rebuild later that you want to do quick and easy, this can shave off cost and time._

#### Parts needed
- <img src="/images/Meanwell-HLG-480H-LED-Driver.png" align="right" height="150px">One or <a href="https://led.meanwell.com/productSeries.aspx">MeanWell LED Drivers</a> (or LED drivers from another manufacturer compatible with PWM dimming) that you want to be able to control.  _Some of the smaller MeanWell LED drivers do not support PWM dimming. The best place to confirm that is the datasheet for your driver.  You can find it by searching "MODEL_NUMBER pdf" (example "HLG-320H-C1400 pdf"). If you have a driver made by a manufacturer and you can't find any information on the datasheet regarding PWM dimming, contact the manufacturer directly to clarify_.
<BR CLEAR=”left” />  
- <img src="/images/esp8266%20wemos%20d1%20mini.jpg" height="150px" align="right"><a href="https://www.google.com/search?q=ESP32+development+boards">ESP32</a> or <a href="https://www.google.com/search?q=ESP8266+development+boards">ESP8266</a> development board with <a href="https://www.google.com/search?q=spi+pins">SPI pins</a>   
_This is the brain of our controller setup. It has built-in Wi-Fi and can easily connect to your Home Assistant server using the ESPHome plugin. And the best part? You can use a cheap option like the <a href="https://www.google.com/search?q=wemos+mini+d1">Wemos D1 Mini ESP8266</a> we will be using in this project._  
<BR CLEAR=”left” />

- [Adafruit TLC5947 **24-Channel** 12-bit PWM LED Driver](https://www.adafruit.com/product/1429)  
or [Adafruit TLC59711 **12-Channel** 16-bit PWM LED Driver](https://www.adafruit.com/product/3995)   
These boards are capable of generating some serious PWM signals, all the way up to 30V! The PWM voltage it generates is determined by the input voltage, which is why we're using a 10V power supply.  
    <img src="/images/Adafruit%2024%20channel%20PWM%20LED%20driver.jpg" height="150"> <img src="/images/Adafruit%2012%20channel%20PWM%20LED%20driver.jpg" height="150">  

- 2x [Break-Away 0.1" 2x20-pin Strip Dual Male Headers](https://www.google.com/search?q=Break-Away+0.1%22+2x20-pin+Strip+Dual+Male+Header) or 8 3x2 pin headers  
You will solder these terminal pin headers onto the PWM board. This is where the positive (+) and negative (-) DIM wire leads from your LED Driver will connect.  
    <img src="/images/Break-Away%200.1-inch%202x20-pin%20Strip%20Dual%20Male%20Header.jpg" height="150">  

- _terminal blocks_
- _project box_  

<p align="right"><a href="#readme">...back to top</a></p>

# To be added:
## Assembly Instructions
## Software Configuration
## Usage
