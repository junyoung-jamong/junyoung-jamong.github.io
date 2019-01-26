---
layout: post
title:  "Android에서 내가 학습한 YOLO 모델을 이용해 Object Detection 구현하기"
date:   2019-01-25 12:00:00 +0900
categories: Machine Learning
---

TensorFlow를 기반으로 학습한 모델은 가중치 정보를 포함하는 파일로 변환하여 다양한 플랫폼에 적용할 수 있다. 
Darkflow 역시 TensorFlow를 기반으로하기 때문에 Darkflow를 이용해 학습한 모델을 여러 응용 프로그램에 적용하는 것은 어렵지 않다.

이번엔 Darkflow를 통해 학습한 나만의 YOLO 모델을 Android 앱에 적용하는 방법을 소개한다.

Android에서는 TensorFlow Lite 라는 이름으로 TensorFlow Library가 제공되고 있다.
tensorflow github에서는 관련 정보와 함께 다양한 플랫폼에서의 tensorflow 샘플 프로젝트를 제공하고 있다.

## TensorFlow in Android

> * Github - [TensorFlow github](https://github.com/tensorflow/tensorflow) <br/>
> * TensorFlow lite - [TensorFlow lite](https://www.tensorflow.org/lite/overview)<br/>

![](/assets/image/how_to_customize_yolo_on_android/img4.png)

Android 앱에 YOLO모델을 적용하기 위해 TensorFlow 팀에서 제공해주는 데모 프로그램을 기반으로 사용할 것이다. 
따라서 해당 repository를 clone 하기로 하자.

```
git clone https://github.com/tensorflow/tensorflow
```

tensorflow/tensorflow/examples/android에 안드로이드 프로젝트가 존재한다. 이를 Android Studio를 이용하여 실행해주면 된다.

![](/assets/image/how_to_customize_yolo_on_android/img5.png)

처음 프로젝트를 불러오면 Gradle, SDK 등을 업데이트하거나 설정하라는 안내가 표시된다. 요구사항을 설치/설정 해주고나서 가장 먼저 할 일은 **bulid.gradle** 파일의 nativeBuildSystem 변수 값을 'none'으로 변경하는 것이다.

```
def nativeBuildSystem = 'none'
```

![](/assets/image/how_to_customize_yolo_on_android/img2.png)

해당 데모 앱에는 ClassifierActivity, StylizeActivity, SpeechActivity, DetectorActivity 총 4가지 기능들이 각각의 Activity로 구성되어 있다.
이 중에서 DetectorActivity에 관심이 있기 때문에 **AndroidManifest.xml**에서 DetectorActivity를 제외한 나머지는 비활성화 해준다.

```
<application android:allowBackup="true"
    android:debuggable="true"
    android:label="@string/app_name"
    android:icon="@drawable/ic_launcher"
    android:theme="@style/MaterialTheme">

    <activity android:name="org.tensorflow.demo.DetectorActivity"
        android:screenOrientation="portrait"
        android:label="@string/activity_name_detection">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>

    <!--
    <activity android:name="org.tensorflow.demo.StylizeActivity"
        android:screenOrientation="portrait"
        android:label="@string/activity_name_stylize">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>

    <activity android:name="org.tensorflow.demo.ClassifierActivity"
              android:screenOrientation="portrait"
              android:label="@string/activity_name_classification">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>

    <activity android:name="org.tensorflow.demo.SpeechActivity"
        android:screenOrientation="portrait"
        android:label="@string/activity_name_speech">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>
    -->
</application>
```

아무런 변경 없이 프로그램을 안드로이드 기기에서 실행해보면 아래와 같이 기존에 미리 설정된 Object Detecting(TF Detect) 프로그램이 정상적으로 동작하는 것을 확인할 수 있다. 

![](/assets/image/how_to_customize_yolo_on_android/img1.png)

## Android Object Detection customizing
소스코드를 조금 변경하여 자신의 Object Detecting 문제로 어플을 커스터마이징 해보자.


우선, [Darkflow를 활용하여 YOLO 모델로 이미지 디텍션 구현(윈도우 환경)](https://junyoung-jamong.github.io/deep/learning/2019-01-22-Darkflow를-활용해-YOLO모델-이미지-디텍션-구현-in-windows.html)을 통해 
기존에 학습된 YOLO 모델 및 가중치 정보를 어플에 적용하기 위해 darkflow/ckpt 내의 meta(weight 정보)파일과 모델의 cfg 파일을 이용해 * .pb 파일을 만들어야 한다.

다음 명령어를 통해 최종적으로 학습된 weight를 * .pb로 저장할 수 있다. 
```
python flow --model cfg/test-tiny-yolo-4c.cfg --load -1 --savepb
```
![](/assets/image/how_to_customize_yolo_on_android/img3.png)

built_graph 폴더 내에 생성된 pb 파일을 안드로이드 프로젝트의 tensorflow\tensorflow\examples\android\assets 경로에 복사한다.

```
tensorflow\tensorflow\examples\android\assets
```

이제 프로젝트 내 두 개의 java 파일을 수정하기만 하면 된다.
<br/>
<br/>

* DetectorActivity.java


```
private static final String YOLO_MODEL_FILE = "file:///android_asset/test-tiny-yolo-4c.pb";
```

```
private static final DetectorMode MODE = DetectorMode.YOLO;
```

![](/assets/image/how_to_customize_yolo_on_android/img6.png)

<br/>
<br/>

* TensorFlowYoloDetector.java
```
private static final int NUM_CLASSES = 4;
```

```
private static final String[] LABELS = {
    "Manufacturer",
    "ProductNumber",
    "ProductName",
    "CasNumber"
};
```

![](/assets/image/how_to_customize_yolo_on_android/img7.png)

다시 안드로이드 기기에 프로그램을 설치하여 실행해보면 자신의 모델이 적용된 앱이 동작하는 것을 확인할 수 있다.

실행 결과:

![](/assets/image/how_to_customize_yolo_on_android/gif1.gif)
