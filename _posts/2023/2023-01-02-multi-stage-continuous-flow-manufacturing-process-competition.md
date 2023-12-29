---
title: "[국방 AI 부문 MINI 경진대회] 공정 프로세스 최종 품질값 예측 대회 참가 후기"
tags: ["AI/ML"]
date: 2023-01-02 20:36:15
---

<!-- excerpt -->
<!-- toc -->

<fig>
<img src="https://i.imgur.com/Ua5sYEA.png">
</fig>

Elice 장병 역량강화 플랫폼에서 12/24 ~ 27 4일간 진행된 [[국방 AI 부문 MINI 경진대회] 공정 프로세스 최종 품질값 예측 대회](https://military22.elice.io/courses/33714/info)에 참여해서, 7등으로 장려상을 수상하게 되었다.

<fig>
<img src="https://i.imgur.com/fzv4vA3.jpg">
</fig>

---

# 대회 개요

이 대회는 여러 단계가 있는 연속 생산 공정에서 특정 단계의 품질값을 예측하는 것이 목표다. 공정은 두 개의 Stage로 분리되며, Stage 1과 Stage 2의 작동 방식은 각각 아래와 같다.

1. Stage 1

- Machine 1, 2, 3은 병렬적으로 작업하며, 각 기계의 Output들은 Combiner로 전달된다.
- Combiner의 output은 15개의 특정 위치에서 품질값이 측정된다.

2. Stage 2

- Stage 1의 Output은 Machine 4, 5를 직렬적으로 통과한다.
- Machine 5의 Output 또한 15개의 특정 위치에서 품질값이 측정된다.

시간(time_stamp)에 따른 Machine 1 ~ 5의 상태 데이터와 Stage 1, 2 output의 품질값이 데이터로 주어지며, 대회의 목표는 주어진 시간과 상태 데이터를 통해 30개의 품질값을 예측하는 것이었다. 각 스테이지에서 모델을 하나씩 사용하도록 하고, 각각의 예측값과 실제값의 **R2**를 계산하여 두 값을 평균낸 100점 만점의 성적으로 채점이 이루어졌다.

---

# 코드 리뷰

## 데이터 분석 및 전처리

이번 데이터는 feature의 의미를 알려주지 않아서, 다른 대회와 다르게 어떤 feature를 남겨야 할지 감을 잡기가 불가능했다. 그래서 지시사항에서 삭제하라고 했던 feature만 제거하고 Stage 1, Stage 2의 데이터를 독립적으로 분리시켜 사용하기로 했다.

또한 시간대별로 데이터가 있어서 시계열로 볼 수도 있겠으나, test data 시간대가 train data 시간대의 과거/미래가 아니라, train data 시간대 중간중간의 빈칸이었기 때문에 시계열로 풀지 않았다. 차라리 `time_stamp`에서 연, 월, 일을 버리고 시, 분, 초만 추출해서 `hour`, `min`, `sec` 세 개의 feature들로 분리했다.

```python 시간 데이터 분리
train_data['hour'] = train_data['time_stamp'].str.slice(start=11, stop=13)
train_data['min'] = train_data['time_stamp'].str.slice(start=14, stop=16)
train_data['sec'] = train_data['time_stamp'].str.slice(start=17, stop=19)
train_data = train_data.drop(columns=['time_stamp'])
```

<fig>
<img src="https://i.imgur.com/WqQ8Q0L.png">
</fig>

```py Stage 1, 2 데이터 분리
stage1_data = train_data[train_data.columns.drop(list(train_data.filter(regex='Machine4|Machine5|Stage2')))]
stage2_data = train_data[train_data.columns.drop(list(train_data.filter(regex='Machine1|Machine2|Machine3|Stage1')))]
```

## 모델 학습

사실 AI/ML 대회는 처음이고 예전에 Kaggle 강의 찍먹해본게 다여서, 딱 하나 사용해봤던 `RandomForestRegressor`를 이번에도 가장 처음으로 시도해볼 수 밖에 없었다.

```py RandomForestRegressor를 이용한 Stage 1 모델
from sklearn.ensemble import RandomForestRegressor

model1 = RandomForestRegressor(n_estimators=600,
                              n_jobs=-1,
                              random_state=0)

model1.fit(stage1_train_X, stage1_train_y)

stage1_predictions = model1.predict(stage1_test_X)
```

기대도 안했는데, 첫 점수가 72점이 나오면서 당시 순위로 3등에 올라가는 변수가 생겨버렸다. 뭔가 더 시도해보면 충분히 점수를 더 올릴 수 있을 것 같아서 model1의 output data를 model2의 input feature로 사용해보기도 했는데, 오히려 정확도가 감소한 결과를 얻었다.

이후에 점수가 더 이상 올라가지 않는 한계에 부딫히자 `ExtraTreesRegressor` 모델을 시도해보았다.
엑스트라 트리 또한 `scikit-learn`에서 제공하며, 각 트리를 무작위 특성으로 분할하는 식으로 무작위성을 증가시킨 모델이다.

```py ExtraTreesRegressor를 이용한 Stage 2 모델
from sklearn.ensemble import ExtraTreesRegressor

model2 = ExtraTreesRegressor(
  n_jobs=-1,
  random_state=0,
  max_features='sqrt',
)
```

Stage 1 model은 `RandomForestRegressor`, Stage 2 model은 `ExtraTreesRegressor`를 이용했을 때 가장 결과가 좋았다.
이후 GridSearchCV를 이용하여 각 모델의 최적의 파라미터를 탐색했다.

```py GridSearchCV를 이용한 하이퍼파라미터 튜닝
from sklearn.model_selection import GridSearchCV

grid = {
  'n_estimators' : [600,800,1000],
  'max_depth' : [6,8,10,],
  'min_samples_leaf' : [3,5,7],
  'min_samples_split' : [2,3,5]
}

model2_grid = GridSearchCV(model2, param_grid = grid, scoring="r2", n_jobs=-1, verbose =1)

model2_grid.fit(stage2_train_X, stage2_train_y)

model2_grid.best_params_
```

```py 최적의 hyperparameter로 구성한 Stage 2 모델로 예측값 도출
best_model2 = ExtraTreesRegressor(
  n_estimators=750,
  n_jobs=-1,
  random_state=0,
  max_depth=40,
  min_samples_split=2
)

best_model2.fit(stage2_train_X, stage2_train_y)

stage2_predictions = best_model2.predict(stage2_test_X)
```

최종 결과는 **76.3932**점이었는데, 1등 82.3점과 고작 6점 차이어서 1등의 코드가 더욱 궁금해진다. 그래도 첫 대회치고 정말 운 좋게 좋은 결과를 얻은 것 같아서 아주 만족스러웠다.

## Extra Trees Model

**Random Forest Model**과 그 변종인 **Extra Trees Model**을 이번 기회에 다시 공부해볼 수 있었다.

두 모델은 기본적으로 **결정트리**Decision Tree 기반이지만, 트리를 생성할 feature을 고르는 방식에서 차이점이 있다. 랜덤 포레스트는 트리 하나를 생성할 때 모든 feature에서 임의로 몇 개를 선택하고 그 중 정보획득량을 기준으로 분할을 생성한다. 그러나 고려해야 할 feature의 개수가 많아지면 최적의 분할을 찾는 데 시간을 많이 소모한다.

엑스트라트리는 여기서 분할하는 feature을 무작위로 나누어서, 트리를 훨씬 빠르게 구성한다. 만약 하나의 결정 트리에서 이렇게 한다면 성능이 낮아지겠지만, 빨라진 계산 속도를 바탕으로 더 많은 트리를 앙상블 하기 때문에 Overfitting을 막고 검증 세트의 점수를 높이는 효과가 있다.

두 모델의 하이퍼파라미터는 거의 동일한데, `ExtraTreesRegressor`의 주요 하이퍼파라미터는 다음과 같다.

<br>

- `n_estimators` (default = 100) : 엑스트라 트리 안의 결정 트리 갯수
  결정 트리가 많을수록 더 깔끔한 Decision Boundary가 나오므로 n_estimators는 클수록 좋으나, 과해지면 overfitting의 우려가 있으며 메모리와 훈련 시간이 증가한다.
- `max_depth` : 트리의 최대 깊이
- `min_samples_split` (default = 2) : 노드를 분할하기 위한 최소한의 샘플 데이터 수
- `min_samples_leaf` (default = 1) : 리프노드가 되기 위한 최소한의 샘플 데이터 수
- `max_features`: 무작위로 선택할 Feature의 개수
  `sqrt`이면 `max_features = sqrt(n_features)`
  `log2`이면 `max_features = log2(n_features)`
  `default`값은 `max_features = n_features` - feature 모두에서 비복원 추출로 선택해 결정 트리를 만든다.
  max_features 값이 크면 랜덤 포레스트의 트리들이 매우 비슷해지고, 가장 두드러진 특성에 맞게 예측을 할 것이다.
  max_features 값이 작으면 랜덤 포레스트의 트리들이 서로 상이해져 Overfitting이 줄어들 것이다다.

> [sklearn.ensemble.ExtraTreesRegressor Official Docs](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.ExtraTreesRegressor.html)

---

# 후기

실제 제출했던 코드로 리뷰를 했다면 더 좋았겠지만, 대회 종료 후 컨테이너에 접근이 막혀서 기억에 의존해서 복기하며 리뷰를 했던게 아쉽다. ML 전문가 친구가 추천해준 LightGBM도 사용해보고 싶었으나, 파이썬 역량과 제출 기한의 문제로 다음에 사용해보기로 했다.

> 본 자료는 SSDC-KATUSA ML/DL팀 Weekly Scrum 자료로 사용되었습니다.
