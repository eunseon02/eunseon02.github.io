---
layout: post
title: Droid-SLAM failure case
categories: [slam, 3d-mapping]
tags: [droid-slam, ad-mapping, slam, deep-learning-based-slam]
fullview: true
comments: Droid-SLAM을 직접 실행시켜보며 확인해볼 수 있었던 tracking failure case를 정리하였다.
---

먼저 대표적으로 visual SLAM에서 tracking을 실패하는 요인은 다음과 같은 것들이 존재한다. 
“textureless, large scale, fast motion, reflection, Dynamic Objects, Repetitive Patterns, Occlusions, Rapid Changes in Illumination:”

실제로 Droid-SLAM을 돌려보면서 느낄 수 있었던 tracking failure case는 아래의 두 가지가 있었다.

## case 1. fast motion
이 case에서 tracking 실패는 Droid-SLAM이 이웃 프레임들과의 시각적 특징의 correlation을 이용해서 mapping & tracking 진행하기 때문에 시각적인 연결성에 크게 의존하기 때문이라고 생각할 수 있었다.

## case 2. Repetitive Patterns
Droid-SLAM은 optical flow를 예측하는 딥러닝 모델인 RAFT를 기반으로 만들어졌는데 optical flow는 연속적인 프레임 상에서 픽셀의 움직임을 추정하는 것이다. 따라서 이 case의 경우 연속적인 프레임 동안에 유사한 시각적 특징의 등장으로 인해 data association의 오류를 유발했다고 생각할 수 있다.


### 
또한 Droid-SLAM의 경우에는 루프클로징이 진행되지 못하였는데 이는 Droid-SLAM의 Front-end 에서 Factor graph를 관리하는 방식으로 인한 것으로 예상된다. 

과거의 몇 프레임만 고려하여 edge를 추가하고 timestamp가 오래된 edge는 제거하는 방식을 이용하며 프레임의 포즈를 구하기 위한 update operator에서 Factor Graph의 edge가 연결된 프레임과의 관계만을 고려한다. 즉, 시스템의 효율성과 정확성을 위해서 최근에 대한 정보를 위주로 처리하기 때문에 장기적 정보는 거의 제거되며 이러한 factor graph 유지,보수 때문에 “장기적인 기억의 부족”문제가 발생한다고 생각하였다. 
