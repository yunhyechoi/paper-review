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
도구들의 연쇄적인 실행이 이뤄지므로, 이전 도구의 출력이 후속 도구의 입력으로 들어가게 된다. 
하지만 LLM에는 입력 토큰의 제한이 있기 때문에 모든 출력을 프롬프트에 입력하기에는 한계가 있다.

이를 해결하기 위해 아이템 후보군을 저장하는 별도의 메모리로 candidate bus를 도입한다. 
- data bus : 상품 후보군 저장 
- tracker : 도구의 출력 기록 -> 나중에 reflection 단계에서 활용 

</br>

#### 갑자기 웬 버스..?
> 컴퓨터에서 bus라는 용어는 실제로 대중교통수단인 버스에서 유래한다. 버스의 역할이 여러 사람들을 한 곳에서 다른 곳으로 실어 나르는 것처럼, 컴퓨터 시스템에서의 버스는 여러 가지 구성 요소들 사이에서 데이터를 전송하는 역할을 한다. 즉, CPU, 메모리, 입출력 장치 등 컴퓨터 시스템의 다양한 구성 요소들을 연결하는 통신 경로를 의미한다. 데이터와 제어 신호가 이 버스를 통해 각 부품으로 전달되고 공유된다.

</br>

### User Profile
도구 호출을 용이하게 하기 위한 유저의 프로파일을 저장하는 메모리이다.
- 구조
  - 단기 메모리 : 현재 프롬프트의 최근 대화에서 지속적으로 파생
  - 장기 메모리 : 현재 대화가 LLM 입력 크기를 초과하면 이전 대화까지의 단기 메모리 프로파일을 불러와 장기 메모리와 병합하여 업데이트
- 구성
  - 딕셔너리 구조로 사용자의 선호도를 캡슐화하여 관리 
  - like : 사용자의 긍정적 취향
  - dislike : 사용자의 부정적 취향
  - expect : 현재 대화에서 사용자의 즉각적인 요청(검색 등)

</br>

## 2. Task Planning
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/54d08f83-b904-480c-8665-d6b1a668fdee)

### Plan-first Execution
API 호출수를 효율적으로 줄이기 위해 step-by-step 대신 2단계 실행 방식을 사용한다.
- plan : LLM이 대화에서 파악된 사용자 의도를 기반으로 도구 실행 계획을 생성
- execution : executor가 계획대로 도구를 호출하고 출력을 메모리 버스에 기록한다. 마지막 도구 출력만 LLM의 관측치로 사용된다.

</br>

### Dynamic Demonstrations
plan 단계에서 few-shot용으로 현 상황에 가장 유사한 예시를 넣어주는 과정이다.
- 예를 들어,
  - 데모1
    - 사용자 의도: "저는 콜오브듀티, 포트나이트 같은 게임을 했었어요. 비슷한 FPS 게임 추천해주세요."
    - 도구 실행 계획: [아이템 검색 도구 호출 -> 랭킹 도구 호출]
  - 데모2
    - 사용자 의도: "스토리가 좋은 롤플레잉 게임 원해요. 200달러 이하로 해주세요."
    - 도구 실행 계획: [SQL 검색 도구 호출 -> 랭킹 도구 호출] 

</br>

동적 데모 할당의 방식은 다음과 같다.
1. 약 20개 정도의 시드 데모 수동 작성
2. LLM을 이용하여 더 많은 데모 생성
   - input-first : LLM이 시드 데모의 의도를 모방하여 새로운 사용자 의도 x를 생성 -> x에 적합한 계획 p 생성
   - output-first : 주어진 계획 p로부터 적합한 의도 x 생성 -> 생성된 의도 x에 대해 새로운 계획 p' 생성 -> p와 일치하는 p'만 사용
3. 현재 사용자의 의도와 가장 유사한 몇개의 데모만 in-context learning에 활용
       
</br>

## 3. Reflection
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/b7713385-23e1-461b-bb78-89e219fe1d1a)


LLM을 통해 계획하고 실행하다보면 에러 발생은 피할 수 없다.
- 존재하지 않는 도구 선택
- 도구의 남용
- 잘못된 출력 형식

이러한 오류들을 줄이기 위해 self-reflection 과정을 수행한다.
- actor : 도구 실행 계획($p_t$)을 세우고 출력($o_t$)과 응답($y_t$)을 생성
- critic : actor의 생성 결과를 보고 평가
- critic의 평가가 긍정적이면 바로 유저에게 결과값 제공, 부정적이면 actor에게 재계획 지시

</br>

## 4. sLM
![image](https://github.com/yunhyechoi/paper-review/assets/166207923/06b63342-c6f3-45b5-901b-98591671ca99)

기본적으로 GPT-4를 사용하지만, 비용을 줄이기 위해 7B Llama를 튜닝하여 적용해본다. 

RecLlama 학습데이터셋은 GPT-4를 활용해 생성한 [지시문, 도구 실행 계획] 쌍을 사용한다.
- 사용자 시뮬레이터와 GPT-4 기반 추천 에이전트 간 대화에서 샘플 수집
- 30개의 다양한 대화를 직접 구성하고, 그 중 3개를 무작위로 선택하여 GPT-4에게 대화 기록과 실행 계획 생성

