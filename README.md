[Garage Door Controller v1.1](https://github.com/andrewshilliday/garage-door-controller)
======================

Monitor and control your garage doors from the web via a Raspberry Pi.

![Screenshot from the original][1] &nbsp; ![Screenshot with a picture][2]

Overview:
---------

This project provides software and hardware installation instructions for monitoring and controlling your garage doors remotely (via the web or a smart phone). The software is designed to run on a [Raspberry Pi](www.raspberrypi.org), which is an inexpensive ARM-based computer, and supports:
* Monitoring of the state of the garage doors (indicating whether they are open, closed, opening, or closing)
* Remote control of the garage doors
* Timestamp of last state change for each door
* Logging of all garage door activity

Requirements:
-----

**Hardware**

* [Raspberry Pi](http://www.raspberrypi.org)
* Micro USB charger (1.5A preferable)
* [USB WiFi dongle](http://amzn.com/B003MTTJOY) (If connecting wirelessly)
* 8 GB micro SD Card
* Relay Module, 1 channel per garage door (I used [SainSmart](http://amzn.com/B0057OC6D8 ), but there are [other options](http://amzn.com/B00DIMGFHY) as well)
* [Magnetic Contact Switch](http://amzn.com/B006VK6YLC) (one, or two per garage door)
    or
* [Garage Door Limit Switch](http://amzn.com//B004CAN18W) (one, or two per garage door)
* [Female-to-Female jumper wires](http://amzn.com/B007XPSVMY) (you'll need around 10, or you can just solder)
* 2-conductor electrical wire

**Software**

* [Raspbian](http://www.raspbian.org/)
* Python >2.7 (installed with Raspbian)
* Raspberry Pi GPIO Python libs (installed with Raspbian)
* Python Twisted web module

Hardware Setup:
------

*Step 1: Install the magnetic contact switches:*

You must install one contact switch to indicate that the door is closed.  A second switch to show that the door is open is optional.
The contact switches are the sensors that the raspberry pi will use to recognize whether the garage doors are open or shut.  You need to install one (or two) on each door so that the switch is *closed* when the garage doors are closed (or open).  
For magnetic switches attach the end without wire hookups to the door itself, and the other end (the one that wires get attached to) to the frame of the door in such a way that they are next to each other when the garage door is shut.  There can be some space between them, but they should be close and aligned properly, like this:

![Sample closed contact switch][3]

For door opener limit switches connect one wire to the single contact and the other to a convenient fastener on the actuator rail.

![Closed switch door closed][7] ![Open switch door closed][6] 

*Step 2: Install the relays:*

The relays are used to mimic a push button being pressed which will in turn cause your garage doors to open and shut.  Each relay channel is wired to the garage door opener identically to and in parallel with the existing push button wiring.  You'll want to consult your models manual, or experiment with paper clips, but it should be wired somewhere around here:

![!\[Wiring the garage door opener\]][4]
    
*Step 3: Wiring it all together*

The following diagram illustrates how to wire up a two-door controller.  The program can accommodate fewer or additional garage doors (available GPIO pins permitting).

![enter image description here][5]

Note: User [@lamping7](https://github.com/lamping7) has kindly informed me that my wiring schematic is not good.  He warns that the relay should not be powered directly off of the Raspberry Pi.  See his explanation and proposed solution [here](https://github.com/andrewshilliday/garage-door-controller/issues/16).  That being said, I've been running my Raspberry Pi according to the above schematic for years now and I haven't yet fried anything or set fire to my house.  Your mileage may vary.

Software Installation:
-----

1. **Install [Raspbian](http://www.raspbian.org/) onto your Raspberry Pi**
    1. [Tutorial](http://www.raspberrypi.org/wp-content/uploads/2012/12/quick-start-guide-v1.1.pdf)
    2. [Another tutorial](http://www.andrewmunsell.com/blog/getting-started-raspberry-pi-install-raspbian)
    3.  [And a video](http://www.youtube.com/watch?v=aTQjuDfEGWc)!

2. **Configure your WiFi adapter** (if necessary).
    
    - [Follow this tutorial](http://www.frodebang.com/post/how-to-install-the-edimax-ew-7811un-wifi-adapter-on-the-raspberry-pi)
    - [or this one](http://www.youtube.com/watch?v=oGbDawnqbP4)

    *From here, you'll need to be logged into your RPi (e.g., via SSH).*

3. **Install the python twisted module** (used to stand up the web server):

    `sudo apt-get install python-twisted`
    
4. **Install the controller application**
        
    I just install it to ~/pi/garage-door-controller.  You can install it anywhere you want but make sure to adapt these instructions accordingly. You can obtain the code via SVN by executing the following:
    
    `sudo apt-get install subversion`

    `svn co https://github.com/andrewshilliday/garage-door-controller/trunk ~pi/garage-door-controller`
    
    That's it; you don't need to build anything.
    
5.  **Create SSL Certificates** (if desired).
    
    If you plan on using SSL by setting the **use_https** option to *true* in the `config.json` file, you will need to complete this step or provide your own private keys and certificate for secure communication.
    
    In order for secure communication to be allowed, the controller application needs SSL certificates.  To quickly generate SSL certificates, do the following:
    
    `mkdir -p /home/pi/garage-door-controller-cert`
    
    `openssl req -new -x509 -days 3650 -nodes -out /home/pi/garage-door-controller-cert/localhost.crt -newkey rsa:4096 -sha256 -keyout /home/pi/garage-door-controller-cert/localhost.key -subj "/CN=localhost"`
    
    `chmod 700 /home/pi/garage-door-controller-cert`
    
    `chmod 600 /home/pi/garage-door-controller-cert/*`
    
6.  **Edit `config.json`**
    
    You'll need one configuration entry for each garage door.
    The prefixes <R\> <1\> or <2\> indicate those required by each sensor configuration.  <S\> is special.  Any setting may be omitted if the default is correct.  If there are <1\> settings for a door with two sensors they are ignored, and vice-versa.

    The settings are defined as follows:

    - <R\> **name**: The name for the garage door as it will appear on the controller app.
    - <R\> **sensor_count**: (1 or 2) sensors per door. **Default: 1**.
    - <R\> **relay_pin**: The GPIO pin connecting the RPi to the relay for that door. Not needed if the door has no opener.
    - <1\> **state_pin**: The GPIO pin connecting to the single 'closed' sensor.
    - <1\> **state_pin_closed_value**: The GPIO pin value (0 or 1) that indicates the door is closed. **Default: 0**.
    - <1\> **approx_time_to_close**: How long the garage door typically takes to close. **Default: 15**.
    - <1\> **approx_time_to_open**: How long the garage door typically takes to open. **Default: 15**.
    - <2\> **open_pin**: The GPIO pin connecting to the 'open' sensor.
    - <2\> **open_pin_open_value**: The GPIO pin value (0 or 1) that indicates the door is open. **Default: 0**.
    - <2\> **closed_pin**: The GPIO pin connecting to the 'closed' sensor.
    - <2\> **open_pin_open_value**: The GPIO pin value (0 or 1) that indicates the door is open. **Default: 0**.
    - <S\> **openhab_name**: Required when linking to openHAB.

    The **approx_time_to_XXX** options are not particularly crucial.  They tell the program when to shift from the opening or closing state to the "open" or "closed" state when there is only a closed sensor. You don't need to be out there with a stopwatch and you wont break anything if they are off.  In the worst case, you may end up with a slightly odd behavior when closing the garage door whereby it goes from "closing" to "open" (briefly) and then to "closed" when the sensor detects that the door is actually closed.

        
7.  **Set to launch at startup**

    Simply add the following line to your /etc/rc.local file, just above the call to `exit 0`:
    
    `(cd ~pi/garage-door-controller; python controller.py)&`
    
8. **Using the Controller Web Service**

    The garage door controller application runs directly from the Raspberry Pi as a web service running on port **8081**, or port **8444** when HTTPS is enabled.  It can be used by directing a web browser (on a PC or mobile device) to **http://[hostname-or-ip-address]:8081/**, or **https://[hostname-or-ip-address]:8444/** when HTTPS is enabled.  If you want to connect to the raspberry pi from outside your home network, you will need to establish port forwarding in your cable modem.  
    
    When the app is open in your web browser, it should display one entry for each garage door configured in your `config.json` file, along with the current status and timestamp from the time the status was last changed.  Click on any entry to open or close the door (each click will behave as if you pressed the garage button once).

TODO:
----------  
This section contains the features I would like to add to the application, but do not currently have time for.  If someone would like to contribute changes or patches, I would be all to happy to incorporate them.

* *Security*: Impose a configurable password on the web service.  Would need to discuss the best strategy (i.e., should we require the pw every time, or can the session persist on any given device which has authenticated).
* *New Feature*: Add a "close all" button to the bottom of the page to close all doors that have a state other than "closed" or "closing"
* *Occupancy sensors*: Add proximity sensors to check if car port is in use
* *IFTTT Integration*: make a smooth secure way to call the door and get information online


  [1]: http://i.imgur.com/rDx9YIt.png
  [2]: http://i.imgur.com/2Uvye9j.png
  [3]: http://i.imgur.com/vPHx7kF.png
  [4]: http://i.imgur.com/AkNl6FI.jpg
  [5]: http://i.imgur.com/48bpyG0.png
  [6]: http://i.imgur.com/qQmahvC.png
  [7]: http://i.imgur.com/Wqn76S1.png
  
