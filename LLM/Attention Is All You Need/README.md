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

# 참고 자료
- [위키독스 - 딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/22893)
- [NMT 시각화 영상](https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention/)
