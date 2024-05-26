# Attention Mechanism
## RNN

### 알고리즘 
타임 스텝 순서대로 인풋이 들어가고 그 정보가 순차적으로 축적되는 방식이다. 
t 스텝 다음에 올 단어를 예측할 때, 이전까지 등장한 모든 단어를 누적하여 생성된 벡터를 활용한다.

- $h_t = tanh(W_{hh}h_{t-1} + W{xh}x_t + b_h$)
- $y_t = W_{hy}h_t + b_y$

<img src="https://github.com/yunhyechoi/paper-review/assets/166207923/fb008610-ecde-4ade-b6c4-50502f3b8fc6" width="600" />

</br>

### 문제점
RNN의 가장 큰 문제점은 초반에 등장한 단어의 정보가 뒤로 갈수록 점점 희미해진다는 것이다. 
이를 해결하고자 등장한 것이 바로 LSTM이다. 

LSTM은 입력 중에서 핵심적인 정보를 잊어버리지 않고 다음 스텝에 전달되도록 고안되었다. 
다음의 게이트들을 이용하여 입력값을 얼마나 더해줄지, 과거 정보는 얼마나 활용할지, 아니면 아예 잊어버릴지 등등을 결정하여 cell state에 반영한다. 
그리고 이 cell state에는 linear한 변형만 함으로써 최대한 정보를 보존할 수 있도록 한다.
- input gate
- forget gate
- output gate
- cell state

<img src="https://github.com/yunhyechoi/paper-review/assets/166207923/449e8679-8174-4d48-8242-c98f5efa9394" width = "600"/>

</br>

## Seq2Seq
RNN 계열은 t 스텝 이전까지의 단어 정보만을 활용하여 다음에 나올 단어를 예측한다. 
다시 말해 입력된 문장의 길이와 출력되는 문장의 길이가 동일하다는 것이다. 
하지만 문장을 번역하는 경우를 생각해보자. 한국어 문장과 번역된 영어 문장의 길이는 서로 다르다. 이 뿐만 아니라 나라마다 어순이 다르기 때문에 전체 문장에 대한 정보가 필요하다. 
Seq2Seq는 이러한 문제들을 해결하고자 고안된 모델이다.

인코더와 디코더는 RNN 계열 모델로 구성된다. 따라서 Encoder에서 나온 임베딩 값은 RNN 모델의 마지막 hidden state이고, 이 값을 context라고 부른다. 
그리고 이 context는 다시 Decoder에서의 RNN의 첫번째 hidden state로 사용된다. 
- Encoder : 문장 임베딩 (context)
- Decoder : context 기반 번역

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/02f6f925-90d0-41d4-89da-9054f8dc2bdc)

</br>

## Attention
Seq2Seq 모델에서는 입력 문장을 하나의 context vector로 압축한다. 이에 따른 문제는 크게 두가지가 있다.
- 고정된 크기의 벡터로 압축하면서 정보를 손실하게 된다.
- gradient vanishing 문제가 발생한다.

이에 따라 문장이 길어질수록 품질이 떨어지는 문제가 여전히 발생한다. 이러한 문제를 해결하고자 context로 압축하지 않고 중요하게 **집중해서 봐야할 단어들의 정보를 중점적**으로 Decoder에 전달하는 Attention 기법이 등장하였다.

기존의 Seq2Seq 모델에서는 각 인풋을 순차적으로 RNN 모델을 통해 축적하여 context vector를 만들고 Decoder에 전달했다.
하지만 Attention을 사용한 모델에서는 모든 인풋에 대한 인코딩 값을 모두 고려하여 현재 예측해야하는 time step에서 가장 중요하게 봐야할 인풋 정보를 활용한다. 

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/b47baec3-3f80-45d6-b3fa-84ca815598dc)

</br>

# Transformer
Attention은 주로 RNN 계열 모델과 함께 사용되는데, 이때 가장 큰 문제점은 RNN의 sequential한 특성으로 인해 병렬화가 불가능하다는 것이다. 
Transformer에서는 RNN 없이 **attention만을 이용**하여 계산의 효율성을 극대화한다.


## Model Architecture
모델 구조도를 보면 RNN 모델 없이 attention으로만 이루어진 것을 볼 수 있다.

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/546dbdd5-7eed-42ec-8371-1af4f10cbb64)

기본적으로 Encoder-Decoder 구조를 따른다. 
- Encoder : 입력 문장을 수치형 벡터로 변환
  - multi-head attention
  - feed forward network

- Decoder : 인코딩 값을 기반으로 각 time별 출력값 생성
  - maked multi-head attention
  - multi-head attention
  - feed forward network 

각 sublayer는 다음과 같이 `residual connection`(잔차 연결)과 `layer normalization`(층 정규화) 과정을 거친다. 
- $LayerNorm(x + Sublayer(x))$

</br>

## Attention
attention은 쉽게 말해서 예측해야 할 단어와 연관이 있는 단어에 좀 더 집중해서 보겠다는 의미이다. 
Transformer 에서는 scaled dot production attention을 사용하며, 이를 함수로 표현하면 다음과 같다. 
$$Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V$$
- Query : 관련된 부분을 찾고 싶은 벡터
- Key : Query와 얼마나 연관이 있는지 확인해야 하는 벡터
- Value : Query와 Key간의 유사도를 가중치로 곱해줄 벡터

즉, attention은 (query와 key의 유사도가 높은) value에 집중하도록 하는 연산 메커니즘이다. 

</br>

### Multi-Head Attention
Transformer이 제안된 가장 큰 이유가 바로 병렬화였다. multi-head attention은 여러 head로 나눠서 attention 계산을 병렬적으로 수행하는 알고리즘이다.

![image](https://github.com/yunhyechoi/paper-review/assets/166207923/e80cd731-bde6-4132-8527-53288f635767)

multi-head attention을 수식으로 나타내면 다음과 같다. 
수식을 간단하게 해석해보면, 각 head는 Q,K,V를 h개의 다른 방식(가중치)로 projection하여 생성한 벡터이며, 이렇게 나눠진 head에 대하여 각각 attention을 수행한 후, 연결하는 방식이다. 논문에서는 h=8로 지정하여 총 8번의 연산을 병렬적으로 수행한다.

$$MultiHead(Q,K,V) = Concat(head_1,...,head_h)W^O$$

$$\text{where, } head_i = Attention(QW_i^Q, KW_i^K, VW_i^V)$$

</br>

한번에 계산이 가능한데도 왜 이런 방법을 사용한걸까? 
> 동일한 입력에 대해 h번 만큼 나누어서 연산을 수행한다는 것은 h가지의 다양한 관점에서 시퀀스를 표현할 수 있다는 것을 의미한다.
> 이러한 이유로 transformer에서는 multi-head 방식을 사용하고 있다.

</br>

### Masked Multi-Head Attention
transformer의 디코더 초기에 등장하는 layer로, masked가 붙었다. 말그대로 '가려진' 상태에서 수행되는 multi-head attention이다. 
가리는 이유는 아주 간단하다. 어텐션 과정에서 미래의 값을 사용하지 못하도록 하기 위함이다. 

학습 과정에는 디코더의 인풋으로 번역된 문장이 들어오게 된다. 따라서 다음에 나올 단어까지 포함하여 어텐션을 수행하게 되면 정답을 보여주는 격이 된다.
따라서 디코더에서는 미래의 나올 단어를 참고하지 못하도록 masking하는 과정이 추가된다. 

마스킹하는 방법은 여러가지가 있지만, 트랜스포머에서는 $- \infty$를 곱하는 방식을 사용한다. 
attention 과정에서 $- \infty$를 곱하는 이유는 이 값이 softmax를 지나게 되면 0으로 변환되기 때문이다. 


![image](https://github.com/yunhyechoi/paper-review/assets/166207923/17955812-3bd4-4898-87fd-5410f2375ba3)

</br>

## Feed Forward Network
FFNN 수식은 다음과 같다.
$$FFNN(x) = ReLU(xW_1 + b_1)W_2 + b_2 = max(0,xW_1 + b_1)W_2 + b_2$$
- linear transformation : $f_1 = xW_1 + b_1$
- activation : $f_2 = ReLU(f_1)$
- linear transformation : $f_3 = f_2W_2 + b_2$

</br>

## Positional Encoding
트랜스포머에서는 RNN 없이 어텐션만 사용한다. 이 때 문제점은 입력되는 단어들의 순서를 모델이 인식하지 못한다는 것이다.
따라서 인풋으로 들어가는 단어의 위치 정보를 주기 위하여 positional encoding 값을 단어 임베딩에 더해준다.

트랜스포머에서 사용하는 positional encoding 식은 다음과 같다. (i : 인코딩 벡터의 차원 / pos : 단어 순서)
$$PE_{(pos,2i)} = sin(frac{pos}{10000^{2i/d_{model}}})$$
$$PE_{(pos,2i+1)} = cos(frac{pos}{10000^{2i/d_{model}}})$$

</br>

위의 식을 통해 알 수 있는 특징은 다음과 같다.
- i가 짝수일 때는 sine 함수, i가 홀수일 때는 cosine 함수를 사용한다.
- i가 커질수록 sine/cosine 함수의 주기가 길어진다.


![image](https://github.com/yunhyechoi/paper-review/assets/166207923/ddfaf4c8-73d2-4634-a1fc-4fc713de7d2a)

</br>

이미지를 통해 알 수 있듯이 i가 큰 경우에는 position에 따른 차이가 크지 않고, i가 작아질수록 그 차이가 커진다.
그럼 이걸로 어떻게 단어의 위치를 파악할까?
> i=4에서 해당 단어가 앞쪽에 위치하는지, 뒷쪽에 위치하는지 정도를 가늠한다. 그리고 i=3에서 앞쪽 중에서도 어느정도 위치인지 대략적으로 파악할 수 있다.
> 이런식으로 각 차원의 정보를 결합함으로써 그 범위를 좁혀간다면 해당 토큰의 위치를 확인할 수 있을 것이다.


</br>

# 참고 자료
- [위키독스 - 딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/22893)
- [NMT 시각화 영상](https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention/)
- [트랜스포머 파헤치기 - positional encoding](https://www.blossominkyung.com/deeplearning/transfomer-positional-encoding)
