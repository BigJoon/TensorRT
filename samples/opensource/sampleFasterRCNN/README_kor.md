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

Faster R-CNN은 Fast R-CNN과 RPN(Region Proposal Network:지역 제안 네트워크)을 합친겁니다. 
