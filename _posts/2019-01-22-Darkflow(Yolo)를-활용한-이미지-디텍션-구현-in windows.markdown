---
layout: post
title:  "Darkflow를 활용하여 Yolo 모델로 이미지 디텍션 구현(윈도우 환경)"
date:   2019-01-22 14:47:44 +0900
categories: Deep learning
---

#### 부제


* labelImg - 이미지 labeling을 위한 도구

* 학습을 위한 darkflow 준비
```
git clone https://github.com/thtrieu/darkflow.git

cd darkflow

python setup.py build_ext --inplace

pip install .
```


* training
```
python flow --model ./cfg/my-tiny-yolo.cfg --labels ./labels.txt --trainer adam --dataset ../data/dataset/ --annotation ../data/annotations/ --train --summary ./logs --batch 5 --epoch 100 --save 50 --keep 5 --lr 1e-04 --gpu 0.5
```


* prediction(detection)
```
python flow --imgdir ../data/testset/ --model ./cfg/my-tiny-yolo.cfg --load -1 --batch 1 --threshold 0.5
```


learning rate : 1e-03 

1430 - 0.5 treshold

![Threshold_0.50](../images/how_to_use_darkflow/29.png)

1430 - 0.03 treshold

![Threshold_0.03](../images/how_to_use_darkflow/30.png)

* No box after train problem
https://github.com/thtrieu/darkflow/issues/80#issuecomment-303931206

[학습팁]

처음 학습에는 learning rate를 크게 설정할 것. 