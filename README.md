# RPiHomeAutomation
Home automation using Raspberry Pi 3B and OpenHAB

I got interested in making my own room automated after watching an Instagram video. Had an old RPi-3 lying around and used it to automate my room.

## Componenets needed
- RPi - any model would suffice as long as it connects to the internet
- Relay Module
- Google Assistant (or Google Home)

### Software Used
- OpenHABian OS
- IFTTT

## Getting Started

### Flashing OpenHABian

- Download the latest OpenHABian system image.
- Flash it onto a SD-card using Etcher.io
- After successful flashing, put the SD-card into Rpi and connect an Ethernet to the Rpi. (You may also use WiFi).
- Boot up the Rpi. You don't need a screen and/or mouse and keyboard as SSH and Samba are enabled already!
- Wait for 30-45 minutes for OpenHAB to finish its initial setup.
- After that, go to http://openhabianpi:8080, This will be the address of your Rpi from where you can access it.

### Configuring OpenHAB

After OpenHAB finishes its initial setup, go to

http://openhabianpi:8080.

- There navigate to Paper UI.
- There, go to Addons>Bindings. Search GPIO in the search bar. Install GPIO binding. the navigate to MISC tab and install openHAB Cloud Connector.

### Configuring OpenHAB

Now we need to access our Rpi through SSH. I am going to use PuTTY. If you are on MacOS or Linux, you can use terminal.

SSH through PuTTY-

- Open PuTTY.
- Go to your router's admin page and find out the IP address. It will be named as OpenHABian.
- Copy the IP address and paste it in PuTTY and click Open.
- Now you need to login -

```
login as: openhabian
password: openhabian
```

- After logging in, type in the following commands

```
$ cd /etc/openhab2
$ ls
```

- Now it will show you all the available directories. We will be using - items (to create different items), rules ( to enable voice commands) and sitemaps ( to create a sitemap for navigation). We will create a sitemap as - home.sitemap. Items file would be - home.items. And rules file would be - home.rules.

```
$ sudo nano items/home.items
$ password: openhabian
```

- This would open a blank document. We will here, create our items that we will be controlling through Rpi. In my case, I used 4 items. You can use as many as you want.

```
//Items File

Switch fan "Fan" <fan_ceiling> { gpio="pin:17 activelow:yes initialValue:low" }
Switch night_light "Night Lamp" <light> { gpio="pin:27 activelow:yes initialValue:high" }
Switch exhaust "Exhaust Fan" <fan_box> { gpio="pin:23 activelow:yes initialValue:high" }
Switch light "Light" <light> { gpio="pin:5 activelow:yes initialValue:low" }

String VoiceCommand
```

- Here, I would explain the above with an example- Switch fan "Fan" <fan_ceiling> { gpio="pin:17 activelow:yes initialValue:low" } What happens here is as follows-

- Switch - it is a keyword that defines that the item is a switch

- fan (generic - name it anything you want) - it is a user-defined identifier for naming different items that one wants to control.

- "Fan" (generic - name it anything you want)- it is the display name that will be displayed in the UI.

- <fan_ceiling> (icon name)- It is the name of the icon that will be displayed along with the name.

- { gpio="pin:17 activelow:yes initialValue:low" } - here gpio is the thing that tells OpenHAB that the item is connected through gpio. pin:17 is the pin that you connect the relay to. activelow:yes(or no) - Active low means that when the switch is off there will be no voltage applied to the gpio pin and when the switch is on there will be voltage applied. initialValue:high (or low) - After that is initialValue and what this does is tell openhab what to set the initial value of the item during initialization. This one is set to high because I want the switch to be off during initialization.

- String VoiceCommand - it is the item that will be used to control other items using voice commands.

- You can create as many items as you want using this syntax-

```
type item-name "item-display_name" <icon-name> { gpio="pin:pin-no activelow: (yes or low) initialValue: (high or low)
```

- After doing this, press Ctrl+X, then Y and Enter.

```
$ cd..
```

### Creating a Sitemap

Sitemap would be used for navigation and control of the relay switches.

- Considering you are continuing after completing above steps, type in terminal

```
$ sudo nano sitemaps/home.sitemap
```

- This file would be the default sitemap for navigation. The above command will open a blank file. You need to create a sitemap as follows-

```
sitemap home label="Smart Home"
{
       Frame label="My Room"
       {
               Switch item=fan
               Switch item=light
               Switch item=exhaust
               Switch item=night_light
       }
}
```

Well, this is a long code with many files and commands. But you only need to consider the following to create your own set of commands.


- if (command.contains("turn on fan") || (command.contains("turn on the fan"))) - here I have listed two options I can say to make the command work. What happens in actual is, when I say the reserved line, OpenHAB recognizes it and checks for the specific rule to do what occurs next.

- fan.sendCommand(ON) - When the above condition is true, this function sends a command ON to the item fan. This can be modified according to your choice.

```
item-name.sendCommand(ON/OFF)
```

Well, if you have made it to here without any problems, Congrats, because most part of the work is done. Now we need to setup the UI and enable remote access for our OpenHAB.


### Configuring BasicUI

Now, we need to tell OpenHAB to use the sitemap we create to use it as the default one. Here's how to do it-

- Go to http://openhabianpi:8080
- Open Paper UI
- Configurations > Services > UI > Configure Basic UI
- Here you can select the theme and icon formats, etc. The main thing you need to do is to change the default sitemap to home
- Click Save
- You can view it by going to http://openhabianpi:8080
- Click on Basic UI and voila you would see your own sitemap there

#### Enabling Remote Access

To enable remote access, follow these steps-

- Go to http://openhabianpi:8080
- Open Paper UI
- Configurations > Services > IO > Configure OpenHAB Cloud
- Change mode to Notifications and Remote Access, Base URL -> https://myopenhab.org/ and items to expose -> Select all of them
- Click Save
- Proceed to http://myopenhab.org
- Sign Up with email address and password.

- For openHAB UUID ->

```
$ sudo nano /var/lib/openhab2/uuid
```

- Copy and paste this UUID into the UUID column.

- For OpenHAB secret->

```
$ sudo nano /var/lib/openhab2/openhabcloud/secret
```

- Copy and paste it into Secret column and hit Sign Up.

- After signing up,

```
$ sudo reboot
```

- Now after Rpi reboots successfully, you will see the status as online in the https://myopenhab.org

- Go to items tab

- Here you can see all of your items you created. If you don't see anything, you need to toggle all those items at least once.

### Hardware

Be very careful, as we would be handling 220V and other electric things.

** BE CAUTIOUS. YOU WILL BE DOING THIS AT YOUR OWN RISK. **

So let's get started.

Connecting the Relay to the Raspberry Pi

To connect the relay to the Raspberry Pi, first remove the jumper between the VCC and JD-VCC.

- Connect the JD-VCC and VCC to 5V each on Raspberry Pi
- Connect GND on relay to GND of Raspberry Pi
- Next connect IN1, IN2,... to the GPIO assigned in the home.items
- To check if everything is working, navigate to BasicUI and try turning off and on the different items. You should hear a Clicking sound on each toggle.


You can also download the OpenHAB app from Play Store / App Store for easier control of your Automation System.

Now we need to connect the wires of the appliances you want to automate to the relay switches. Turn off Rpi and Main Supply before doing this to be on the safer side.

After connecting wires, make sure there is no live wire left uncovered that may prove to be fatal.

Now turn back on your Rpi and give it time to boot. After booting, you will be able to control appliances from Basic UI or from the mobile app.

If you don't want voice automation, you don't need to follow the net steps.

### Connecting with Google Assistant

For this we will use IFTTT.com

- Go to IFTTT.com
- Create an account if you don't have one.
- Click on New Applet
- Select This and select Google Assistant and select Say a phrase with a text ingredient
- In what do you want to say, enter - Turn $ item-name Ex- Turn $ fan
- Click create trigger
- Select that and select OpenHAB. Link your account
- Choose send a command
- Select item as VoiceCommand
- Command to send as - Turn {{TextField}} item name. Ex- Turn {{TextField}} fan
- Create Action

Give around 10 seconds for it to initialize and then Voila, use google assistant to send the command.

## Schematic

![image](https://user-images.githubusercontent.com/71216807/121498779-61844f00-c9fa-11eb-8dfe-7317429d33dd.png)


The code that I used is provided in the respective folder.
