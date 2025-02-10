# Mitsubishi-MSZ-HJ25VA-Home-Assistant-Integration
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
