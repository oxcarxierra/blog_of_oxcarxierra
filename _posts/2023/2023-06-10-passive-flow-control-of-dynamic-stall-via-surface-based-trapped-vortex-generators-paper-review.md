---
title: "[Paper Review] Passive Flow Control of Dynamic Stall via Surface-Based Trapped Vortex Generators"
tags: ["Engineering"]
date: 2023-06-10 15:17:56
---

> _Al-Jaburi, Khider, and Daniel Feszty._ **"Passive flow control of dynamic stall via surface-based trapped vortex generators."** _Journal of the American Helicopter Society 63.3 (2018): 1-14._

# Abstract

Lift를 크게 감소시키지 않으면서 Drag와 Negative pitching moment를 감소시키는 NACA0012 airfoil의 surface modification을 제안.

# Introduction

Retreating blade에서는 유효 받음각의 빠르고 주기적인 변화로 인해 dynamic stall이 발생한다. 이는 로터 허브나 제어장치의 진동으로 부정적인 영향과 직결되며, 최대비행속도가 제한되는 요인이기도 하다.
몇몇 선행연구에서 영감을 받은 계단형 에어포일이 해당 문제를 해결할 수 있는지에 대한 타당성을 조사해보는 연구이다.

# Conclusion

제안된 형태의 에어포일을 사용하면 retreating blade에서 발생하는 dynamic stall이 감소한다. Airfoil에 추가한 step이나 cavity가 "trapped vortex" 즉 일종의 와류를 만들어서 표면에 걸리는 힘을 재배치하는 원리라고 설명한다.
에어포일의 위쪽 표면을 가공한 경우가 아래쪽보다 성능이 우수했다. 가장 성능이 좋았던 모델의 경우 negative pitching moment를 50-60%, drag를 30-40% 가량 줄일 수 있었다. 받음각이 최대가 되는 구간에서도 drag의 감소를 확인할 수 있었다.
실제 에어포일에서는 "groove"의 형태로 적용되어 dynamic stall을 줄일 수 있을 것.

<fig>
<img src="https://i.imgur.com/esnylgB.png" />
</fig>

# Terminology

## Retreating Blade와 Dynamic Stall

<fig>
<img src="https://i.imgur.com/Y8FHgRZ.png">
</fig>

일반적인 Stall은 에어포일의 받음각이 임계받음각을 넘어 증가했을 때, 표면에서 유동 박리가 발생함에 따라 양력계수가 작아지는 현상을 의미한다. 특히 그 중 하나의 종류인 Dynamic Stall은 unsteady flow 내에서 에어포일의 받음각이 빠르게 바뀔 때(주로 증가할 때) 발생하는 현상이다. 고정익기의 기수를 급격하게 드는 경우에 문제가 되기도 하나, 주로 헬리콥터의 retreating blade에서 발생하게 된다. 헬리콥터에서 발생하는 dynamic stall은 블레이드에 강한 진동을 발생시키고 부하를 발생시킨다.

헬리콥터의 blade는 기체와 같은 방향으로 움직일 때 advancing blade가 되며 반 바퀴 회전하면 기체와 다른 방향으로 후퇴하는 retreating blade가 된다. 공중에서 멈춰있을 때는 상관이 없지만, 헬리콥터가 전방 v의 속력으로 기동하는 상황을 가정하면 advancing blade는 `v+rw`, retreating blade는 `v-rw`의 서로 다른 유속에 놓이게 된다. **(Dissymmetry of Lift)**

일반적으로 블레이드의 받음각을 조절하는 "flap"을 통해 이 문제를 해결한다. 블레이드가 Retreating blade Side에 진입하면, 낮은 mach number에서도 동일한 lift를 발생시키기 위해 블레이드를 회전시키면서 더 큰 받음각을 설정하는 방식이다.

이 방식의 가장 큰 문제점은, 받음각이 임계받음각에 근접하거나 초과하게 되는 경우 retreating blade에서 유동 박리가 발생하며 필요한 만큼의 lift를 받지 못하게 된다는 점이다. **(Retreating Blade Stall)** 고정익의 경우 수직하강으로 stall에서 회복될 수 있지만, 이 경우 advancing blade에서는 계속 양력을 받아서 기체에 retreating blade쪽으로의 강한 Roll moment가 발생하게 된다.

## Flight Envelope

<fig>
<img src="https://i.imgur.com/jYhXck6.gif">
</fig>

속도, 엔진 추진력, 조작성, 풍속, 고도 같은 안전한 비행을 위해 넘어서면 안되는 내적 또는 외적 최저/최고치.
