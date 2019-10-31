# Mindwave
### Member
- 박규태, 문현기, 박은하, 목하은 

## Introduction

"Sleep Bebe"는 뇌파를 분석하여 아기의 수면 정도에 따른 알람을 해주는 시스템으로, 아기와 떨어져 있는 공간에서도 스마트폰앱과 스마트 전구 휴(Hue)로 아기가 잠에서 깼는 지 알려준다. 아기의 수면 정도에 따라 휴의 색이 바뀌고, 앱에서는 전구모형의 그림의 색이 바뀐다. 예를 들어, 아이의 잠이 깬 상태에서는 전구가 빨간색으로 빛나고 휴대폰에 알람이 온다. 아기의 수면도를 측정하고 언제 깰 지 예측할 뿐만 아니라, 아이의 수면도에 따른 통계를 추가함으로써 아기에게 관심이 많은 부모들에게 관련 정보를 제공하고자 하였다. 

<center><img src="/baby.jpg"></center>

## Planning
### 1. Goal: 3R
- Rest for mother: 아기가 잘 때, 부모님들이 휴식을 취하거나 잠시 다른 일을 할 수 있도록 함
- Relief of child: 아기가 깼을 때, 눈을 뜨면 부모님을 보며 안정감을 찾을 수 있도록 함
- Replace sound to light: 청각 장애인 부모님들에게 아기의 우는 소리를 시각적으로 알려줌
### 2. storyboard
![Human-Computer-Interaction-Mindwave](Storyboard.pdf)


### 2. 구성요소: 앱, 휴, EEG
EEG와 앱, 휴를 블루투스로 연동하여 EEG로 측정된 값을 스마트폰앱과 휴로 전송하는 방식으로 작동된다. 

#### 2-1 앱: 
자는 아기와 거리상 떨어져 있더라도 아기의 수면상태를 파악할 수 있도록 앱을 통해 알려준다. 본 프로젝트에서는 XD프로그램을 통해 앱의 프로토타입을 제작하였으나, planning단계에서는 실제 데이터를 볼 수 있는 앱 사용을 고려하였다. 앱의 용도는 컴퓨터 연결로 대체하였다. 
#### 2-2 휴: 
아이의 소리를 들을 수 없는 청각 장애인들도 아기가 깼다는 것을 인지할 수 있도록 스마트 전구를 사용하여 알려 줄예정이다. 
#### 2-3 EEG: 
EEG 측정은 본 project에서 mindwave mobile 기기를 사용하여 실험하였으나, 제품 planning 단계에서는 무선 EEG기기 사용을 고려하였다.  
## Prototype
### 1. paper prototype
![Human-Computer-Interaction-Mindwave](paper prototype.pdf)
### 2. low-prototype (APP)
1차 프로토타입은 User research를 바탕으로 제작되었다.

**User research** 
<center><img src="/concept feedback.png"></center>

**1차 프로토 타입**
[Human-Computer-Interaction-Mindwave](https://xd.adobe.com/view/e8a4776e-a197-4c6b-4b89-7b2da3243a68-9e95/)
<center><img src="/entire prototype.png"></center>

아래는 Usability Test의 feedback으로 이를 바탕으로 2차 프로토 타입을 제작하였다. 

**Usability test**
<center><img src="/prototype weakness.png"></center>

**2차 프로토타입**
[Human-Computer-Interaction-Mindwave](https://xd.adobe.com/view/e8a4776e-a197-4c6b-4b89-7b2da3243a68-9e95/)
<center><img src="/entire prototype2.png"></center>


### 3. setting
#### 1. Windows 10에 Pybluez 설치
1. Anaconda 설치 (Python 3.7)
2. 아래 사이트를 참고하여 PyBluez 설치: https://github.com/pybluez/pybluez/issues/180#issuecomment-448102727
	1) Download and run "Visual Studio Installer": https://www.visualstudio.com/pl/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15
	2) Install "Visual Studio Build Tools 2017", check "Visual C++ build tools" and "Universal Windows Platform build tools"
	3) git clone https://github.com/pybluez/pybluez
	4) cd pybluez
	5) python setup.py install

### 4. high-prototype(code)
