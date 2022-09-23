# R-CNN
the summary of R-CNN's thesis
/hr

# RCNN

Region Proposal : 

- Sliding Window방식의 비효율성을 개선하기위해 Selective search를 이용해 물체가 있을법한 영역을 빠르게 찾아내는 알고리즘.
- 방식
    - edge boxes : 이미지의 엣지, 라인의 유사영역묶기
    - selective search : 이미지의 픽셀의 유사영역 묶고, 해당 영역을 점차적으로 넓혀감.

Region proposal의 Selective search(SS) : "object가 있을 만한 후보 영역을 찾자"

- 컬러, 무늬(texture), 크기(size), 형태(shape)에 따라서 유사한 Region 을 계층적으로 계산 → 이웃하는 region 사이의 유사도를 구하고, 유사도가 높은 것 부터 차례대로 Merge 하여 2000개 구성
- 원본이미지 -> 최초 segmentation -> 후보 objects 검출 -> segmentation -> 후보 objects 검출
- segmentation 그룹핑(중복되는 segment 합치기)을 계속 반복하면서 수행하게 되면, 후보 objects 검출 정확도가 올라가게됨.

![Screen Shot 2022-09-21 at 3.30.09 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/88e8a0f1-badf-48d6-a781-81a2f782dd6b/Screen_Shot_2022-09-21_at_3.30.09_PM.png)

![https://s3.us-west-2.amazonaws.com/secure.notion-static.com/88e8a0f1-badf-48d6-a781-81a2f782dd6b/Screen_Shot_2022-09-21_at_3.30.09_PM.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220923%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220923T044110Z&X-Amz-Expires=86400&X-Amz-Signature=54df0709d4ab87334aa88d516447879c862f308b14443d5ae365671bcb75bc28&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Screen%2520Shot%25202022-09-21%2520at%25203.30.09%2520PM.png%22&x-id=GetObject]
Image input → extract region proposal을 실행 해 2000건의 warped region생성. →  추후 classifiaction dense layer를 위한 227*227사이즈 조절(보통 warped region의 사이즈를 늘리기 때문에 찌그러짐) → CNN적용(flatten FC Input layer<img width="465" alt="Screen Shot 2022-07-06 at 5 26 32 PM" src="https://user-images.githubusercontent.com/61241244/191893264-46283e40-7789-49c9-8117-e9a20375b821.png">
까지)하여 feature map생성 후  → SVM Classifier(classification / regression)

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

- feature vector를 Linear SVM으로 Class별 Score를 계산하여 Clssification

### 4. Bounding Box Regression

![Screen Shot 2022-09-22 at 8.37.46 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d431af28-da34-4652-9d77-df3cf0200af3/Screen_Shot_2022-09-22_at_8.37.46_PM.png)

- P : region proposal 통해 제안된 Bounding Box
- G : Ground Truth Bounding Box( 정답박스)

P를 G의 거리를 최소화 되도록 학습하는 것이 bounding box regression의 최종목표

### 5. Fine-tuning

1. Postive Sample : Selective Search로 만들어낸 region과 할당된 object의 실제 box 사이의 IOU값이 0.5이상의 값들로 traing
2. 새로운 조건(아래)의 샘플을 구성하고, 앞에서 구성한 fine-tunned CNN으로 얻은 피처 벡터로 학습.
- Postive Sample : Class 별 object의 ground-truth box
- Negative Sample : Selective Search로 만들어낸 region과 할당된 object의 실제 box 사이의 IOU값이      0.3 미만의 값

### 결론

- Fine-tuning을 할수록 좋은 성능과 bounding box regression 학습을 할수록 좋은 성능을 나타낸다.
- 고정된 크기의 이미지가 인풋으로 들어가면서 이미지가 손상되다.
- region proposal에서 사용된 selective search는 cpu를 사용하는 알고리즘이며, selective search에서 2000개의 영역 이미지를 각각 CNN을 수행하기에 학습시간이 너무 길다.

출처 :

[https://velog.io/@whiteamericano/R-CNN-을-알아보자](https://velog.io/@whiteamericano/R-CNN-%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)

고려대학교 산업경영공학부 DSBA 연구실 천우진  - Rich feature hierachies for accurate object detection and semantic segmentation : 

[https://www.youtube.com/watch?v=X4TxIPuYu0E](https://www.youtube.com/watch?v=X4TxIPuYu0E)
