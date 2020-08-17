# 의류 가상 시착용 서비스 Fittingroomanywhere

## Overview
<p align="center"><img src="https://ifh.cc/g/9yXD8Y.jpg"></img></p>

**Fittingroomanywhere**은 이미지 상의 인물의 의상을 자연스럽게 바꿔주는 가상 의류 시착용 서비스입니다. Mask R-CNN을 통해 이미지 상의 의류를 mask로 인식하여 segmentation 작업을 진행하였으며, CycleGAN을 활용하여 의류를 원본 이미지 상의 의류에 맞게 색상과 패턴을 적용했습니다. 


## Requirements

```
pip install keras
pip install tensorflow==1.13.1
```

## Project Details

### Data  수집 및 전처리
<p align="center"><img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpjCpD%2FbtqxqJeofdT%2FdqQ0RGDVhGzMQFyfaGpf71%2Fimg.png" width="800px"></p>

- Deep Fashion 2 Dataset 에서 이미지의 segmentation 좌표와 Category 정보 포함된 COCO dataset을 활용함
- category= ‘short sleeve top’ 인 이미지 중 수작업으로 정면 이미지만 선별
- COCO 형태의 segmentation 요소를 VIA의 polygon 형태로 변환


### Image Segmentation
<p align="center"><img src="https://ifh.cc/g/mZlkC3.jpg" width="800px"></p>

- Mask R-CNN을 통해 의류 이미지 segmentation 모델 학습
- pretrained된 모델에 layer 추가, epoch=30으로 진행했을 때 최적의 결과가 나옴

 
### Image Generation

<p align="center"><img src="https://ifh.cc/g/mvso7G.jpg" width="800px"></p>

- CycleGAN을 통해 원본 이미지(Input image, 흰 옷)를 가상으로 시착용하고자 하는 의류(파란 옷)로 매핑시켜 최종 결과물(Ouput image)를 얻음
- dentity loss 를 학습할 때 가중치가 너무 크면 스타일을 반영하지 못하여 가중치를 줄이고 identity loss 를 20 step 당 한번씩 주고 학습
- Smooth l1 loss, weigth decay=0.09, epoch=40으로 하이퍼 파라미터를 조정함

### Overall Framework

<p align="center"><img src="https://ifh.cc/g/Bp3rtm.jpg" width="800px"></p>

1)  사용자가 착용하기를 희망하는 style image를 segmentation 진행
2)  style image를 색상, 패턴에 따라 분류한 후 해당하는 분류에 따른 weight값을 load함
3)  사용자가 업로드한 이미지를 segmentation하여 segmentation한 이미지, mask, bounding box에 대한 정보를 얻음
4) segment된 유저 이미지를 CycleGAN 모델에 적용하기 위해 256 사이즈로 resize해줌
5)  resize된 이미지와 style image에 대한 weight값을 적용하여 새로운 이미지를 생성함
6)  새롭게 생성된 이미지를 원본 이미지로 resize해줌
7)  bounding box 사이즈만큼 crop해줌
8)  그리고 원본 이미지 사이즈만큼 padding을 추가함
9)  user image와 새롭게 생성된 이미지에 mask를 적용시켜줌
10)  두 이미지를 합쳐 최종 output을 반환함

