---
layout: post
title:  "Using TarsosDSP in Android for Pitch Detection"
date:   2019-02-09 12:00:00 +0900
categories: signal processing
---

![](/assets/image/how_to_use_tarsosDSP/img_000.png)

이번 글에서는 안드로이드 환경에서 사운드 데이터의 주요 주파수를 측정하기 위해 TarsosDSP 라이브러리를 사용하는 방법에 대해 다룬다. TarsosDSP를 이용하여 마이크로부터 사운드를 녹음하는 동시에 Pitch Detection을 수행하는 방법과 녹음된 파일을 재생하면서 다시 Pitch Detection을 수행하는 방법을 소개한다.
글에서 다루는 프로젝트 소스코드는 [Github](https://github.com/junyoung-jamong/Chinese-Accent-Comparison/tree/master/FunctionalSampleProject)를  통해 다운받을 수 있다.

TarsosDSP
---------
[TarsosDSP](https://github.com/JorenSix/TarsosDSP)는 오디오 프로세싱을 위한 Java 라이브러리다. 
TarsosDSP는 타악기 히트 감지와 YIN, Mcleod Pitch Method 및 Dynamic Wavelet Algorithm Pitch Tracking 등과 같은  다양한 Pitch Detection 알고리즘들이 구현되어 있다.
또한 Goertzel DTMF 디코딩 알고리즘, WSOLA(Time Stretch Algorithm), 리샘플링, 필터, 간단한 합성, 일부 오디오 효과 및 피치 쉬프트 알고리즘이 포함되어 있다.

Android TarsosDSP
----------------
Android를 위한 TarsosDSP 라이브러리도 제공되고 있다. 
주요 배포판은 javax.sound.xxx에 의존하지 않으며 Android에서도 잘 동작한다. 
Android Studio 프로젝트에 TarsosDSP를 추가하려면 TarsosDSP Android 릴리스를 다운로드해야 한다.
[다운로드 링크](https://0110.be/posts/TarsosDSP_on_Android_-_Audio_Processing_in_Java_on_Android)

![](/assets/image/how_to_use_tarsosDSP/img_00.png)

<https://0110.be/releases/TarsosDSP/TarsosDSP-latest/>

![](/assets/image/how_to_use_tarsosDSP/img_01.png)

다음은 설치한 TarsosDSP-lastest.jar 파일을 안드로이드 스튜디오에서 생성한 프로젝트에 포함하는 과정을 보이고 있다.
lib 폴더에 라이브러리를 추가 하기위해 Project 구조를 변경해준다.

![](/assets/image/how_to_use_tarsosDSP/img_02.png)

다운받은 TarsosDSP-lastest.jar 파일을 libs 폴더에 복사/붙여넣기 해준다.

![](/assets/image/how_to_use_tarsosDSP/img_03.png)

libs 폴더에 있는 TarsosDSP 라이브러리 파일을 프로젝트 Dependency로 등록해준다.

![](/assets/image/how_to_use_tarsosDSP/img_04.png)

![](/assets/image/how_to_use_tarsosDSP/img_05.png)

정상적으로 등록되었다면, build.gradle 파일에 아래와 같이 표시될 것이다.

```
dependencies {
    ...
    implementation files('libs/TarsosDSP-Android-latest.jar')
}
```

프로젝트에서 TarsosDSP 라이브러리를 사용할 준비를 끝냈으니 본격적으로 사용방법을 살펴보겠다.
기본적으로 마이크  기능을 사용할 예정이며 권한 등록의 편의를 위해 targetSdkVersion을 22로 설정해주자.

```
android {
    ...
    defaultConfig {
        ...     
        targetSdkVersion 22
        ...
    }
    ...
}
```

AndroidManifest.xml에도 오비오 및 스토리지 권한을 등록해준다.

```
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

### Activity UI
안드로이드 앱의 GUI는 다음와 같이 구성하였다. 녹음을 위한 버튼과 녹음된 파일을 재생하기 위한 버튼, 녹음되는 음성에 대한 추정 Pitch를 실시간으로 표시하기 위한 텍스트뷰. 

![](/assets/image/how_to_use_tarsosDSP/img_06.png)
 
***activity_record_play.xml***<br/>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/recordButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="녹음" />

    <Button
        android:id="@+id/playButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="재생" />

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="16dp">

        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Pitch:" />

        <TextView
            android:id="@+id/pitchTextView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="0"
            android:textSize="50sp" />

    </FrameLayout>

</LinearLayout>
```

### TarsosDSP Format settings
채널 수, 샘플 속도, 샘플 크기, 바이트 순서, 프레임 속도 및 프레임 크기 등과 오디오 인코딩 형식 등을 설정하기 위해 TarsosDSPAudioFormat 클래스 객체를 생성한다.

> **Class References**
> * [TarsosDSPAudioFormat](https://0110.be/releases/TarsosDSP/TarsosDSP-latest/TarsosDSP-latest-Documentation/be/tarsos/dsp/io/TarsosDSPAudioFormat.html)

```
public TarsosDSPAudioFormat(TarsosDSPAudioFormat.Encoding encoding,
                            float sampleRate,
                            int sampleSizeInBits,
                            int channels,
                            int frameSize,
                            float frameRate,
                            boolean bigEndian)
```

> **Constructor parameters** <br/>
> * encoding - the audio encoding technique
> * sampleRate - the number of samples per second
> * sampleSizeInBits - the number of bits in each sample
> * channels - the number of channels (1 for mono, 2 for stereo, and so on)
> * frameSize - the number of bytes in each frame
> * frameRate - the number of frames per second
> * bigEndian - indicates whether the data for a single sample is stored in big-endian byte order (false means little-endian)

> **Encording Technique**<br/>
> * TarsosDSPAudioFormat.Encoding.ALAW - Specifies a-law encoded data.
> * TarsosDSPAudioFormat.Encoding.PCM_SIGNED - Specifies signed, linear PCM data.
> * TarsosDSPAudioFormat.Encoding.PCM_UNSIGNED - Specifies unsigned, linear PCM data.
> * TarsosDSPAudioFormat.Encoding.ULAW - Specifies u-law encoded data.


```
TarsosDSPAudioFormat tarsosDSPAudioFormat;

protected void onCreate(Bundle savedInstanceState)
{
    ...
    
    tarsosDSPAudioFormat=new TarsosDSPAudioFormat(TarsosDSPAudioFormat.Encoding.PCM_SIGNED,
                22050,
                2 * 8,
                1,
                2 * 1,
                22050,
                ByteOrder.BIG_ENDIAN.equals(ByteOrder.nativeOrder()));
    
    ...
}

```

### Voice Recoding & Pitch Detection

recoding() 함수에서는 TarsosDSP를 이용하여 마이크를 통해 음성을 녹음하는 동시에 주요 주파수를 측정한다. TarsosDSP 라이브러리는 하나의 Dispatch 객체에 하나 이상의 AudioProcessor 객체를 추가하여 사용하는 패턴을 갖는다.

AudioDispatcher 클래스는 파일을 재생하고 등록된 AudioProcessor에 플로트 배열을 전달한다. 이 클래스는 FFT, 피치 감지기, 오디오 플레이어 등을 공급하는 데 사용할 수 있다.

AudioDispatcherFactory 클래스는 마이크, PCM wav 파일 또는 하위 프로세스에서 파이프 된 PCM 샘플 등 다양한 소스로부터 AudioDispatcher 객체를 만든다.

AudioProcessors 클래스는 실제 디지털 신호 처리를 담당한다. AudioEvent 객체에서 작동하는 프로세스 메서드를 인터페이스로써 사용한다. AudioEvent에는 일부의 float 및 raw 바이트로 같은 정보를 포함한 버퍼가 포함되어 있다.
AudioProcessors 클래스는 체인화되어 있다. 예를 들어, 효과를 실행한 다음 사운드를 재생한다. 오디오 프로세서 체인은 프로세스 메서드에서 false를 반환하여 중단할 수 있다.

> **Class References**
> * [AudioDispatcher](https://0110.be/releases/TarsosDSP/TarsosDSP-latest/TarsosDSP-latest-Documentation/be/tarsos/dsp/AudioDispatcher.html)
> * [AudioDispatcherFactory](https://0110.be/releases/TarsosDSP/TarsosDSP-latest/TarsosDSP-latest-Documentation/be/tarsos/dsp/io/jvm/AudioDispatcherFactory.html)
> * [AudioProcessor](https://0110.be/releases/TarsosDSP/TarsosDSP-latest/TarsosDSP-latest-Documentation/be/tarsos/dsp/AudioProcessor.html)

녹음/재생될 파일의 경로
```
String filename = "recorded_sound.wav";
File sdCard = Environment.getExternalStorageDirectory();
file = new File(sdCard, filename);
```


AudioDispatcherFactory 클래스를 이용해 마이크를 소스로하는 dispatcher 객체를 생성한다.

> fromDefaultMicrophone(int sampleRate, int audioBufferSize, int bufferOverlap)

```
dispatcher = AudioDispatcherFactory.fromDefaultMicrophone(22050,1024,0);
```

지정한 출력으로 진행중인 사운드를 기록하는 AudioProcessor인 WriterProcessor 객체를 생성하여 dispatcher에 추가한다.

> WriterProcessor(TarsosDSPAudioFormat audioFormat, java.io.RandomAccessFile output) 

```
RandomAccessFile randomAccessFile = new RandomAccessFile(file,"rw");
AudioProcessor recordProcessor = new WriterProcessor(tarsosDSPAudioFormat, randomAccessFile);
dispatcher.addAudioProcessor(recordProcessor);
```

Pitch Detection은 PitchProcessor 클래스를 통해 수행할 수 있다. 이 때, 실시간 Detection 추정 결과를 전달 받기 위한 쓰레드 핸들러 객체를 지정해줘야 한다.

> PitchProcessor(PitchProcessor.PitchEstimationAlgorithm algorithm, float sampleRate, int bufferSize, PitchDetectionHandler handler)

> **PitchEstimationAlgorithm** <br/>
> * AMDF - A pitch extractor that extracts the Average Magnitude Difference (AMDF) from an audio buffer.
> * DYNAMIC_WAVELET - An implementation of a dynamic wavelet pitch detection algorithm (See DynamicWavelet), described in a paper by Eric Larson and Ross Maddox "Real-Time Time-Domain Pitch Tracking Using Wavelets
> * FFT_PITCH - Returns the frequency of the FFT-bin with most energy.
> * FFT_YIN - A YIN implementation with a faster FastYin for the implementation.
> * MPM - McLeodPitchMethod.
> * YIN - YIN algorithm.

```
PitchDetectionHandler pitchDetectionHandler = new PitchDetectionHandler() {
    @Override
    public void handlePitch(PitchDetectionResult res, AudioEvent e){
        final float pitchInHz = res.getPitch();
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                pitchTextView.setText(pitchInHz + "");
            }
        });
    }
};

AudioProcessor pitchProcessor = new PitchProcessor(PitchProcessor.PitchEstimationAlgorithm.FFT_YIN, 22050, 1024, pitchDetectionHandler);
dispatcher.addAudioProcessor(pitchProcessor);
```

Thread로 dispatcher 실행해준다.
```
Thread audioThread = new Thread(dispatcher, "Audio Thread");
audioThread.start();
```

**녹음 중지**는 dispatcher의 stop()함수를 호출해주면 된다.
```
public void stopRecording()
{
    releaseDispatcher();
}

public void releaseDispatcher()
{
    if(dispatcher != null)
    {
        if(!dispatcher.isStopped())
            dispatcher.stop();
        dispatcher = null;
    }
}
```

### Play Sound File with Pitch Detection

playAudio() 함수에서는 TarsosDSP를 이용하여 저장된 사운드 파일을 실행하는 동시에 실시간 Pitch Detection을 수행한다.

마이크가 아닌 파일을 소스로하는 Dispatcher를 생성하기 위해서 AudioDispatcher 객체 생성 시 UniversalAudioInputStream 객체를 인자로 전달한다.

> UniversalAudioInputStream(java.io.InputStream underlyingInputStream, TarsosDSPAudioFormat format) 


```
FileInputStream fileInputStream = new FileInputStream(file);
dispatcher = new AudioDispatcher(new UniversalAudioInputStream(fileInputStream, tarsosDSPAudioFormat), 1024, 0);
```

AndroidAudioPlayer 클래스는 오디오 플레이를 위한 AudioProcessor이다.

> AndroidAudioPlayer(TarsosDSPAudioFormat audioFormat, int bufferSizeInSamples, int streamType)

```
AudioProcessor playerProcessor = new AndroidAudioPlayer(tarsosDSPAudioFormat, 2048, 0);
dispatcher.addAudioProcessor(playerProcessor);
```

Pitch Detection은 Recording 시와 동일하다.
```
PitchDetectionHandler pitchDetectionHandler = new PitchDetectionHandler() {
    @Override
    public void handlePitch(PitchDetectionResult res, AudioEvent e){
        final float pitchInHz = res.getPitch();
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                pitchTextView.setText(pitchInHz + "");
            }
        });
    }
};

AudioProcessor pitchProcessor = new PitchProcessor(PitchProcessor.PitchEstimationAlgorithm.FFT_YIN, 22050, 1024, pitchDetectionHandler);
dispatcher.addAudioProcessor(pitchProcessor);
```

```
Thread audioThread = new Thread(dispatcher, "Audio Thread");
audioThread.start();
```

**RecordPlayActivity.java** 전체 소스코드
```
package com.smartjackwp.junyoung.functionalsampleproject;

import ...

public class RecordPlayActivity extends AppCompatActivity {
    AudioDispatcher dispatcher;
    TarsosDSPAudioFormat tarsosDSPAudioFormat;

    File file;

    TextView pitchTextView;
    Button recordButton;
    Button playButton;

    boolean isRecording = false;
    String filename = "recorded_sound.wav";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_record_play);

        File sdCard = Environment.getExternalStorageDirectory();
        file = new File(sdCard, filename);

        /*
        filePath = file.getAbsolutePath();
        Log.e("MainActivity", "저장 파일 경로 :" + filePath); // 저장 파일 경로 : /storage/emulated/0/recorded.mp4
        */

        tarsosDSPAudioFormat=new TarsosDSPAudioFormat(TarsosDSPAudioFormat.Encoding.PCM_SIGNED,
                22050,
                2 * 8,
                1,
                2 * 1,
                22050,
                ByteOrder.BIG_ENDIAN.equals(ByteOrder.nativeOrder()));

        pitchTextView = findViewById(R.id.pitchTextView);
        recordButton = findViewById(R.id.recordButton);
        playButton = findViewById(R.id.playButton);

        recordButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(!isRecording)
                {
                    recordAudio();
                    isRecording = true;
                    recordButton.setText("중지");
                }
                else
                {
                    stopRecording();
                    isRecording = false;
                    recordButton.setText("녹음");
                }
            }
        });

        playButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                playAudio();
            }
        });
    }

    public void playAudio()
    {
        try{
            releaseDispatcher();

            FileInputStream fileInputStream = new FileInputStream(file);
            dispatcher = new AudioDispatcher(new UniversalAudioInputStream(fileInputStream, tarsosDSPAudioFormat), 1024, 0);

            AudioProcessor playerProcessor = new AndroidAudioPlayer(tarsosDSPAudioFormat, 2048, 0);
            dispatcher.addAudioProcessor(playerProcessor);

            PitchDetectionHandler pitchDetectionHandler = new PitchDetectionHandler() {
                @Override
                public void handlePitch(PitchDetectionResult res, AudioEvent e){
                    final float pitchInHz = res.getPitch();
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            pitchTextView.setText(pitchInHz + "");
                        }
                    });
                }
            };

            AudioProcessor pitchProcessor = new PitchProcessor(PitchProcessor.PitchEstimationAlgorithm.FFT_YIN, 22050, 1024, pitchDetectionHandler);
            dispatcher.addAudioProcessor(pitchProcessor);

            Thread audioThread = new Thread(dispatcher, "Audio Thread");
            audioThread.start();

        }catch(Exception e)
        {
            e.printStackTrace();
        }
    }

    public void recordAudio()
    {
        releaseDispatcher();
        dispatcher = AudioDispatcherFactory.fromDefaultMicrophone(22050,1024,0);

        try {
            RandomAccessFile randomAccessFile = new RandomAccessFile(file,"rw");
            AudioProcessor recordProcessor = new WriterProcessor(tarsosDSPAudioFormat, randomAccessFile);
            dispatcher.addAudioProcessor(recordProcessor);

            PitchDetectionHandler pitchDetectionHandler = new PitchDetectionHandler() {
                @Override
                public void handlePitch(PitchDetectionResult res, AudioEvent e){
                    final float pitchInHz = res.getPitch();
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            pitchTextView.setText(pitchInHz + "");
                        }
                    });
                }
            };

            AudioProcessor pitchProcessor = new PitchProcessor(PitchProcessor.PitchEstimationAlgorithm.FFT_YIN, 22050, 1024, pitchDetectionHandler);
            dispatcher.addAudioProcessor(pitchProcessor);

            Thread audioThread = new Thread(dispatcher, "Audio Thread");
            audioThread.start();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void stopRecording()
    {
        releaseDispatcher();
    }

    public void releaseDispatcher()
    {
        if(dispatcher != null)
        {
            if(!dispatcher.isStopped())
                dispatcher.stop();
            dispatcher = null;
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        releaseDispatcher();
    }
}

```
