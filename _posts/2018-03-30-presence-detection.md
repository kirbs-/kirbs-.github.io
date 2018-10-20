---
layout: page
title: Room Presence Detection with Room Assistant and a Raspberry Pi Zero W
order: 1
---
Getting reliable presence detection is not easy. I've tried apps, BLE, and wifi based presence tracking platforms, but none proved quick, reliable, and simple. All of these methods rely on polling to determine if a given phone is present in an area. They also require bluetooth or wifi be enabled on your phone.

My solution was to stop using my phone for presence detection and use a BLE iBeacon instead. Radbeacon Dots, made by Radius Networks, fit perfectly. Small, cheap and easy to configure. This guide wi

First, lets setup the nodejs environment and room-assistant dependencies.
```
sudo apt-get install nodejs nodejs-legacy npm git
npm install bleacon bluetooth-hci-socket
```
Next, install room-assistant. Note, this will install in the current working directory.
```
git clone
cd room-assistant
npm install
```
Now to configure room-assistant to listen for the Radbeacon and publish updates via MQTT.
```
cp config/default.json config/local.json
```
