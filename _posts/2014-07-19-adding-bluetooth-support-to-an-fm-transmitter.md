---
layout: post
title: Adding Bluetooth Support to an FM Transmitter
category: articles
tags: [bluetooth, radio]
---

A few years ago I wanted to build an FM transmitter from a kit. I settled on the [FM25B](http://www.ramseyelectronics.com/Synthesized-FM-Stereo-Transmitter-Kit/dp/B0002NRIM8?class=quickView&field_availability=-1&field_browse=6290123011&id=Synthesized+FM+Stereo+Transmitter+Kit&ie=UTF8&refinementHistory=brandtextbin%2Csubjectbin%2Cprice&searchNodeID=6290123011&searchPage=1&searchRank=salesrank&searchSize=12) from Ramsey. It's delivers really nice stereo quality for the price. I did a few experiments with it for a while and when I got tired of that I started using it as a way to broadcast music around my house. My main stereo system has a radio and I also have a [shower radio](http://www.sangean.com/products/product.asp?mid=93). It's really nice to play music or internet radio in the morning and be able to walk around or take a shower and continue to listen.

The way I had set it up before was that I just always had a hanging 1/8th cable coming off my bookshelf. This really felt like a hassle sometimes and it's unsightly, so this weekend I decided to try to retrofit the transmitter with a bluetooth audio receiver.

### Parts

I looked around on the internet and I found what I think would be a [good device for this](http://www.parts-express.com/bt-1a-bluetooth-receiver-module-for-wireless-reception-of-bluetooth-audio-signals--320-353?utm_source=google&utm_medium=cpc&utm_campaign=pla) but I didn't want to wait for shipping so instead I walked over to Best Buy and bought [one of these](http://www.bestbuy.com/site/aluratek-istream-universal-bluetooth-audio-receiver/6581843.p?id=1218759758130&skuId=6581843&st=bluetooth%20audio&cp=1&lp=1). This thing has a battery (which we won't need).

Since I wanted to power the device with the same power source as the radio, I knew i would need some kind of voltage regulator. I am assuming this thing runs off 3.3V because it is powered by USB so I stopped by RadioShack to look for a 3.3V regulator but could only find a [5V regulator](https://www.sparkfun.com/datasheets/Components/LM7805.pdf). I decided to just try and make this work anyway.

### Build

First step was to find some good points to leach off the radio. I looked around on the circuit and tested some spots until I found a place located after the filtering caps that was providing a clean power signal. I suspected that the 5V might be fine if the device had an internal regulator. I built this test rig and let it sit for a while to see if anything got hot.

<img alt="Test Circuit" src="/public/images/test_circuit.jpg">

Turns out it handled it just fine. Next thing I did was freeform solder the circuit then stick it into the transmitter. I ran the audio cable out the back and looped it back into the input. I can easily hardwire this later but for now it's not noticeable.

<img alt="Finalized Radio" src="/public/images/finalized_radio.jpg">

Now when I am at home, I can just choose my transmitter in my bluetooth options and it will route all the audio to all the speakers in my house.

<img alt="Bluetooth setting" src="/public/images/bluetooth_setting.jpg">

