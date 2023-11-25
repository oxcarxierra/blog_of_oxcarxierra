---
title: "AVDL 인턴 : CFD로 Airfoil의 Lift, Drag 계산"
tags: [Engineering]
date: 2023-06-18 22:49:36
---

<!--excerpt -->
<!-- toc -->

20년도 2학기에 유체역학을 수강하고 전산유체 분야에 흥미를 느껴서, 무작정 항공우주공학부 이관중 교수님께 항공우주 비행체 설계 연구실(AVDL)에 학부생 인턴으로 참여하여 CFD에 대해 더 공부해보고 싶다는 메일을 드렸다. 유체에 겨우 입문한 상태였지만 감사하게도 홍윤표 박사과정 연구원님께서 지도를 맡아주셨고, 덕분에 겨울방학동안 개략적인 CFD 이론을 배워보고 CFD에서 자주 이용되는 툴인 Pointwise 및 Ansys Fluent를 익히는 좋은 기회가 되었다.

---

# CFD 공부

사수님의 도움을 받아 Versteeg의 <유한체적법을 이용한 전산유체역학> 교재를 실습에 필요할 정도만 독학해봤다. 이해하기 어려운 수준은 아니었으나 수치해석쪽 지식이 없었다면 난감했을 것 같다. 시간이 좀 지나고 포스팅을 하는 중이라 기억이 많이 나지는 않지만, 다양한 CFD의 방법론을 위주로 공부해봤고 그 중 후속 실습에 이용할 **SST k-w method**를 집중적으로 읽었던 것 같다.

# NACA 0012 Airfoil의 받음각에 따른 _Cl_, _Cd_ 계산 실습

대표적인 에어포일인 NACA0012 형상의 받음각에 따른 *Cl*와 *Cd*를 연산하고 참값과 비교해보는 프로젝트를 진행해보았다. 실험을 통해 구해진 참값은 NASA에서도 공식적으로 제공하고 있다.

## Generating mesh grid around the airfoil using Pointwise

연산에 앞서 에어포일 주변의 공간을 분할하는 Grid를 짜야 한다. 이 과정에서는 대표적인 격자 생성 프로그램인 **Pointwise**를 이용하였다.
우선 Airfoil의 단면이 포함된 2D 평면을 분할한 후, z축으로 그대로 쌓아서 공간으로 확장하였다. 이 떄 2D 평면을 분할하는 방법으로는 아래 그림처럼 1) C Grid 2) H Grid 3) O Grid 3가지가 있다. Leading Edge에서는 O Grid, Trailing Edge 후방으로는 H Grid를 채택하여 혼합한 것이 C Grid라고 볼 수 있다. 본 실습에서는 **C-Grid**와 **O-Grid** 두 가지만 시도해보았다.
이 때 Airfoil에 가까운 공간일수록 마찰의 효과를 많이 받으므로 격자가 더 촘촘하게 짜여지도록 만들어야 한다. 격자의 크기가 작을수록 정밀한 연산이 가능하지만, 당연하게도 연산에 걸리는 시간은 길어진다.

<fig>
<img src="http://i.imgur.com/V1VsSka.png">
<figcaption>우상단 좌상단 C-grid, 좌하단 H Grid, 우하단 O Grid</figcaption>
</fig>

## Computing with Ansys Fluent

연산을 위한 프로그램으로는 Ansys Fluent를 이용하였다. Pointwise로 만든 mesh 파일을 CAE(Computer Aided Engineering) 메뉴를 이용하여 export하면, Fluent 프로그램에서 import해올 수 있다.

연산을 돌리면 mesh내 각 grid의 압력을 계산할 수 있으며, 이를 통해 전체 Airfoil에 가해지는 Lift와 Drag이 도출된다. 설정해준 밀도와 유속을 이용하면 무차원 상수 Cl과 Cd를 얻어낸다. 이 방식으로 본 실습에서는 1°부터 19°까지 2° 간격으로 받음각을 증가시키면서 Cl과 Cd를 각각 계산했다. 이 때 에어포일 자체를 회전시키는 것이 아니라 유동의 방향이 받음각만큼 아래로 설정해주어야 했기에, 구해진 Lift와 Drag에 삼각함수를 적절히 곱해서 후처리하는 과정이 필요했다.

대부분의 CFD에 필요한 연산량은 PC로는 감당하지 못하는 정도라서, 아마 AVDL 연구실에 할당된 원격 CPU에서 연산을 한 것 같았다. (관련 설정은 사수님이 도와주셔서 확실하지 않다) 성능이 좋을 터임에도 불구하고 하나의 받음각에 대해 연산을 돌리고 iteration이 어느 정도 없어지기까지 대략 30분의 시간이 걸렸던 것 같다. Grid 수가 많았다면 더 걸렸을 것이다.

결과 데이터로 pressure gradient나 streamline을 그리는 등 Postprocessing 과정도 수행할 수 있다.

<fig>
<img src="http://imgur.com/JWgH5c6.png">
<figcaption>Pressure gradient and streamlines around NACA0012 airfoil at AoA = 19°</figcaption>
</fig>

<fig>
<img src="http://i.imgur.com/a4MB2t6.jpg">
<figcaption>aoa - Cl graph of NACA0012</figcaption>
</fig>

받음각이 13° 미만인 경우에는 reference data와 거의 일치했지만, 15° 이상의 받음각 상황에서는 경향성을 벗어나는 연산 결과를 얻었다. 처음에는 연산 과정에서의 오류거나 충분한 iteration을 하지 않은 줄 알았으나, 사실 그 이유는 다른 곳에 있었다.
애초에 steady flow를 가정하고 풀고 있었으므로 13° 미만의 받음각 상황에서는 문제가 없지만 unsteady flow가 생기는 상황에서는 초기 조건부터 잘못되었던 것이다. 새롭게 설정하여 다시 연산해야 했는데, 아쉽게도 그 부분은 진행하지 못했다.
