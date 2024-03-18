---
title: "[논문 리뷰] Dyanamic Window Approach (DWA) 알고리즘"
tags: [Tech, Robotics]
usemathjax: true
unsplash:
image_username:
---

<fig>
<img src="https://i.imgur.com/gnh04l0.png">
<figcaption>Fox, D., Burgard, W., & Thrun, S. (1997). The dynamic window approach to collision avoidance. IEEE Robotics & Automation Magazine, 4(1), 23-33.</figcaption>
</fig>

Dynamic Window Approach (DWA) 알고리즘은 로봇이 장애물을 회피하면서 목표 지점으로 갈 수 있는 경로를 찾는 Path planning 알고리즘이다. 이 경로와 더불어 경로를 따라가도록 하는 로봇의 속도와 각속도도 결정하기 때문에 local planner 혹은 경로추종에 사용된다. 1997년 발표된 Fox와 Thrun의 논문에서 처음 소개되었는데 다양한 variation으로 아직까지도 실용적으로 많이 사용되는 것 같다. (Nav2 라이브러리에도 DWA의 확장인 dwb_controller가 있다.) 한국어로 잘 요약된 자료가 검색해도 잘 안나오길래 내가 이해한대로 힌 번 정리해봤다.

# The Dynamic Window Approach

DWA 알고리즘의 특징은 waypoint까지의 경로를 계산하는 top-down 방식이 아닌 로봇이 갈 수 있는 경로들 중 waypoint에 가까워지는 경로를 선택하는 bottom-up 방식의 접근을 한다는 점이다. 로봇의 속도와 각속도를 알고있다면 t초 뒤 위치도 알 수 있으므로 로봇이 이동하게 될 경로를 미리 계산할 수 있다. 이 때 속도와 각속도를 상수로 고정하면 그 경로는 원형, 정확히는 호를 그리게 되는데 이를 논문에서 **curvature**로 정의한다.

## Search space

로봇의 다음 속도와 각가속도로 가능한 영역(Search space)은 다음의 3단계로 제한해나갈 수 있다.

1. 로봇의 dynamic config를 고려한 최대 속도와 최대 각속도를 토대로 $$V_{s} = \{(v,\omega) \| v < v_{max}, -\omega_{max} < \omega < \omega_{max}\}$$ 식으로 정의되는 2D velocity search space를 그릴 수 있다.
2. Admmisible velocities - 해당 $$(v,\omega)$$ 쌍으로 만들어지는 curvature가 obstacle에 부딫히지 않는 (또는 부딫히기 전에 멈출 수 있는) 영역으로 제한한다. 그림에서는 장애물에 부딫히게 되는 velocity의 영역을 검게 표시했다.
3. Dynamic window - 로봇의 dynamic config에 따른 구속조건 혹은 안전상의 이유로 최대 가속도와 각가속도를 정의할 수 있는데, 이를 이용하면 현재 velocity를 중심으로 한 직사각형의 window의 영역으로 다음 순간의 velocity를 제한할 수 있다.

각 단계에서 제한되는 로봇의 velocity space를 $$V_{s} , V_{a} , V_{d}$$라고 하면, 결과적으로 우리가 고려해야 하는 search space $$V_{r}$$은 세 영역의 교집합으로 나타낼 수 있다. 그림에서는 직사각형 영역에서 회색을 제외한 흰색 영역만이 고려 대상이 된다.

<fig>
<img src="https://i.imgur.com/GFK0luL.png">
</fig>

$$
V_{r} = V_{s} \cap V_{a} \cap V_{d}
$$

## Optimization

이제 앞에서 구한 Search space의 curvature들을 다양한 critic으로 평가하여 최적의 경로를 하나 선정하는 과정이 남았다. 논문에서는 이를 위한 evaluation function G를 아래처럼 정의한다.

$$
G(v,\omega) = \sigma (\alpha \cdot heading(v,\omega) + \beta \cdot dist(v,\omega) + \gamma \cdot velocity(v,\omega))
$$

세 개의 critic(논문에선 component로 표현하긴 한다)에 parameter이 곱해진 형태로 구성한 함수 G를 $$V_{r}$$ 내의 무작위 점에 대해 계산하여 maximize하는 값을 선택한다.

1. **Target heading $$heading(v,\omega)$$** : 로봇의 heading 방향과 target으로의 방향이 얼마나 일치하는지 평가한다. 두 벡터 사이의 각도가 $$\theta$$일 때 $$180-\theta$$로 계산한다. (Maximize하는 값을 선택하므로) 다만 로봇은 계속 이동하므로 도착할 것으로 예상되는 위치에서 이 값을 계산하는데, 구체적으론 선택된 속도로 다음 time interval 동안 이동한 후 maximum deceleration으로 감속했을 때 지점을 사용한다. 이는 로봇이 장애물을 우회했을 때 부드럽게 방향을 전환하기 위함이라고 한다.
2. **Clearance $$dist(v,\omega)$$** : curvature와 만나게 되는 장애물이 있는 경우 그들 중 가장 가까운 것 까지의 거리를 평가한다. 만약 장애물과 만나지 않는다면 큰 상수값을 사용하기 때문에 장애물을 피하는 경로를 선택하게 하는 critic으로 볼 수 있다.
3. **Velocity $$velocity(v,\omega)$$** : 단순히 로봇의 속도 $$v$$를 평가하는데 같은 경로를 따라갈 경우 빠르게 가는 경로를 선호하도록 하는 역할을 한다.

이 앞에 붙은 가중치들은 [0,1]로 normalize된 각 critic들의 반영 비율을 조절하여 경로를 smoothing해주는 역할을 한다. 예를 들어 heading의 파라미터인 $$\alpha$$가 너무 크면 목적지 방향으로 거의 돌진하다 처음으로 만나게 되는 장애물에 막히게 되고, 너무 작으면 clearance와 속도가 크게 반영되어 아무것도 없는 free space를 맴돌게 된다.

<fig>
<img src="https://i.imgur.com/gaO5fKJ.png">
</fig>

위 그래프는 $$V_{r}$$ 영역에 대해 G를 계산한 예시인데 수직선으로 표시된 점이 discretize된 점들 중 가장 큰 값이므로 로봇은 저 점에 해당하는 $$(v, \omega)$$값을 채택하여 다음 time interval동안 움직일 것이다. 로봇은 이 과정을 매 interval동안 반복하며 새롭게 경로를 결정하면서 target으로 나아가게 된다.

[Nav2의 dwb_controller](https://github.com/ros-planning/navigation2/blob/main/nav2_dwb_controller/README.md)는 논문에서 제시한 3가지 critic 이에도 global planner가 짜준 path를 얼마나 잘 따라가는지 평가하는 PathAlign, goal까지의 거리를 평가하느 GoalDist 등의 플러그인을 제공한다. 이런 방식으로 상황에 맞는 critic을 만들거나 parameter tuning으로 커스텀하여 사용할 수 있는 것이 DWA 방식의 장점인 것 같다.
