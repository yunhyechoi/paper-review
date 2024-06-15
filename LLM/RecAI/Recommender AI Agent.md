Recommender AI Agent: Integrating Large Language Models for Interactive Recommendations

# Introduction
모든 애플리케이션에서 LLM은 대화형 인터페이스로의 발전에 큰 기여를 하고 있다. 이는 추천 시스템에서도 마찬가지다.
하지만, LLM만을 이용하여 추천시스템을 만드는 것에는 몇가지 문제점이 존재한다.
- 도메인 특화된 패턴에 약하다.
- 오픈 데이터가 없는 도메인은 커버할 수 없다.
- 새로 나온 상품에 대한 업데이트가 어렵다.

따라서 본 논문에서는 LLM의 이러한 한계점을 도메인 특화 모델 (추천 시스템 모델)과 결합함으로써 해결할 수 있는 프레임워크를 제안한다. 
- LLM : brains -> 유저와의 대화를 통해 모든 계획과 도구 실행을 총괄하는 역할
- Recommender System : tools -> LLM의 실행 요청에 맞게 작동하는 도구

</br>

# Methodologies
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/c08cd1f6-2b6d-47bb-a7d5-25de6c1366c0)


## IntRecAgent Framework
LLM이 유저와 대화하며 유저의 의도를 파악하고, 어떤 도구를 사용할지 결정하고, 실행 결과를 응답에 활용한다. 따라서 도구들의 성능이 굉장히 중요하게 작용한다. 
InteRecAgent에서는 기본적으로 3가지 tool을 사용하고 있다. 

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/a38c3c85-273b-47df-b714-5e1ee75b9873)

### 1. Information Query
유저 문의를 처리하는 도구
- SQL을 통해 데이터베이스로부터 상품의 정보를 추출한다.
- ex. "이 게임은 언제 나왔고, 가격을 얼마인지 알려줘" 

</br>

### 2. Item Retrieval
유저의 요구를 만족시키는 상품의 후보들을 리스트업하는 도구
- 하드 컨디션(Hard Conditions)
  - 명시적인 아이템 속성에 대한 요구 사항
  - SQL을 통해 데이터베이스에서 후보군 추출
  - ex. "인기 있는 스포츠 게임을 추천해 줘" 또는 "100달러 미만의 RPG 게임을 추천해 줘"
- 소프트 컨디션(Soft Conditions)
  - 명확히 표현할 수 없는 의미적 유사성에 대한 요구
  - 임베딩을 통한 유사 아이템 매칭
  - ex. "콜오브듀티, 포트나이트 같은 게임을 추천해줘"
 
</br>

### 3. Item Ranking
유저 기록을 기반으로 상품 추천 후보군에 대한 선호도를 예측하는 도구
- retrieval 단계에서 나온 상품 아웃풋
- 또는 유저가 직접 언급한 상품들
  - ex. "A랑 B 중에 어떤게 나한테 더 잘 맞을까?"   

### 정리
3가지 도구가 어떻게 사용되는지 전체적인 흐름을 정리해보자.
0. 유저 : "콜오브 듀티를 해봤는데, 이번에는 이것보다 나중에 나온 퍼즐 게임을 하고 싶어. 추천해줘."
1. Item Query : SQL을 통해 콜오브 듀티의 출시일자 확인
2. Item Retrieval : SQL을 통해 출시일자 이후의 퍼즐게임 추출 (hard condition)
3. Item Ranking : 콜오브 듀티를 유저의 프로파일로 간주하고 ranking model로 랭킹

</br>

이제 이런 도구들을 어떻게 계획하고 실행할 것인가?

초기에는 step-by-step 접근법을 사용하였으나, 여러 한계들이 발견되었다. 이를 해결하기 위해 메모리, 작업 계획, 도구 학습 능력 등의 핵심 구성요소를 개선하였다.
하나 하나 살펴보자.

</br>

## 1. Memory Mechanism
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/a50efad5-40ee-432e-a739-0c0e656e0e29)

### Candidate Bus

### User Profile


## 2. Task Planning
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/54d08f83-b904-480c-8665-d6b1a668fdee)

## 3. Reflection
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/5a06d2d9-90e8-4d9d-a4a4-41e31b3c5716)

## 4. sLM
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/06b63342-c6f3-45b5-901b-98591671ca99)


