---
title: 'Smart Inventory Management System'
author: '**Jen Galicia and Michael Swinton**'
date: '*EEC172 SQ24*'

toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'The Smart Inventory Management System tracks inventory levels in real-time and alerts users when stock levels fall below predefined thresholds or reach maximum capacity. Our product is designed to optimize inventory control for its users by providing instant visibility and insights to ensure efficient stock management.
<br/><br/>
Our source code can be found 
<!-- replace this link -->
<a href="https://github.com/mwswinto/EEC172_Final_Project">
  here</a>.

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/svfLRaA66TI?si=3UUPCqoFZo3ciFb-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
'
---

# Design

## System Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
    As shown in the System Architecture Flowchart, we use a CC3200 board as the main module for connecting the peripherals of our system together. The board reads and decodes the IR transmissions from the IR Receiver as user input and displays that information onto the OLED display for the user to see. The board also reads the values from the weight sensors through a custom protocol. By using RESTful API, the board sends a notification to the AWS cloud when the board detects that the values from the weight sensors hit a certain threshold. This notification is sent to the user's email, which can be viewed on their personal device.
  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 400px;">
    <div class="fig">
      <img src="./media/System-Architecture.jpg" style="width:90%;height:auto;" />
      <span class="caption">System Flowchart</span>
    </div>
  </div>
</div>

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 300px;">
    The state diagram illustrates the functionality of our system. The system begins at the 'Start Screen' where the user is prompted to press 'Enter' whenever they are ready to start using the Smart Inventory Management System. Next, the user is asked to enter an item name, using the IR remote controller. Once a valid item name is entered, the system requires the user to calibrate the weight sensor. This can be done by placing an object of known weight onto the sensor (in our case we used a roll of tape that weighed 66 grams). After the weight sensor is calibrated, the user is asked to place a single item onto the weight scale to track. After placing the weight of the single item, the user is then asked to place the rest of the items onto the scale and the inventory tracking system becomes active. This data is transmitted to the CC3200 board, which monitors the inventory weight. If the weight reaches a specified threshold, the board sends an alert to AWS, which notifies the user via email with inventory details.
  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 500px">
    <div class="fig">
      <img src="./media/FunctionalSpecification.jpg" style="width:90%;height:auto;" />
      <span class="caption">State Diagram</span>
    </div>
  </div>
</div>

# Implementation

### CC3200-LAUNCHXL Evaluation Board

All of the control and logic is handled by the CC3200 Launchpad Board. The board is responsible for transitioning between the states of the system, decoding IR transmissions from the IR remote controller, controlling the OLED display via SPI, using a custom protocol to receive weight data from the weight sensor, and sending inventory status information through AWS.

## Functional Blocks

### IR Receiver

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 200px'>
    The user inputs item information using the IR Remote Controller. The falling edges from the signal triggers interrupts. The program decodes the pulse distances between each trigger to detect what button the user has pressed (0-9, delete, and enter). We integrated our multi-tap texting interface to allow users to input a variety of characters through repeated presses within a time threshold.
  </div>
  <div style='display: inline-block; vertical-align: top;flex:0 0 400px'>
    <div class="fig">
      <img src="./media/IR-Reciever.jpg" style="width:auto;height:2.5in" />
      <span class="caption">IR Receiver Wiring Diagram</span>
    </div>
  </div>
</div>

### OLED Display

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    We interfaced the OLED display via SPI. The user is able to view the item name they are entering, when the system wants them to calibrate the scale, when to place items, when the tracking system is activated, and when the system is sending a notification to their device.
  </div>
  <div style='display: inline-block; vertical-align: top;flex:0 0 400px'>
    <div class="fig">
      <img src="./media/OLED-Display.jpg" style="width:auto;height:2in" />
      <span class="caption">OLED Wiring Diagram</span>
    </div>
  </div>
</div>

### Weight Sensors

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    The weight sensor module consists of two components. The first component is the 20kg load cell. The load cell has 4 leads and a 5-10V drive voltage. It produces a direct output voltage signal due to force changes. To properly use the load cell, one end of the sensor is fixed by the screw hole while the other end is kept in a floating state. Force is applied on the side that is floating.  The second component of the weight sensor module is the HX711 AD Weight Module. This module uses 24 high-precision A/D converter chip HX711, is designed for high precision electronic scale and design. To interface with this weight sensor module, we first had to make the right connections between the 20kg load cell and the HX711 as well as between the HX711 and the CC3200.
    <ul>
      <li>red wire (load cell) -> E+ (HX711)</li>
      <li>black wire (load cell) -> E- (HX711)</li>
      <li>green wire (load cell) -> A+ (HX711)</li>
      <li>white wire (load cell) -> A- (HX711)</li>
      <li>5V (CC3200) -> Vin (HX711)</li>
      <li>gnd (HX711) -> gnd (CC3200)</li>
      <li>output GPIO (CC3200) -> clk(HX711)</li>
      <li>dataOut(HX711) -> input GPIO</li>
    </ul>
    After making the right connections, in order to read data from the sensor we had to write code to follow the procedures of the custom serial protocol detailed in the HX711 data sheet. These procedures included:
    <ul>
      <li>When dataOut is not ready for retrieval, the pin is high</li>
      <li>When dataOut goes low, it indicates data is ready for retrieval</li>
      <li>By applying 25 positive clock pulses at the clk pin, data is shifted out from the dataOut ouput pin</li>
      <li>Each clk pulse shifts out one bit, starting with the LSB first until all 24 bits are shifted out</li>
      <li>The 25th clk pulse pulls dataOut pin back to high</li>
    </ul>
    Once we were able to read the raw sensor data, the data needed to go through processing such as averaging and calibration. After all that, the weight sensor is ready prepared to give accurate weight readings.
  </div>
</div>

### AWS IoT

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    The AWS IoT Core allows the board to send emails to the user when a specified threshold is reached. We utilize RESTful API to store data to the device shadow.
  </div>
</div>


# Challenges

One big challenge that we faced was figuring out how to read the weight sensor data from the HX711. Initially, we though we were going to be able to communicate with the peripheral through i2c. After some more thorough Internet searching we discovered that the HX711 is incompatible with i2c. Our next attempt was to use SPI co interface with the weight sensor. However, after more research we discovered that the typical SPI method that we learned in class was not going to work here either. As an aside, there are some HX711 APIs on github but they are only compatible with Arduino and not the CC3200. We spent some time trying to convert the Arduino code to work with the CC3200 but after a couple failed attempts, we just decided to reference the HX711 data sheet and write our own interface functions from scratch. In the end we figured it out. Another challenge was the construction of the scale. This challenge was mostly just due to poor planning and the time constraint.

# Future Work

Due to time constraints, we were unable to finish integrating AWS into our system. In the future, we plan to complete setting up AWS, as it is a crucial feature of the Smart Inventory Management System that distinguishes it from existing products. Additionally, we aim to enhance the UI on the OLED display, making it more visually appealing to improve usability and efficiency, thereby enhancing overall user engagement. We also plan to develop a web application that displays inventory levels. Additionally. we would also spend more time developing our scale design. For this prototype we utilized cardboard to construct our scale to cut time and resources. With more time and resources it would be ideal to develop a shelving structure scale.


# Finalized BOM

<!-- you can convert google sheet cells to html for free using a converter
  like https://tabletomarkdown.com/convert-spreadsheet-to-html/ -->

<table style="border-collapse:collapse;">
<thead>
  <tr>
    <th><p>No.</p></th>
    <th><p>PART NAME</p></th>
    <th><p>DESCRIPTION</p></th>
    <th><p>Qty</p></th>
    <th><p>SUPPLIER / MANUFACTURER</p></th>
    <th><p>UNIT COST</p></th>
    <th><p>TOTAL PART COST</p></th>
    <th><p>Purpose</p></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><p>1</p></td>
    <td><p>CC3200-LAUNCHXL</p></td>
    <td><p>MCU Evaluation Board</p></td>
    <td><p>1</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$66.00</p></td>
    <td><p>$66.00</p></td>
    <td><p>Control Remote and Local Devices</p></td>
  </tr>
  <tr>
    <td><p>2</p></td>
    <td><p>Adafruit 1431 OLED</p></td>
    <td><p>128x128 RGB OLED Display. SPI protocol</p></td>
    <td><p>1</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$39.95</p></td>
    <td><p>$39.95</p></td>
    <td><p>Display Inputs and Different States</p></td>
  </tr>
  <tr>
    <td><p>3</p></td>
    <td><p>Vishay TSOP31130 IR RCVR</p></td>
    <td><p>30kHz carrier frequency</p></td>
    <td><p>1</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$1.41</p></td>
    <td><p>$1.41</p></td>
    <td><p>Decode user inputs</p></td>
  </tr>
  <tr>
    <td><p>4</p></td>
    <td><p>ATT-RC1534801 Remote</p></td>
    <td><p>General-purpose TV remote. IR NTC protocol</p></td>
    <td><p>1</p></td>
    <td><p>Provided by EEC172 Course</p></td>
    <td><p>$9.99</p></td>
    <td><p>$9.99</p></td>
    <td><p>Allow user inputs</p></td>
  </tr>
  <tr>
    <td><p>5</p></td>
    <td><p>HX711 Module</p></td>
    <td><p>20kg load cell and HX711 Weight Sensor Module</p></td>
    <td><p>1</p></td>
    <td><p>Amazon</p></td>
    <td><p>$12.00</p></td>
    <td><p>$12.00</p></td>
    <td><p>Measure item weight</p></td>
  </tr>
  <tr>
    <td colspan="3">
      <p>TOTAL PARTS</p></td>
    <td><p>5</p></td>
    <td colspan="2">
      <p>TOTAL</p></td>
    <td><p>$129.34</p></td>
    <td></td>
  </tr>
  <tr>
    <td colspan="3">
      <p>TOTAL PARTS (Excluding Provided)</p></td>
    <td><p>1</p></td>
    <td colspan="2">
      <p>TOTAL (Exluding Provided)</p></td>
    <td><p>$12.00</p></td>
    <td></td>
  </tr>
</tbody>
</table>