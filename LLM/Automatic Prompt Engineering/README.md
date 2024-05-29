# 개요

### Motivation
- LLM은 prompt 만으로도 높은 성능을 보이지만, prompt에 민감하게 반응한다.
- 이 때문에 모든 task에 대해 사람이 직접 적합한 prompt를 작성하는 것에는 한계가 있다.
- 이러한 문제에 대한 대안으로 사람의 손을 거치지 않고 **LLM이 prompt를 직접 생성**하도록 자동화하는 연구들이 진행되고 있다.

</br>

### Main Point
프롬프트 생성 자동화에는 다음의 두가지 문제가 존재한다.
- Instruction Generation : 주어진 task에 맞게 지시문 생성
- Instruction Ranking : 얼마나 좋은 프롬프트인지 성능 예측 평가

</br>

### 요약
|stage|feature|APE|Auto Instruct|
|--|--|--|--|
|Instruction Generation|백본 모델|Instruct GPT|ChatGPT-4|
||방식|3개의 meta-prompt로 여러개 생성|7가지 meta-prompt로 3개씩 총 21개 생성|
|Instruction Ranking|백본 모델|Instruct GPT|FLAN-T5-LARGE|
||방식|accuracy & log prob|trained ranking model|

</br>

### 논문 리스트 
- Automatic Prompt Engineer (APE) : https://arxiv.org/abs/2211.01910
- Auto Instruct : https://arxiv.org/abs/2310.13127

</br>

# APE

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/51e50da2-f280-4d57-a1ef-20f5f4254b0f)

지시문 초안과 few-shot 예시가 주어진 상황에 사용하기 좋은 방법이다. zero-shot에도 활용 가능하다.
1. **Instruction Generation**
   
   다음의 3가지 meta prompt 중 상황에 적합한 프롬프트를 선택하여 후보군 생성
   - Forward Mode Generation 
   - Reverse Mode Generation 
   - Customized Prompts 
2. **Instruction Ranking**
   
   1에서 생성한 지시문을 통해 예측한 결과 평가
   - Execution Accuracy : 0-1 loss
   - Log Probability : 모델의 출력 확률
3.** Iterative Instruction Generation (Optional)**
   - 2의 score가 기준치를 만족하지 못할 경우, 추가적으로 실행하는 단계
   - 2에서 상위 score였던 intruction을 활용하여 유사한 지시문 재생성을 통해 후보군 증강
   - 최종적으로 score가 가장 높게 나타난 instruction 선택

</br>

# Auto-Instruct


![image](https://github.com/yunhyechoi/paper-review/assets/166207923/91b1c284-d17f-4ca7-b25a-70eb8853c404)


1. Instruction Generation
   - 4가지 스타일의 meta-prompt를 통해 다양한 유형의 지시문 후보를 생성
   - 각 스타일별로 3개씩 후보 샘플링
3. Instruction Ranking
   - FLAN-T5-LARGE 모델을 fine-tuning하여 사용
     - 다음의 프롬프트를 입력하고 'yes' 토큰 logit prob 계산
       - ```Example: x. Input: Ic. Is this a good instruction to solve the example?```
     - 지시문 후보 (Ic)를 통한 출력결과 $\hat{y}$와 y간의 ROUGE-L 계산
     - 두 점수 사이의 KL-divergence가 최소화되도록 학습
   - 학습된 모델을 통해 각 후보별로 score 계산
   - score가 가장 큰 지시문 최종 선택
