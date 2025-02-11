# Mitsubishi MSZ-HJ25VA Home-Assistant integration
Modding and integrating Mitsubishi MSZ-HJ25VA HVAC into Home Assistant via ESPHome

# Background information
My old MSZ-HJ25VA air conditioner does not support WiFi module. It looks like all the HVACs created after do have a port to connect to the WiFi module.<BR>
I started searching online and I found some interesting information about Mitsubishi AC in general.
Fist of all, all the Mitsubishi HVACs use a connector called CN105 that is used to connect the WiFi module. And there is a very nice project on github that enable the support of this connector using an ESP device:
<p>https://github.com/SwiCago/HeatPump (Arduino library to control Mitsubishi Heat Pumps via connector CN105.)</p>
<p>This library is meant for those who likes coding with the arduino IDE, that is not (yet) my case. So I digged a little more and I found another project on github that leverages the SwiCago and embeds into ESPHome:</p>
<p>https://github.com/geoffdavis/esphome-mitsubishiheatpump (ESPHome Climate Component for Mitsubishi Heatpumps using direct serial connection)</p>
<p>This project looks very promising. I further found a fork that seems to add few more features and is slightly more up to date:</p>
<p>https://github.com/echavet/MitsubishiCN105ESPHome (ESPHome firmware inspired by GeoffDavis’s esphome-mitsubishiheatpump, directly integrating the SwiCago library within its codebase)</p>
<p>From a software point of view, it looks like you can connect a ESP device and manage you HVAC without the need to buy expensive modules like the MAC-587IF-E.<br>
With those projects in my mind, I started looking if there were an option to replace the logic board of my MSZ-HJ25VA or if I had to abandon the idea of connecting it to my Home Assistant. I found several details on how to use IR component, eventually the SmartIR integration for Home Assistant, but using infrared lacks the ability to check the status of the heat pump: you can send commands but you do not know is some changed the configurations using the remote.</p>
<p>I run into this post on the HA community:</p>
<p>https://community.home-assistant.io/t/mitsubishi-ac-with-wemos-d1-mini-pro/107007/246?page=13</p>
<p>where user <i>vperas</i> found that the logic board of his MSZ-HJ35VA (the same family, jost more BTUs!) has the layout of the CN105 connector printed on the board: it just lack 2 resistors and the connector itself and this was confirmed in this topic too, although many comments were deleted but the owner being off-topic:</p>
<p>https://github.com/SwiCago/HeatPump/issues/13</p>
<p>So let's start and see how I did it.</p>

# Requirements
Besides being able to solder (and you will need to solder 2 SMD resistors and a connector), I bought the following items:
- S05B-PASK-2 connectors (https://it.aliexpress.com/item/1005008145895889.html)
- Optional PAP-05V-S connectors (https://it.aliexpress.com/item/1005007784137238.html)
- ATOMS3 Lite ESP32S3 M5Stack (https://it.aliexpress.com/item/1005005177952629.html)
- 4 Pin Jumper Wire HY2.0mm Pitch Pin Universal Grove Buckled Cable 20cm (https://www.amazon.it/dp/B0D4PFPSMY?ref=ppx_yo2ov_dt_b_fed_asin_title)
- 0x 0603 SMD 1200ohm resistor (https://www.amazon.it/dp/B0DGQ6GKMD?ref=ppx_yo2ov_dt_b_fed_asin_title)

<p>I choosed the Atom S3 Lite because it is very small and can be connected easly with the Grove 4 pin cable. If you want the best result, you can replace one connector of the grove cable and use the proper PAP-05V-S connector: we need only 4 wires. Alternatively you can cut one edge of the connector and the clip to let it fint into the S05B-PASK-2 connector soldered on the board.<br>
I wasnt able to find a bounce of 10 120ohm SMD 0603 resistor, so I bought a kit.</p>

# Opening the MSZ-HJ25VA unit
Opening the unit is fairly simple: you need to remove 2 small covers on the bottom left and right under the vane and unscrew 2 philips screws.<br>
<img src="https://github.com/user-attachments/assets/7458a613-9210-45fd-a72b-26e8fde128eb" width=40% height=40%>
<img src="https://github.com/user-attachments/assets/131faec3-24f3-4d79-abce-e5b3613fabc1" width=40% height=40%>

Slightly pull from the bottom after checking a push/pull plastic lock in the middle of the vane.<br>
<img src="https://github.com/user-attachments/assets/4edb1b67-ed52-4e61-a8e8-47cc91f4f9df" width=60% height=60%>

On the left you will have access to the logic board:<br>
<img src="https://github.com/user-attachments/assets/94275b12-6e80-4315-827e-07e6bd1de4b8" width=40% height=40%>

Unplug the three connectors and remove the board with the plastic cover: on the bottom left you can unlock the plastic. Gently turn the board and, once opened on the side, pull it up and remove it from the chassy. Pay attention the remove gently all the wires from the plastic.

# The logic board
Once in your hand, the logc board should look like this:<br>
<img src="https://github.com/user-attachments/assets/acda0ef4-6f64-429a-9626-242f961b7dd8" width=40% height=40%>
<img src="https://github.com/user-attachments/assets/b663f6fd-93d8-4fb3-bade-142b75e21e14" width=40% height=40%><br>

First clean the hole where you'll need to place the connector and the resistors:<br>
<img src="https://github.com/user-attachments/assets/d0e0b3cc-f719-41e4-8046-34d8eae2daf6" width=40% height=40%>
<img src="https://github.com/user-attachments/assets/8f1672e3-6476-4efb-9a9f-018c4e936e10" width=40% height=40%>

Remove the flux with isopropilic alchool (I took the picture before cleaning it), and the solder the connector and the 2 resistors. Resistors are 0603 SMD rated 1200 ohm (the lable on top is 122).<br>
<img src="https://github.com/user-attachments/assets/274a48a7-52f9-4889-9ba8-d15307db80b1" width=40% height=40%>
<img src="https://github.com/user-attachments/assets/e45d8c9d-b9ae-4f5f-b67d-cc18b85b65a3" width=40% height=40%>

these are not my best tin welds, but they works!

# The ATOM S3 Lite with ESPHome
Here is the M5Stack Atom S3 Lite with a 4 pin Groove connector:<br>
![IMG_20250209_173758](https://github.com/user-attachments/assets/42ae5aa2-9ee2-4901-b587-da40df8a0344)

The cable properly fits the Atom S3 Lite and can be adapted to fill the S05B-PASK-2 on the logic board by cutting the side border (near the black cable) and the hook:<br>
![IMG_20250209_174248](https://github.com/user-attachments/assets/3ab83282-8e8d-403a-a50b-a13560b6ab16)

As a reference, you will find a sample configuration of the ESPHome device:<br>
[mitsubishi-hvac-sample.yaml ](mitsubishi-hvac-sample.yaml)<br>
Please replace the <macaddress> or the names on the _substitutions_ section. Considering I have 2 splits I make use of the lasst 6 char of the mac address to get different names and ids.<br>
This yaml uses the ATOM S3 Lite led as a status led. I added a lambda function to deliver different colors depending from the mode of the HVAC (OFF, FAN_ONLY, DRY; COOL, HEAT).<br>
A "Status Led" switch is used to keep the led always off.


