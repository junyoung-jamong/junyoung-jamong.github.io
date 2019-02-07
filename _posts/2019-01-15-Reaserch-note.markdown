# Everyday research note

#### 2019-01-15

* implemented triplet loss function
```
def triplet_loss(x):
    anchor, positive, negative = x

    positive_distance = tf.reduce_sum(tf.square(tf.subtract(anchor, positive)), 1)
    negative_distance = tf.reduce_sum(tf.square(tf.subtract(anchor, negative)), 1)

    single_loss = tf.add(tf.subtract(positive_distance, negative_distance), ALPHA)
    loss = tf.reduce_mean(tf.maximum(single_loss, 0.0), 0)

    return loss
```
Source - [Triplet Loss in Keras](https://codepad.co/snippet/F1uVDD5N)

FastMGTS
--------
2019-02-07 <br/>

> * 예정 실험
>   * FastMGTS와 DTW constraint(5%)의 쿼리 예측 클래스 비교. - FastMGTS는 DTW를 대체 가능한가?
>
>
 
Doublet loss for weighted multi grid similarity
-------------------------------------------------
2019-01-25 <br/>

> * 문제 <br/>
>   * Error rate of Equal weight MGTS <= weighted MGTS
>   * 기대치 - weighted MGTS의 분류 정확도 > Equal weight MGTS 
>
> * 예상원인
>   * normalize의 부재
>   * overfitting - tracking loss & validation acc
>
> * 현재상태로써 사용가능성
>   * Grid selection - weight가 0으로 수렴하는 그리드는 유사도 측정에서 제외할 수 있음.
>   * Grid ordering - weight가 높은 그리드를 먼저 사용.
>
> * Doublet 선택방법
>   * 현재 : 각 batch에 대한 현재 weight를 기반으로 LOOCV(leave-one-out cross validation)을 통해 incorrect일 경우 1등 오답 클래스와 1등 정답 클래스를 각각 nagative와 positive로 사용.
>   * 시도예정 : LOOCV을 통해 incorrect일 경우 1등 정답 클래스보다 쿼리와 가까운 모든 오답 클래스 데이터와의 조합을 사용.
> * GMDTW
>   * warping window size constraint(5%). 
> * ensemble
>   * All STS3
>   * All GMED
>   * All GMDTW
>   * STS3 + GMED + GMDTW


