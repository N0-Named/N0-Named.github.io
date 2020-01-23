---
layout: post
title: "making esp8266 deauther"
date: 2020-01-24 
excerpt: "ESP8266 NodeMCU을 이용해서 WIFI 공격 도구를 만들 수 있다."
project: true
tags: [reversing,pwnable,hacking,exploit,system,codegate]
comments: false
---

**ESP8266 NodeMCU**을 이용해서 **WIFI** 공격 도구를 만들 수 있다.
[https://github.com/spacehuhn/esp8266_deauther](https://github.com/spacehuhn/esp8266_deauther)

이 도구의 주요 공격은 **deauthentication attack**  (인증 해제 공격) 이다.

 **IEEE 802.11 표준**의 큰 취약점을 이용한 것으로, 인증 해제 패킷만 뿌리기만 하면 기기의 와이파이 연결이 해제된다.

deauthentication 패킷을 뿌리는 시간동안 와이파이 연결이 계속 끊기게 되므로, 사실상 해당 와이파이를 사용할 수 없게 되는 것이다.

이 취약점을 해결하기 위해 2009년에 802.11w라는 것이 나왔는데 이 기술이 적용된 제품은 매우 극소수이다.

이 프로젝트의 목적에는 해당 취약점이 있는 회사가 해당 문제를 해결하게 하는 데에 있다고 한다.

----------
#### 주의

깃허브에도 나와있지만, deauthentication attack 은 zamming(재밍) 이 아니다.

공유기가 사용하는 2.4Ghz나 5Ghz 주파수 대역을 방해하는 방식이 아니다.

단순히 deauth 패킷만을 뿌릴 뿐이다.

(패킷하나 날렸다고 해당 공유기를 사용하는 이용자들의 와이파이 연결을 끊을 수 있다니..)


**재밍은 전파법에 걸린다. 하지말자.**

**deauth공격도  반드시  허가 받은 기기에만 시도하자.**

----------

제작 방법은 위키에 나와있다.

[https://github.com/spacehuhn/esp8266_deauther/wiki](https://github.com/spacehuhn/esp8266_deauther/wiki)

나는 l2c로 작동하는 oled display, 네오픽셀 led, 버튼 3개를 구입했다.

SPI방식의 oled 디스플레이를 구매하면, 연결해야되는 핀 개수가 늘어난다. 그러면 납땜할때 귀찮아진다.. 그냥 l2c 방식으로 구매하는 것을 권장한다.

네오픽셀 led는 현재 작동 상태를 보여준다.

버튼은 3개~6개까지 달 수 있다.

기본 : UP, DOWN, A

추가 : BACK, LEFT, RIGHT

테스트를 위해 브레드보드와 점퍼케이블도 구매하자.

0.  **NodeMCU V1.0 Lua WiFi ESP8266**

[http://mechasolution.com/shop/goods/goods_view.php?goodsno=539585&category=](http://mechasolution.com/shop/goods/goods_view.php?goodsno=539585&category=)

1.  **버튼**

[http://mechasolution.com/shop/goods/goods_view.php?goodsno=542428&category=](http://mechasolution.com/shop/goods/goods_view.php?goodsno=542428&category=)

2.  **네오픽셀 led**

[http://mechasolution.com/shop/goods/goods_view.php?goodsno=540705&category=](http://mechasolution.com/shop/goods/goods_view.php?goodsno=540705&category=)

3.  **0.96인치 12864 OLED LCD 모듈 4핀**

[http://mechasolution.com/shop/goods/goods_view.php?goodsno=540942&category=](http://mechasolution.com/shop/goods/goods_view.php?goodsno=540942&category=)

4.  **브레드보드 400핀 - 하프사이즈**

[http://mechasolution.com/shop/goods/goods_view.php?goodsno=7&category=](http://mechasolution.com/shop/goods/goods_view.php?goodsno=7&category=)

5.  **점퍼케이블**

20cm를 사용했었는데 충분하다.

------------------

만능기판에 전부 납땜해서 연결을 해주었다.

.납땝할 때 사용한 선은 집에 굴러다니던 선 잘라서 사용했다.

.철사로 해봤는데 잘 안된다.

.케이블에서 나온 전선 부분을 동그랗게 올가미 형태로 해주면 납땜이 잘 된다.

![](/assets/img/img2/ardu0-0.png) ![](/assets/img/img2/ardu0-1.png)

스위치 하나를 SD3에 연결했다. 납땜 후 제대로 작동을 안해서,, 글루건으로 고정해주었다.

집에 버튼 남아서 back용으로 하나 더 달아주었다.

----------

**사용 방법**은 3가지가 있다.

시리얼, 웹, oled 디스플레이

**시리얼**을 이용하면 여러 명령어를 사용할 수 있다.

다만, 반드시 컴퓨터와 usb로 연결해서 시리얼 모니터를 열어야만 한다.

**웹**은 컴퓨터와 usb로 연결하지 않고 와이파이로 연결해서 웹페이지를 이용해 제어할 수 있다.

그래서 기기와 스마트폰만을 가지고 휴대하면서 사용할 수 있게 해준다.

하지만 기기의 와이파이에 접속해야한다는 불편함이 있다. (+ 당연히 인터넷이 안된다.)

기기에 oled display, 버튼3개이상을 달아주면 기기만 가지고 제어를 할 수 있다.

----------

메인 메뉴를 보면 5가지가 있다.

**SCAN**
------------

스캔을 통해 공격할 대상을 찾는다.

- SCAN AP + ST

- SCAN APs

- SCAN Stations

세 가지가 있는데, AP는 공유기, Stations는 연결된 장비들 인 것 같다.

**SELECT**
----------

스캔으로 찾은 목록을 보여주며, 이 곳에서 공격할 대상을 지정할 수 있다.


 **ATTACK**
 -----------------

공격을 할 수 있다. 세 가지의 공격이 있다.

-  **deauth**는 인증 해제

- **beacon**은 가짜 와이파이를 비콘을 뿌린다. 연결을 시도하면 연결이 되지 않는다.
 ![](/assets/img/img2/ardu0-2.jpg)

어! 무료 와이파이이다! 하고 연결을 시도하면 연결이 되지 않는 공격

 - **probe**는 아직 잘 모르겠다.  _연결 잘 되던데..?_

**PACKET MONITOR**
-------------

선택한 채널에 돌아다니는(?) 패킷의 양을 보여준다.
![](/assets/img/img2/ardu0-3.jpg)

 **CLOCK**
 ------------

그냥 시계인데, 전원을 끈 시간동안은 시계가 멈추기 때문에 전원을 계속 연결 주고 있어야 한다.
