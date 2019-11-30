---
layout: post
title: Remotely Turn On Your PC Over the Internet
comments: true
---

当你有一台 PC 主机，上面存储了你日常工作和学习的资料。而你此时希望通过 Macbook, 或者 iOS or Android 设备访问文件时。你通常需要移动鼠标，或者按动键盘才能使处于 'sleep' 状态的 PC 唤醒。这通常不难，但是现在我们要一种更加简单的方式。

> Wake-on-LAN (WoL) is an Ethernet or Token ring computer networking standard that allows a computer to be turned on or awakened by a network message - [Wikipedia](https://en.wikipedia.org/wiki/Wake-on-LAN)

## Magic packet

> The magic packet is a broadcast frame containing anywhere within its payload 6 bytes of all 255 (FF FF FF FF FF FF in hexadecimal), followed by sixteen repetitions of the target computer's 48-bit MAC address, for a total of 102 bytes. [MagicPacket](https://en.wikipedia.org/wiki/Wake-on-LAN#Magic_packet)


```js
/**
 * [createMagicPacket]
 * @param  {[type]} mac [description]
 * @return {[type]}     [description]
 * @wiki https://en.wikipedia.org/wiki/Wake-on-LAN
 * @docs http://support.amd.com/TechDocs/20213.pdf
 */
function createMagicPacket(mac){
  const MAC_LENGTH    = 0x06;
  const MAC_REPEAT    = 0x16;
  const PACKET_HEADER = 0x06;
  var parts  = mac.match(/[0-9a-fA-F]{2}/g);
  if(!parts || parts.length != MAC_LENGTH)
    throw new Error("malformed MAC address '" + mac + "'");
  var buffer = new Buffer(PACKET_HEADER);
  var bufMac = new Buffer(parts.map(function(p){
    return parseInt(p, 16);
  }));
  buffer.fill(0xff);
  for(var i = 0; i < MAC_REPEAT; i++){
    buffer = Buffer.concat([ buffer, bufMac ]);
  }
  return buffer;
};
```
