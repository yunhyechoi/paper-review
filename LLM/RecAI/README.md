RecAI: Leveraging Large Language Models for Next-Generation Recommender Systems

# Introduction
LLM은 general intelligence로, 여러 application에 적용하고자 하는 시도가 계속되고 있다. 추천시스템도 마찬가지이다. 
하지만 LLM을 직접 추천시스템에 적용하는 데에는 문제가 몇가지 있다.
- 마지막 학습 시점에 정보가 멈춰있다.
- 사용자의 선호는 도메인에 특화되어 있으며 빠르게 바뀐다.

</br>

따라서 RecAI는 LLM의 자연어 처리 능력과 RS의 전문적인 기능을 결합함으로써 다음의 기능을 제공한다.
- 추천에 대한 설명
- 대화를 통한 상품 추천
- 사용자 제어 강화

</br>

# Recommender AI Agent
LLM과 추천시스템을 하나의 프레임워크에 결합시킨다. 여기서 LLM은 모든 과정을 총괄하는 **brain**의 역할을 하고, 기존의 추천시스템은 **tool**로서 작동한다.
즉, 사용자와의 대화를 통해 가장 적절한 tool을 계획하고 실행하는 방식이다.

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/ae80f090-4098-46a7-a07a-1ff55b5b01d8)

위 그림에서 크게 3개의 component로 나눠서 볼 수 있다.

</br>

### 1. Task Planning
단계별 접근 방식 대신 계획 우선 방식을 채택한다. LLM은 유저 인풋, 맥락, 도구 설명, 동적으로 선택된 데모(dynamic demo)에 기반하여 계획을 수립한다.

</br>

왜 dynamic demo를 사용할까?
> 현재의 유저 의도와 가장 유사한 예시를 선택하기 위해 상황에 맞게 동적으로 고품질의 설명을 사용한다. 이를 통해 더 효과적인 계획 수립이 가능하다.

왜 계획 우선 방식을 사용할까?
> 그 이유는 계획을 미리 세워둠으로써 API call을 최소하하여 대화의 latency를 줄이고자 함이다. 

</br>

### 2. Memory
다음의 두 모듈을 통해 메모리를 관리한다.

#### Candidate bus
> 현재 추천 후보들과 도구 출력 결과를 저장하는 메모리 시스템.
> LLM의 입력 길이 제한을 우회하고, 도구들 간의 효율적인 상호작용을 가능하게 한다.

#### User Profile
> 대화 기록을 바탕으로 장기 및 단기 메모리로 나눠 사용자의 선호도를 저장한다.
> 이를 통해 사용자의 즉각적인 요청과 장기적인 선호도를 모두 고려할 수 있다.

</br>

### 3. Tools
도구들은 candidate bus를 통해 서로 상호작용하며, 최종 출력만 LLM에 전달된다. 
- infromation query : 데이터베이스에서 유저가 질문한 상품 관련 정보를 가져오는 도구 
- item retrieval : 사용자 선호에 기반하여 상품 추천
  - hard condition : 사용자가 명시한 상품 (SQL로 뽑아오는)
  - soft condition : 유사 상품 추천 (item-item 유사도로)
- item ranking : 유저 프로파일이나 기록을 기반으로 선호도 예측

</br>

### 4. Tool-learning
그림에서 나타나는 component는 아니지만, 더 작은 모델(sLM)을 훈련시켜서 비용 효율화하기 위한 과정이다.
Llama-7B모델을 사용하며 학습 데이터셋은 GPT-4가 생성한 [지시, 도구 실행 계획] 쌍으로 구성한다. 
추가로 데이터의 다양성을 위해 수작업으로 만든 대화 등을 포함한다. 
이렇게 학습한 모델을 RecLlama라고 부르며, InteRecAgent에서 GPT-3.5-turbo보다 높은 성능을 보였다. 

</br>

정리하자면, 
1. 사용자의 요청은 LLM(또는 RecLlama)에 의해 해석되고, 계획이 수립된다.
2. 이 계획에 따라 도구들이 Candidate Bus를 통해 상호작용하며 작업을 수행한다.
3. 최종적으로 사용자의 요구에 맞는 개인화된 추천을 제공한다.

</br>

# Recommendation-Oriented Language Model


