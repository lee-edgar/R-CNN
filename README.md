# R-CNN
the summary of R-CNN's thesis


Region Proposal : 

- Sliding Window방식의 비효율성을 개선하기위해 Selective search를 이용해 물체가 있을법한 영역을 빠르게 찾아내는 알고리즘.
- 방식
    - edge boxes : 이미지의 엣지, 라인의 유사영역묶기
    - selective search : 이미지의 픽셀의 유사영역 묶고, 해당 영역을 점차적으로 넓혀감.

Region proposal의 Selective search(SS) : "object가 있을 만한 후보 영역을 찾자"

- 컬러, 무늬(texture), 크기(size), 형태(shape)에 따라서 유사한 Region을 계층적으로 계산
- 원본이미지 -> 최초 segmentation -> 후보 objects 검출 -> segmentation -> 후보 objects 검출
- segmentation 그룹핑(중복되는 segment 합치기)을 계속 반복하면서 수행하게 되면, 후보 objects 검출 정확도가 올라가게됨.

![Screen Shot 2022-09-21 at 3.30.09 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/88e8a0f1-badf-48d6-a781-81a2f782dd6b/Screen_Shot_2022-09-21_at_3.30.09_PM.png)

Image input → extract region proposal을 실행 해 2000건의 warped region생성. →  추후 classifiaction dense layer를 위한 227*227사이즈 조절(보통 warped region의 사이즈를 늘리기 때문에 찌그러짐) → CNN적용(flatten FC Input layer까지)하여 feature map생성 후  → SVM Classifier(classification / regression)

R-CNN순서:

- region propoal → feature extractor →  classification / regression
- regino proposal → cnn(with pre-traine d) → SVM → bounding box regression

### 1. Region Proposal Level

selective search알고리즘을 사용하여 임의의 bounding box 설정.

selective search algorithm

- 1)Bounding box들을 Random하게 작게 많이 생성을 한다.
- 2) 그리고 이것들을 계층적 그룹핑 알고리즘을 사용해 조금씩 Merge 해나간다.
- 3) 이를 바탕으로 ROI(Regions of Interest)라는 영역을 제안하는 Region Proposal 형식으로 진행된다.*

### 2. CNN(with pre-trained, Alex Net[Image Net]) Level

CNN은 인풋값의 크기가 고정되어 있기때문에, region proposal level에서 나온 bounding box의 사이즈들을 같은 사이즈로(227*227) 통일해주는 Warping 진행.

### 3. SVM(Support Vector Machine) Level

![Screen Shot 2022-09-22 at 12.57.00 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99875aae-cfec-41c7-a0ba-b5a53659a8dd/Screen_Shot_2022-09-22_at_12.57.00_PM.png)

### 4. Bounding Box Regression

![Screen Shot 2022-09-22 at 8.37.46 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d431af28-da34-4652-9d77-df3cf0200af3/Screen_Shot_2022-09-22_at_8.37.46_PM.png)

- P : region proposal 통해 제안된 Bounding Box || G : Ground Truth Bounding Box( 정답박스)

P를 G와 비슷하게 맞추도록 transform하는것이 bounding box regression의 최종 목표!

### R-CNN 단점

우선 모델이 너무 복잡하다. (여기서 4가지로 나눠서 정리했지만) 일단 모델이 3가지 이상이나 사용된다. 또한 R-CNN의 첫 번째 모듈인 Region Proposal에서 사용되는 Selective Search는 CPU를 사용하는 알고리즘이다..! 그리고 이 Selective Search에서 뽑아낸 2000개의 영역 이미지들에 대해서 모두 CNN 모델에 넣는다.GPU 에서 이미지 한 장당 대략 13초가 걸리는데, CPU로는 53초가 걸린다고 한다. 이걸 2000장 반복하는 것이다..! 따라서 당연히 시간이 오래 걸릴 수밖에 없다. 또한 각 모듈이 동시에 일어나지 않고 시간도 오래 걸리므로 real-time 분석이 어렵다는 단점 역시 있다.

이러한 여러 한계점들로 인해 보완된 모델들이 계속 나오게 되는데 오늘 R-CNN의 기본 구조에 대해 이해하고 있다면 이후 모델들은 더 쉽게 공부할 수 있을 거라고 생각된다!

출처 :

[https://velog.io/@whiteamericano/R-CNN-을-알아보자](https://velog.io/@whiteamericano/R-CNN-%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)
