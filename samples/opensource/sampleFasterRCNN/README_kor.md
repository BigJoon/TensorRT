# Object Detection With Faster R-CNN

**순서나 목차 내용들**
- [Description](#description)
- [How does this sample work?](#how-does-this-sample-work)
    * [Preprocessing the input](#preprocessing-the-input)
    * [Defining the network](#defining-the-network)
    * [Building the engine](#building-the-engine)
    * [Running the engine](#running-the-engine)
    * [Verifying the output](#verifying-the-output)
    * [TensorRT API layers and ops](#tensorrt-api-layers-and-ops)
- [Running the sample](#running-the-sample)
    * [Sample `--help` options](#sample---help-options)
- [Additional resources](#additional-resources)
- [License](#license)
- [Changelog](#changelog)
- [Known issues](#known-issues)


## Description

FasterRCNN 예제는 TensorRT plugins을 사용하며 추론을 수행하고 FasterRCNN모델의 종단 간 추론을 위해 통합이 된 사용자정의의 레이어를 사용합니다. 특히 이 예제에서 여러분은 TensorRT에서 구현된 Faster R-CNN네트워크를 볼 수 있고, 성능 테스트를 보며, 융합되어 구현된 사용자 정의 레이어들을 확인할 수 있습니다. 첨언하여 INT8 캘리브레이션(교정)이나 사용자가 훈련한 네트워크 등을 최적화 시킬 수 있습니다. Faster R-CNN 네트워크는 [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](https://arxiv.org/abs/1506.01497) 요오기의 논문에서 확인 가능합니다.

## How does this sample work?

Faster R-CNN은 Fast R-CNN과 RPN(Region Proposal Network:지역 제안 네트워크)을 합친겁니다. 후자 그러니깐 RPN은 각 위치에서 물체의 경계와 물체 점수를 동시에 예측하는 Fully Convolutional Network 입니다. RPN은 Fast R-CNN과 하나의 네트워크로 통합할 수 있습니다. 왜냐하면, Fast R-CNN 탐지 네트워크와 함께 한쪽 끝에서 다른 쪽 끝까지 학습되고, 이는 전체 이미지 Convolutional feature를 공유하기 때문에 Region Proposal(지역 제안)이 거의 공짜로 되는 거죠. 이 Region Proposals(지역 제안)은 이후에 Fast R-CNN에 의해 Detection(탐지)에 사용되게 됩니다.

Faster R-CNN은 RCNN이나 Fast R-CNN 보다 빠르고 정확합니다. 왜냐하면 종단간 추론을 가능케 하였고 Region Proposal이 독립적으로 있는게 아니라 합쳐졌기 때문입니다. 첨언을 하자면 Fast R-CNN에 selective Search는 독립적으로 존재했구요. 또는 Classification 방법 중에 RCNN에서 사용한 SVM도 독립적인 모듈이었죠.

자자.. 이 예제는 이렇게 5가지의 단계들이 있습니다:
- [Preprocessing the input](#preprocessing-the-input)
- [Defining the network](#defining-the-network)
- [Building the engine](#building-the-engine)
- [Running the engine](#running-the-engine)
- [Verifying the output](#verifying-the-output)

sampleFasterRCNN 예제는 TensorRT plugin library에 있는 plugin을 사용합니다. TensorRT plugin library에는 Faster R-CNN의 RPN과 ROI Pooling레이어들이 통합구현된 것이 들어있습니다. 이 특정한 레이어들은 Faster R-CNN 논문에서 가져왔고 'RPNROIPlugin'이라는 오직 단 하나의 plugin으로 만들어져있습니다. 이 plugin은 TensorRT Plugin Registry에 'RPROI_TRT'라는 이름으로 등록되어 있어요.

### Preprocessing the input(입력 좀 다듬기)

Faster R-CNN은 입력으로 3개의 채널 그리고 375x500사이즈로 된 이미지입니다. TensorRT는 그 어떤 컴퓨터 비젼 라이브러리도 사용하지 않기 때문에, 이미지들은 각 픽셀마다 'R', 'G' 그리고 'B'라는 순서의 바이너리 값들로 구성되어 있죠. 포맷은 Portable PixMap(PPM)인데, 이건 netpbm 색깔 이미지 포맷이에요. 이 포맷에서 'R', 'G', 'B' 값들은 주로 (0~255)의 바이트 정수로 구성이 됩니다. 그것들은 픽셀마다 같이 저장되어 있어요.(이거 쓸데없는 말임.안봐도 됨.궁금하면 찾아서 이미지 포맷 확인하고)

그런데, Faster R-CNN을 보면 'B', 'G', 'R'순서로 입력을 받아요. 그래서 이 네트워크에 입력으로 넣어서 옳바른 결과를 얻고 싶다면 순서를 뒤짚어야해요.
