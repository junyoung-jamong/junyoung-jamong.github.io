---
layout: post
title:  "imgaug를 이용하여 바운딩박스 정보를 포함한 이미지 증폭시키기"
date:   2019-01-23 11:45:44 +0900
categories: Machine Learning
---

Object Detection 문제를 포함하여 딥러닝을 이용한 영상인식 접근 방법은 많은 수의 학습 이미지가 요구된다. 직접적으로 최대한 확보할 수 있는 이미지의 수가 적절한 모델을 학습하기에 충분하지 않을 경우에는 원본 이미지에 약간씩의 변형을 줌으로써 새 이미지들을 생성하는 **image augmentation** 방법을 고려할 수 있다. 

Object Detection 문제에서는 보유한 학습 이미지 데이터에 대해 Supervised learning을 위한 bounding-box labeling 작업이 포함되어 있다. 100장의 원본 이미지가 있을 때, 100개의 Annotation(bounding-box 정보를 포함한 정답 label)을 만들어야 하는데 Annotation 생성은 수작업으로 이뤄진다. 그렇다면, 만약 image augmentation을 통해 100장의 이미지를 10000장으로 증가시켰다면, 새로 생성된 9900장에 대해서 다시 Anntation을 만들어야 하는 것일까? 

물론 이미지 수를 늘릴 때, 원본 이미지의 크기나, 방향은 유지하면서 색상 필터나 노이즈를 추가하는 경우에는 큰 문제가 되지 않는다. 그러나 image augmentation에는 이미지의 회전, 반전, 사이즈 축소 및 확대 등도 포함되어 있다. 이에 따라 
이미지를 증폭시키면 변형된 이미지에 새로 Annotation labeling 작업을 수행해야 하는가 걱정이 앞선다. 

하지만 다행이도, 다행이도 이를 위한 도구가 제공되고 있다.

Python 라이브러리 **imgaug**를 사용하면 Annotation을 포함한 이미지 증폭을 수행할 수 있다. 예를 들어, 이미지를 증가시키는 동안 이미지가 회전하면 라이브러리는 그에 따라 모든 객체의 Bounding-box를 함께 회전 시킨다.

![imgaug](/assets/image/how_to_use_imgaug/img1.png)

* Image augmentation for machine learning -  [imgaug](https://github.com/aleju/imgaug) 

imgaug를 이용하면 여러 학습 이미지를 새롭게 변경된 새 이미지 세트로 변환할 뿐만 아니라 bounding-box 정보, land mark 정보, segmentation map 정보 등을 새로 변환된 이미지에 적용된 상태로 유지할 수 있다.

## imgaug installation

```
pip install six numpy scipy Pillow matplotlib scikit-image opencv-python imageio Shapely
```

imgaug를 위한 사전 모듈들을 설치하는 과정에서 Shapely 모듈이 정상적으로 설치되지 않았다.

![](/assets/image/how_to_use_imgaug/img2.png)

현재 pip로 Shapley가 정상 설치 되지 않았기에 직접 whl 파일을 다운받고 설치해줄 필요가 있다.

[whl 파일 다운로드](https://www.lfd.uci.edu/~gohlke/pythonlibs/#shapely)

![](/assets/image/how_to_use_imgaug/img3.png)

다운로드 받을 때 자신의 Python 버전에 맞춰서 받아야 한다. cp36은 Python 3.6 버전을 의미한다. 

![](/assets/image/how_to_use_imgaug/img4.png)

다운로드 받았으면, 받은 경로로 이동하여 whl 파일을 실행한다.


```
python -m pip install Shapely-1.6.4.post1-cp36-cp36m-win_amd64.whl
```

![](/assets/image/how_to_use_imgaug/img5.png)

**imgaug 모듈 설치하기**

```
pip install imgaug
```

![](/assets/image/how_to_use_imgaug/img6.png)

imgaug가 정상적으로 설치되었다.

## Bounding-box image augmentation

[Example: Bounding Boxes](https://imgaug.readthedocs.io/en/latest/source/examples_bounding_boxes.html)

###Bounding-box에 대한 imgaug 라이브러리의 지원 기능은 다음과 같다:
> * Bounding-boxes를 객체로 표현한다 (*imgaug*.BoundingBox). <br/>
> * Bounding-boxes를 이미지와 함께 증가시킨다. <br/>
> * 이미지에 Bounding-box를 그린다. <br/>
> * 이미지에 경계 상자 이동 / 이동, 다른 이미지 (예 : 크기 조정 후 동일한 이미지)로 투영하고, 교집합/합집합 및 IoU 값을 계산한다.

