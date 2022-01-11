# Video Deblurring for Hand-held Cameras Using Patch-based Synthesis

Sunghyun Cho et al.

![figure1](1.jpg)
**Figure 1:** 각 다른 방식을 사용해서 motion-blurred 동영상을 deblurring한 결과를 비교한 것이다. (a) 자동차 동영상의 blurry frame. (b) Input 영역을 확대한 것. (c) 단일 이미지 deblurring (d) 다중 프레임 deblurring (e) 이 논문이 제시하는 방법 (f) 가까운 sharp frame에서의 비슷한 이미지 영역.

## Abstract

손으로 들어서 찍은 동영상에는 종종 카메라가 심하게 흔들려 있는데, 이는 많은 프레임들을 blurry하게 한다. 이러한 흔들리는 동영상을 복원하는 데에는 카메라 움직임을 부드럽게 하고, content를 stabilize 하는 것 뿐 아니라 동영상 프레임에서의 blur를 제거하는 것 또한 필요하다. 그러나, blur kernel은 공간, 시간적으로 변하기 때문에 동영상에서 blur를 제거하는 것은 현존하는 영상 deblurring 기술로는 제거하기 어렵다. 이 논문은 카메라 흔들림으로 생긴 blurry한 프레임들을 sharp하게 효율적으로 복원하는 동영상 deblurring 기술을 소개한다. 우리 방법은 카메라의 흔들림의 특성 상 모든 동영상 프레임이 똑같이 blurry하지는 않는다는 관찰을 바탕으로 세워졌다. 같은 물체라도 어떤 프레임에선 sharp 하지만 다른 프레임에선 blurry 할 수 있다. 우리 방법은 동영상에서 sharp한 영역을 탐지하고, 이들을 근처 프레임에서 같은 물체지만 blurry한 영역을 복원하는 데에 사용한다. 또한 이 방법은 patch 기반 합성을 사용하기 때문에 시공간적으로 일관성 있는 deblurred 된 영상을 만들어낸다. 실험 결과는 이 방법이 기존 deconvolution 기반 방법은 할 수 없었던, 움직이는 물체와 다른 이상치가 존재하기에 생기는 복잡한 비디오 blur를 효율적으로 제거한다.

**CR Categories**: I.4.3 [영상 처리와 컴퓨터 비전]: Enhancement - Sharpening과 Deblurring

**Keywords**: motion blur, video deblurring, lucky region, patch-based synthesis

[[Original Paper]](http://cg.postech.ac.kr/papers/14_Video-Deblurring-for-Hand-held-Cameras-Using-Patch-based-Synthesis.pdf) [[Web Page]](http://cg.postech.ac.kr/research/video_deblur/)

## 1. Introduction

카메라 모션은 전문가와 아마추어가 촬영한 동영상을 명확히 구분짓는 중요한 요소 중 하나이다. 전문가의 동영상을 찍을 때는 dolly나 steadicam 과 같은 카메라 모션을 부드럽게 하기 위한 특수 장치들이 사용된다. 흔들리는 카메라가 동영상에 끼치는 영향은 두가지로 나눌 수 있는데, 그 중 하나는 보기 안 좋은 동영상의 순간적인 떨림이고 다른 하나는 카메라 흔들림이 강할 때의 동영상 blur가 심해진다는 것이다.

동영상 stabilization 시스템은 흔들리는 동영상에서 카메라 모션을 부드럽게 하기 위해서 최근 발표되었다. 동영상을 stabilize 하는 데에는 성공했지만, 기존 카메라 모션에서 있었던 blurry 한 프레임은 그대로 유지되었다. 그 결과, blurry 프레임은 추가 자료 동영상에서 보듯이 stabilized 된 동영상에서 가장 눈에 띄는 단점이 되었다.

한편으론 blurry 동영상은 동영상 stabiliztion 이 좋은 결과를 만드는 데에 방해가 된다. 대부분은 stabilization은 카메라 모션을 계획하기 위해 feature tracking에 의지한다. 그러나 blurry 프레임에서의 feature tracking은 sharp한 영상의 부족으로 인해 신뢰도가 떨어진다. Video Deblurring이라 불리는, 카메라 모션에 의해서 blurry하게 된 프레임들을 sharp하게 복원하는 일은 고퀄리티의 stabilization 결과를 생성하는 데에 매우 중요하다. 이러한 이유로 동영상 deblurring은 stabilization 기술을 적용하기 전에 사용되어야 한다. stabilize한 후 간단한 deblurring을 적용하는 이전의 workflow와 큰 차이를 보인다.

동영상 모션 deblurring에서 단순히 생각할 수 있는 방법은 먼저 blurry한 프레임을 찾고, 현존하는 단일, 다수 영상 deblurring 기술을 여기에 적용하는 것이다. 불행히도 이미 존재하는 deblurring 기술은 동영상에선 만족스러운 결과를 만들어내지 못한다. Fig.1에서의 예를 보면, 최근의 단일 영상 deblurring과 다중 영상 deblurring 기술을 사용하여 blurry 프레임을 deblur하려는 모습을 볼 수 있다. 두 결과 모두 여러 이유로 심각하고 납득이 안되는 artifact를 만들어낸다. 먼저, 동영상에서의 blur kernel은 카메라와 물체의 움직임 때문에 시공간적으로 변한다. 최근의 deblurring 기술은 일반적인 카메라 모션을 효율적으로 처리하지만, 물체의 움직임은 아직 모든 프레임에서 정확한 blur kernel 예측을 방해한다. 두번째로, 좋은 kernel estimation임에도 deconvolution은 noise나 saturated 픽셀 같은 다양한 이상치에 민감하고 ringing artifact를 발생시키기 좋다. 세번째로는, 대부분의 이전 다중 프레임 deblurring 방식들이 input 이미지가 잘 정돈되어 있길 바라지만, 정확하게 동영상 프레임을 나란히 하는 것은 움직이는 물체와 깊이의 차이 때문에 보통은 불가능하다. 마지막으로, 동영상 deblurring은 시간적인 일관성을 요구한다. 영상 deblurring을 각 프레임이 직접 적용하는 것은 주로 시간적으로 일관되지 않은 결과를 출력한다.

이 논문에서, 우린 동영상 deblurring에서 kernel estimation이나 deconvolution을 동영상 프레임에 직접적으로 적용하지 않는 효율적이고 실용적인 해결책을 제시할 것이다. 이 방법은 카메라 흔들림이 주로 잦은 빈도의 불규칙한 손 모션에 의해 발생간다는 주요 관찰에서 시작되었다. 이는 움직이는 속도가 작을 때 어떤 프레임에선 선명하게 보이며 속도가 높을 땐 더 blurry하게 보이게 한다. 적절한 정렬과 모션 compensation이 있다면 이 선명한 프레임은 연관된 영역의 blurry한 프레임을 복원하는 데에 직접적으로 쓰일 수 있다.

이 방법에서, 우린 처음엔 매 프레임마다 parametic, homographic 기반의 움직임을 이용해 시차(관측 위치에 따른 물체의 위치나 방향의 차이) 때문에 더욱 복잡한 실제 움직임을 대략적으로 예측했다. 그리고 선명함을 측정하기 위해 픽셀의 luckiness(행운)를 정의하기 위해 이 대략화된 움직임을 사용했다. 동영상 프레임을 deblur하기 위해 우린 더 lucky한 픽셀을 근처 프레임에서 찾고 이들을 현재 프레임의 less lucky한 픽셀에 대체하였다. 여러 프레임 간 픽셀 대응은 추정된 동질성을 사용하여 얻은 다음 모션 모델의 부정확성을 보상하기 위해 가장 잘 일치하는 이미지 패치를 로컬 검색한다. blurry한 패치와 선명한 패치를 비교하기 위해, blurry 패치에서 추정된 blur 함수와 선명한 패치를 정방향으로 합성하여 blur 처리를 한다. blurry 프레임에서 lucky 픽셀을 복사할 때, 물체의 구조를 더 잘 유지하기 위해 패치 기반의 텍스쳐 합성 기법을 사용한다. 마지막으로, 우리는 deblur 된 프레임의 시간적 일관성을 유지하기 위해 연속 프레임의 해당 패치에 유사성 제약 조건을 부과한다.

우리 방법은 동영상에서 카메라 움직임으로 인해 정적인 물체에 생기는 blur만 제거하기 위해 설계되었다. 이는 배경만을 stabilize하는 것에 목표를 둔 동영상 stabilization 기법의 설계 목표 대신인 것이다. 움직이는 물체에 대해, 이들의 blur는 주로 최종 동영상에도 남아있는 물체의 움직임에 의한 것이 많기 때문에 그들의 blur는 주로 눈에 띄는 artifact가 되진 않는다. 반면에, 사람의 인식은 거의 변하지 않거나 천천히 변하는 배경 영역의 blur에 더 예민하게 반응하는 것을 발견했다. 우리 방법은 가장 성가신 배경 blur를 효과적으로 지울 수 있고, Sec.5에서 시연할 것처럼 전경 물체를 이동할 때 안정적으로 잘 작동할 수 있다.

### Note

**_homography-based motion_**
**_latent image_**
**_duty cycle_**
**_energy minimalization formulation_**
