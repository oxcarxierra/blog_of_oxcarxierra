---
layout: post
title: RS485 통신으로 DC모터 제어하기
tags: [Tech, Robotics]
usemathjax: false
unsplash: "iVfOFaEghqU"
image_username: "ubahnverleih"
---

인턴으로 일하면서 기계과에선 배우지 않는 Serial communication에 대해 알게되었다. 그 중 범용적으로 사용되는 프로토콜인 RS-485로 [ZLAC8015D](http://www.zlrobotmotor.com/info/401.html)라는 서보드라이버와 통신하며 로봇에 사용될 Wheel-In Motor를 제어해보았다. 


### Serial Commmunication

신호선으로 데이터를 주고받는 방법. 대개 High/Low Voltage로 1/0을 구분하여 binary의 형태로 송수신한다.

- 직렬(Serial)과 병렬(Parallel)

하나의 선으로 데이터를 전송하면 직렬, 여러개의 선을 이용해서 전송하면 병렬. 단위시간당 전송하는 데이터 양은 병렬 통신이 더 많지만 단자의 소형화를 위해 직렬 통신이 더 많이 쓰임.

- 동기 시리얼 통신(Synchronous)과 비동기 시리얼 통신(Asychronous)

동기 시리얼 통신은 별도의 Clock선을 이용, 비동기 시리얼 통신은 그렇지 않은 대신 데이터 패킷의 시작과 끝에 대한 정보를 같이 보내야 함. 비동기 시리얼 통신이 리소스도 적게 들고 간단해서 많이 쓰임.

- 프로토콜 종류

비동기 통신은 RS-232와 RS-485, CAN통신 정도가 범용적으로 많이 쓰임. 그 중 RS-485은 1:N 통신이 가능하다는 특징이 있다.

### Motor Control

[Github Repo : oxcarxierra/ROS2_ZLAC8015D_serial](https://github.com/oxcarxierra/ROS2_ZLAC8015D_serial){:target="\_blank"}

