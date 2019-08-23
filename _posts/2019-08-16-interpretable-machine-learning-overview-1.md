---
layout: post
title: "Interpretable Machine Learning 개요: (1) 머신러닝 모델에 대한 해석력 확보를 위한 방법"
date: 2019-08-16 09:00:00 +0900
author: kilho_kim
categories: [Introduction]
tags: [interpretable machine learning, interpretability, explainable artificial intelligence]
comments: true
name: interpretable-machine-learning-overview-1
---

지금까지의 포스팅을 통해, 수아랩 블로그에서는 다양한 task에 대하여 동작하는 딥러닝 모델을 직접 제작하고 학습해 왔습니다. 다만 대부분 맨 마지막 과정에서 학습이 완료된 모델을 테스트하는데, 일정 크기의 테스트 데이터셋에 대한 모델의 예측 결과를 바탕으로 정확도(accuracy)와 같이 하나의 숫자로 표현되는 정량적 지표의 값을 계산하고, 그 크기를 보아 '딥러닝 모델이 예측을 얼마나 잘 했는지'를 판단하였습니다. 여기에 덧붙여 보완적으로, 올바르게 예측하였거나 잘못 예측한 예시 결과들을 몇 개 샘플링하여 관찰하고, 모델의 예측 결과에 대한 근거를 '추측'한 바 있습니다.

하지만 단순한 정량적 지표 및 샘플링된 예측 결과들 일부를 관찰하는 것만으로, 딥러닝 모델이 예측을 수행하는 자세한 과정에 대하여 완전히 '이해'하였다고 할 수 있을까요? 또, 테스트 데이터셋에서 다뤄지지 않은 보다 특수한 상황에서도 딥러닝 모델이 늘 예측을 올바로 수행할 것이라고 충분히 '신뢰'할 수 있을까요? 딥러닝의 폭발적인 성능에 매료되어 다양한 task에 대하여 성능 향상을 끊임없이 추구해 오던 딥러닝 연구자들은, 수 년 전부터 서서히 '이해'와 '신뢰' 확보를 위한 연구로 관심을 돌리기 시작했습니다. 

이러한 목표를 보통 **해석력(interpretability)**이라는 한 단어로 표현하며, 사람의 해석이 가능하도록 하여 '이해'와 '신뢰'를 만들어 내기 위한 머신러닝 연구 분야를 **interpretable machine learning**(이하 IML)이라고 부릅니다. 본 글에서는 현재까지 머신러닝 관련 학계 전반에서 고찰하고 정립해 온 IML의 필요성, 분류 기준 및 요건 등에 대하여 언급하고, 딥러닝 모델인 신경망의 해석력 확보를 위한 대표적인 방법론들을 안내해 드리고자 합니다.

- **주의: 본 글은 아래와 같은 분들을 대상으로 합니다.**
  - 머신러닝 알고리즘의 기본 구동 원리 및 주요 머신러닝 모델(e.g. 의사 결정 나무)의 기본적인 특징을 알고 계신 분들
  - 딥러닝 알고리즘의 기본 구동 원리 및 딥러닝 모델에 대한 기초적인 내용들을 이해하고 계신 분들


## 서론

이 글을 읽고 계시는 독자 여러분들 중 대부분은, 한 번쯤 점(占)을 본 적이 있으실 거라고 생각합니다. 머신러닝 모델에 대한 '이해'와 '신뢰'의 문제를 이해하기 위해, 예전에 여러분들이 점을 봤던 기억을 잠시 떠올려 보도록 하겠습니다.

{% include image.html name=page.name file="tarot-cards.jpeg" description="점술의 예시: 타로 카드(Tarot card)를 통한 예측" class="large-image" %}

보통 점술가는 그 장르(?)에 따라 특정한 매개물(동양은 관상이나 사주, 서양은 수정구 또는 타로 카드 등)을 적절히 활용, 이를 통해 관찰된 결과에 기반하여 여러분의 현재 또는 미래 상태에 대한 예측을 수행합니다. 이 과정에서 점술가는 매개물을 통해 도출한 예측의 근거를 화려한 언변을 섞어 설명하며, 이를 듣는 여러분들은 "오, 그럴싸한데?" 라고 생각하면서 점술가의 말을 계속 들어 나갔을 것입니다.

점을 보는 과정에서 여러분이 은연 중에 중요하게 생각했던 것은 무엇이었을까요? 당연히 '예측의 정확성'을 일차적으로 중요하게 생각했을 것이나, 그 배경에는 '그럴싸한 설명'이 반드시 수반되었음을 간과하기 어려울 것입니다. 보통 활용하는 매개물이 기본적으로 상징하는 바에서 출발하여, 뭔가 이성적인 것처럼 보이는 근거를 덧붙이고, 여기에 일부 감성을 자극하는 스토리텔링까지 추가할 수록, 점술가의 예측에 대한 여러분들의 '이해'와 '신뢰'의 정도는 일반적으로 높아집니다. 심지어는 예측 결과가 틀린 것으로 밝혀졌더라도, 관대한 누군가는 (설명 과정이 너무나도 그럴싸했으므로) "다음 예측 때는 맞추겠지.." 하는 식의 반응을 보일 것입니다.

> 단적인 반례로, 온 눈과 귀를 가린 점술가가 아무런 설명 없이 여러분의 현재 신상을 하나도 틀림 없이 예측했다면, 여러분은 그 점술가를 신뢰하기보다는 "나를 뒤에서 몰래 스토킹했나?" 하는 생각에 도리어 의심할 가능성이 높습니다. 

{% include image.html name=page.name file="tarot-card-reading.gif" description="타로 카드의 기본적인 상징성에 기반한 설명<br><small>(실제 점술가들은 바로 이 지점부터 본격적으로 썰(?)을 풀기 시작하면서, 고객에게 '이해'와 '신뢰'를 심어주게 됩니다..)</small>" class="large-image" %}

정리하자면, 점술가의 예측에 대한 여러분의 '이해'와 '신뢰'를 만들어내기 위해서는, 예측 결과에 대하여 여러분이 납득할 수 있는 적절한 *설명*이 필요하다고 할 수 있겠습니다. 
  

### 머신러닝 모델에 대한 설명을 통한 해석의 필요성

그런데 흥미롭게도, 어느 머신러닝 모델의 예측 결과를 '이해'하고 '신뢰'할 수 있는가에 대한 이슈도, 점술가의 예측 결과에 대한 이해와 신뢰 가능성의 이슈와 연관지어 생각해 보면 크게 다르지 않습니다. 예를 들어, 일반화 성능을 담보하기 위해 따로 정해놓은 테스트 데이터셋에 대하여, 학습된 머신러닝 모델의 예측 정확도가 거의 100%를 달성하였다고 하더라도, '이 모델이 일반적인 상황에서도 지금과 같이 잘 작동할 것이다'는 신뢰를 사용자가 가지기 위해서는, 보통의 사용자가 납득할 만한 근거를 가지고 예측을 수행했다는 설명을 사용자에게 적절한 형태로 제공해 줘야 합니다. 

{% include image.html name=page.name file="explaining-prediction-of-model.png" description="머신러닝 모델의 예측 결과에 대한 적절한 설명의 예시 <small>(Marco T. Ribeiro et al.)</small><br><small>('Explanation'의 초록색으로 표시된 요인들은 '독감' 예측을 지지하는 근거,<br>빨간색으로 표시된 요인들은 '독감' 예측과는 반대의 근거를 나타냄)</small>" class="full-image" %}

예측 결과에 대한 적절한 설명이라고 한다면, 이를테면 위 그림에서 나온 사례를 들 수 있습니다. 환자의 여러 기본 정보 및 각종 증상 발생 여부 등을 포함하는 데이터를 기반으로 하여 해당 환자의 독감 발병 여부를 예측하는 모델이 있다고 합시다. 이 모델은 지도 학습(supervised learning)에 기반하여 학습되었기 때문에 {'독감', '독감X'} 따위의 예측 결과를 출력할 뿐, 그 과정에서 어떠한 요인들(features)을 근거로 삼았는지를 보통 직접적으로 보여 주지는 않습니다. 

모델의 예측 결과를 참조하여 진단을 해야 하는 의사의 입장에서는, 이 모델의 예측 정확도가 수치적으로 높다고는 하더라도, 예측에 대한 근거를 함께 받지 않은 상황에서 이를 마냥 신뢰하기가 어려울 것이라고 짐작할 수 있습니다. 그러나 만약 머신러닝 모델의 예측 결과의 주요한 근거가 무엇이었는지 설명을 받을 수 있다면, 의사 입장에서는 예측 결과가 납득할 만한 과정을 거쳐 도출되었다는 것을 확인함으로써 해당 예측 결과에 대한 신뢰를 가질 수 있습니다. 물론 이는 진단 결과를 전달받게 되는 환자의 입장에서도 마찬가지로 적용됩니다. 위 사례와 같이, 상용화 및 대중화를 염두에 두고 있으면서 머신러닝 기술이 핵심이 되는 제품을 출시하고자 할 수록, 이러한 적절한 설명의 중요성은 더 높아질 것이라고 짐작할 수 있습니다. 

한편, 머신러닝 모델의 예측 결과에 대한 설명이 제공될 경우, 단순히 제품 사용자에게 신뢰를 주는 것 외에도 다양한 상황에서 효용을 가져올 수 있습니다. 한 가지 예로, 머신러닝 모델을 개발하는 과정에서, 해결하고자 하는 task의 핵심과 무관한 부분을 모델이 집중적으로 관찰하여 예측하는지에 대한 '*디버깅(debugging)*'을 수행함으로써, 모델의 일반화 성능 향상을 유도할 수 있습니다. 이러한 상황은 생각보다 빈번하게 발생하는데, 학습 데이터셋 상에 사람이 인지하기 어려운 편향(bias)이 존재할 경우 여기에 기인하여 발생합니다.

{% include image.html name=page.name file="explaining-email-classification.gif" description="이메일 분류 문제에서의 예측 결과에 대한 설명 제시 사례 <small>(Marco T. Ribeiro et al.)</small><br><small>(좌측: Algorithm 1, 이메일의 제목 및 내용을 보고 분류;<br>우측: Algorithm 2, 이메일의 제목/내용과 무관한 헤더를 보고 분류)</small>" class="full-image" %}

위 그림은 어느 주어진 이메일에 대하여, 그 주제가 'Christianity(기독교)' 또는 'Atheism(무신론)' 중 무엇에 대한 것인지 분류하도록, 서로 다른 2개의 알고리즘을 적용한 머신러닝 모델을 학습한 뒤 테스트한 결과를 하나 보여주고 있습니다. 실제 클래스가 'Athiesm'인 어느 이메일에 대하여, Algorithm 1을 적용한 모델과 Algorithm 2를 적용한 모델은 둘 모두 'Athiesm'으로 예측하였고, 정/오답 측면에서 보면 둘 모두 정답을 맞혔다고 할 수 있습니다. 

그러나 이 때 각 모델이 이메일 상의 어느 부분에 집중하여 예측을 수행했는지에 대한 설명이 제시되면, 두 모델 간의 우열이 명확하게 갈립니다. 학습된 모델로 하여금 사용자가 기대했던 것은 이메일의 제목 또는 내용을 보고 이메일의 주제를 분류하는 것이었을 거고, (비록 예측 결과는 동일하게 나왔더라도) 이를 제대로 수행한 것은 Algorithm 1을 적용한 모델임을 확인할 수 있습니다.

심지어, 머신러닝 모델을 적절하게 설명하는 것은 '*인류의 새로운 발견과 지식 축적*'을 위해서도 도움을 줄 수 있습니다. 사람의 어느 특정 작업에 대한 입력값 및 기대 출력값만을 주입하여 학습시킨 모델에 대하여, 그 행동 방식에 대한 설명을 함께 받을 수 있다면, 해당 작업을 해결하는 데 있어 사람이 그 전까지 알지 못했던 새로운 방법 또는 사실을 발견하게 될 것이고, 그것이 거듭될 수록 다양한 방면에서의 인류의 지식을 점진적으로 증진시키는 결과를 기대할 수 있습니다. 이건 좀 너무 나간 주장이 아닌가 싶으실텐데, 이를 입증할 만한 사례는 가까운 곳에서 찾을 수 있습니다. 지난 2016년 등장한 바둑 두는 기계인 '알파고(AlphaGo)'와 한국의 이세돌 9단의 대국 이벤트를 생각해 보면 됩니다. 

{% include image.html name=page.name file="alphago-vs-lee-sedol.jpg" description="알파고의 수를 복기하면서 힘겨워하는 이세돌 9단" class="large-image" %}

알파고는 단지 딥러닝(정확히는 딥 강화 학습)에 기반하여, 매 차례 현재의 바둑판의 상태를 읽어들인 뒤 최적의 다음 착수 위치만을 출력하도록 학습되었기 때문에, 단순히 그 출력 결과들만을 보고 사람이 그 속에 담긴 '전략' 내지는 '의도'를 파악해 내는 것은 매우 어려운 일입니다. 이러한 이유 때문에, 이세돌 9단도 거듭되는 패배 속 지난 대국에 대한 복기를 하는 과정에서 깊은 어려움을 토로한 바 있습니다.

그러나 착수 결과에 대한 적절한 설명이 부재하였음에도 불구하고, 이후 바둑계에서는 알파고의 수들에 대한 나름의 분석들을 통해 기존과는 다른 새로운 패러다임을 서서히 정립해 나가기 시작하였고, 심지어는 바둑 대회의 규칙 및 프로 바둑 기사의 역할 등에 있어서도 변화를 유발하는 계기가 되었다고 합니다(<a href="https://news.joins.com/article/21352123" target="_blank">관련 기사</a>). 만약 알파고의 착수 결과에 대하여 바둑 기사들이 이해할 수 있는 설명이 제공되었다면(이를테면, 무슨 의도로 해당 수를 두었는지), 바둑계의 이러한 변화는 더 크고 빠르게 찾아왔을지도 모릅니다.


### 머신러닝 모델에 대한 설명의 어려움  

그러나 문제는, <a href="{{ site.url }}{% post_url 2017-09-04-what-is-machine-learning %}#러닝-모델" target="_blank">머신러닝 모델의 함수적 특성</a> 상, 어느 입력값이 주어지면 그 입력에 부합하는 최적의 출력값을 산출하는 데에만 집중하여 내부 수식이 변화할 뿐, 그 과정에서 일반 사용자의 눈에 그럴싸한 설명을 제공하기 위한 그 어떠한 노력(?)도 거치지 않는다는 것입니다(함수는 단지 숫자를 받아 숫자를 출력할 뿐입니다). 특히, 보다 '깊은' 딥러닝 모델 구조로 갈 수록 함수 자체는 점점 더 복잡해지게 되고, 그것의 예측 결과에 대한 그럴싸한 설명은 더욱 어려워지는 경향을 보입니다. 머신러닝 모델의 이러한 특징을 흔히 'black box'라고 표현하며, 이러한 맥락에서 머신러닝 모델을 'black box model'로 부르기도 합니다.

{% include image.html name=page.name file="deep-neural-network-as-black-box.png" description="딥러닝 모델의 black box적 속성" class="large-image" %}

> 다시금 점(占)에 비유하자면, 딥러닝 모델은 '예측 능력은 아주 우수한데, 설명 능력이 최악인 점술가'라고 할 수 있겠습니다.

 이러한 black box적인 속성을 지니는 머신러닝 모델을 잘 설명하기 위한 방법이 다양한 각도에서 연구되어 왔으며, 몇 년 전부터는 IML이라는 이름의 분야로 서서히 정립되기 시작했습니다. 지금부터는 이를 자세히 들여다 보고자 합니다.


## IML의 분류 기준

불과 몇 년 전까지만 하더라도 머신러닝 모델의 해석력에 대한 정의가 하나로 정립되지 않았었고 IML에 대한 연구 또한 파편화되어 진행되어 온 경향이 있었습니다. 그러나 2010년대 중반에 접어들면서 IML 분야에 대하여 학술적으로 정립해 나가려는 시도가 하나둘씩 등장하기 시작하였고, 일정한 기준에 따라 한 단계 상위 레벨에서 IML 방법론들을 분류하려는 시도를 하게 되었습니다. 

IML 방법론들에 대한 분류 기준들에 대해 이해하게 되면, 머신러닝 모델에 대한 해석력 확보를 위해 어떠한 맥락에서 고민이 이루어졌으며, 어떠한 방법으로 이를 달성하였는지에 대하여 이해하는 데 도움을 얻을 수 있습니다. 그리고 이를 기반으로 새로운 IML 방법론이 등장했을 때 이를 어떻게 활용할 수 있을지 효과적으로 계획할 수 있습니다. 본 글에서는 근 1-2년 내에 발표된 IML 관련 서적 및 survey 논문(e.g. Amina Adadi et al.) 등에서 가장 지배적으로 채택되는 기준들을 소개하였습니다.

### Intrinsic vs. Post-hoc

머신러닝 모델의 복잡성(complexity)은 해석력과 깊은 연관이 있습니다. 좀 더 정확하게는, 둘 간에 일종의 tradeoff가 존재합니다. 머신러닝 모델의 복잡성이 낮아지면서 단순한 구조를 가질수록 그 자체에 대한 사람의 해석이 용이해지는 경향이 있으며, 반대로 복잡성이 높아지면서 점점 복잡한 구조를 가지게 될 수록 이에 대한 사람의 해석은 난해해집니다.

낮은 복잡성을 보여주는 대표적인 머신러닝 모델이 바로 **의사 결정 나무(decision tree)**입니다. 의사 결정 나무에서는 몇몇 요인들의 값을 기준으로 하여 둘 이상의 가지들로 분기하면서 뻗어 나가는 형태를 지니는데, 예측 결과를 도출하게 된 과정에 대한 해석이 그 자체로 매우 직관적이고 용이하다는 장점이 있습니다. 

{% include image.html name=page.name file="decision-tree-example.jpg" description="가장 간단한 의사 결정 나무 예시<br><small>(어느 예측 결과에 해당하는 마디를 따라 올라가면서, 그 근거를 손쉽게 확인할 수 있음)</small>" class="medium-image" %}

이런 의사 결정 나무의 경우 그 자체적으로 해석력을 이미 확보하고 있다고 볼 수 있으며, 이러한 머신러닝 모델은 '**intrinsic**(본래 갖추어진)'하다고 지칭합니다. 그 외에도 기존의 <a href="{{ site.url }}{% post_url 2017-09-04-what-is-machine-learning %}#선형-모델" target="_blank">선형 모델(linear model)</a>에 <a href="https://en.wikipedia.org/wiki/Lasso_(statistics)" target="_blank">Lasso</a> 등의 요인 선택(feature selection) 기법을 적용하여 얻어진 **희소 선형 모델(sparse linear model; <small>취사 선택된 일부 요인들에 대해서만 0이 아닌 가중치가 존재</small>)**, 각 요인들에 대한 일련의 if-else 규칙들로 정의된 **규칙 리스트(rule lists)** 또한 해석이 근본적으로 용이하다는 측면에서 intrinsic 모델로 분류할 수 있습니다. 

{% include image.html name=page.name file="rule-lists-example.png" description="규칙 리스트 예시: Falling rule lists <small>(Fulton Wang and Cynthia Rudin)</small>" class="large-image" %}

반면, 복잡성이 극도로 높은 전형적인 머신러닝 모델로는 **신경망(neural network)**, 즉 딥러닝 모델이 있습니다. 신경망은 내부적으로 복잡한 연결 관계를 지니고 있으며, 이로 인해 예측 결과를 도출하는 과정에서 하나의 입력 성분값(이미지 데이터의 경우 픽셀 1개의 값)이 어떻게 기여했는지 의미적으로 해석하는 것이 대단히 어렵습니다. 당연하게도, 신경망의 은닉층(hidden layer)의 수가 많아질수록 그 복잡성은 높아지며, 해석력 확보는 어려워집니다.

이렇게 복잡성이 높은 머신러닝 모델에 대하여 적용할 수 있는 대안으로, 비교적 간단한 형태를 지니는 별도의 '설명용 모델(interpretable model)'을 셋팅하고(e.g. 의사 결정 나무, 희소 선형 모델, 규칙 리스트 등) 이를 설명 대상이 되는 머신러닝 모델에 적용하는 방법을 시도할 수 있습니다. 설명용 모델의 예측 결과는 원본 머신러닝 모델을 모방하도록 하되, 복잡성은 가능한 한 낮추어 해석력을 확보하기 위한 전략으로, 이를 '**post-hoc**(사후적 설명)' 방법이라고 지칭합니다. Post-hoc 방법의 경우 곧 설명할 model-agnostic 방법과 밀접한 연관이 있기 때문에, 뒤에서 좀 더 자세히 알아보도록 하겠습니다. 

{% include image.html name=page.name file="post-hoc-methods.png" description="Post-hoc 방법에 기반한 설명용 모델과 원본 머신러닝 모델 간의 관계<br><small>(파란색으로 표시된 'A->B' 관계의 경우, B가 A로부터 학습을 수행한다는 것을 표현함)</small>" class="large-image" %}

머신러닝 모델의 복잡성과 해석력 간에는 tradeoff가 존재한다고 하였는데, 오로지 해석 가능성 향상을 위해 마냥 복잡성이 낮은 intrinsic 모델을 사용하는 것은 실제 예측 성능의 측면을 고려하면 좋은 선택이 아닙니다. 이미지 인식 등과 같이 복잡성이 낮은 머신러닝 모델로는 해결이 어려운 task가 존재하며, 이러한 경우에는 해석력을 일부 희생하면서라도 복잡성이 높은 딥러닝 모델을 사용하여 예측 성능 자체를 높이는 데 집중하는 것이 더 나은 대안일 수 있습니다. 다시 말해, 해결하고자 하는 task에 따라서, 그것에서 요구되는 예측 성능의 최소 기대 수준과 더불어, 예측 결과에 대해 필요한 해석력의 정도 등을 종합적으로 고려하여 적절한 머신러닝 모델을 선택해야 합니다.

### Model-specific vs. Model-agnostic

IML 방법론이 어느 특정한 종류의 머신러닝 모델에 특화되어 작동하는지, 혹은 모든 종류의 머신러닝 모델에 범용적으로 작동하는지에 따라 이들을 분류할 수도 있습니다. 전자의 경우 '**'model-specific**(모델 특정적)', 후자의 경우 '**model-agnostic**(모델 불가지론적)' 방법이라고 지칭합니다. 

앞서 언급했던 의사 결정 나무 등과 같이 *intrinsic한 속성을 지니는 머신러닝 모델은, 그 자체가 본질적으로 model-specific한 속성을 동시에 지닙니다*. 다시 말해, 이러한 머신러닝 모델들은 그 자체를 통해 자연스럽게 제공될 수 있는 특수한 형태의 설명을 본질적으로 갖추고 있다고 할 수 있습니다. 

반면 신경망과 같이 복잡성이 높아 자체적인 해석력을 확보하기 어려운 머신러닝 모델의 경우, 별도의 post-hoc 방법을 통해 설명용 모델을 생성하고 이를 활용한다고 하였습니다. 현재 나와 있는 post-hoc 방법들의 경우, 그 적용 대상 머신러닝 모델의 종류와 무관하게 범용적으로 작동하도록 디자인되어 있는 경우가 대부분에 해당합니다. 다시 말해, *post-hoc 특성을 지닌 IML 방법론은 곧 model-agnostic하다*고 해도 크게 무리가 없습니다. 

정리하면, intrinsic과 model-specific, post-hoc과 model-agnostic은 서로 간의 관점에 차이가 있을 뿐, 실질적으로는 동시적으로 적용될 수 있는 특성이라고 봐도 크게 무리가 없다고 할 수 있겠습니다.

### Local vs. Global

어느 머신러닝 모델의 모든 예측 결과에 대하여, IML 방법론이 그럴싸한 설명을 빠짐 없이 제시할 수 있는 경우, 해당 모델을 해석하는 데 있어 가장 이상적일 것입니다. 이렇게 어느 IML 방법론이 머신러닝 모델의 예측 결과들에 대하여 '전역적으로(globally)' 완벽하게 설명을 수행할 수 있는 경우, 이를 '**global**(전역적)' 방법이라고 지칭합니다.

반면 설명 대상 머신러닝 모델의 복잡성이 증가할수록, 단일 IML 방법론이 모든 예측 결과에 대하여 그럴싸한 설명을 제시하는 것이 점점 어려워집니다. 이 때문에 몇몇 IML 방법론들은 완벽하게 '전역적인' 설명을 포기하는 대신, 모델의 어느 예측 결과에 대하여 적어도 그와 유사한 양상을 나타내는 '주변'  예측 결과들에 한해서는 '국소적으로(locally)' 그럴싸한 설명을 제시할 수 있도록 디자인되었습니다. 이를 '**local**(국소적)' 방법이라고 지칭합니다.

{% include image.html name=page.name file="local-explanations.png" description="IML 방법의 local한 설명을 표현하는 그림 <small>(Marco T. Ribeiro et al.)</small><br><small>(위: 이진 분류(binary classification) task에서 원본 머신러닝 모델의 전체 예측 결과를 나타낸 공간, <br>아래: Local한 설명용 모델(갈색 점선)들이 원본 머신러닝 모델의 전체 예측 결과들 중 각각 일부분만을 커버하도록 셋팅된 결과)</small>" class="large-image" %}

위 그림에서 볼 수 있듯이, local한 IML 방법은 원본 머신러닝 모델의 전체 예측 결과 중 일부 영역들만을 커버할 수 있도록 작동합니다. 좀 더 구체적으로, 어느 하나의 테스트 이미지에 대하여 원본 머신러닝 모델이 예측한 하나의 결과를 설명하고자 할 때, (원본 머신러닝 모델의 입장에서) 해당 이미지와 유사하게 인식한 다른 몇몇 테스트 이미지의 예측 결과들에 대해서도 그럴싸한 설명을 제공할 수 있는 설명용 모델을 즉석에서(ad-hoc) 제시합니다. 

오늘날 신경망과 같이 복잡성이 높은 머신러닝 모델을 사용하는 일반적인 상황에서, 예측 결과에 대하여 전역적으로 완벽한 설명을 제시하는 것은 현실적으로 매우 어려운 일입니다. 비록 머신러닝 모델의 전체 예측 결과에 대하여 완벽한 설명을 한 번에 제시하는 것은 불가능하더라도, 적어도 사용자가 관심을 가지는 몇 개의 예측 결과에 한하여 즉각적으로 그럴싸한 설명을 제시해 줄 수 있다는 측면에서, local한 IML 방법들은 설명의 실용성 측면에서 각광을 받고 있습니다. 

마지막으로, intrinsic(model-specific) vs. post-hoc(model-agnostic), local vs. global 기준에 의거하여, 현재까지 보고된 주요 IML 방법론들을 플롯팅해 본 결과를 아래와 같이 제시하였습니다.  

{% include image.html name=page.name file="iml-methods-plot.png" description="Intrinsic(Model-specific) vs. Post-hoc(Model-agnostic),<br>Local vs. Global 기준에 의거한 IML 방법론의 플롯팅 및 그룹화 결과 <small>(Amina Adadi et al.)</small>" class="large-image" %}

위 플롯에서 확인할 수 있듯, intrinsic(model-specific)한 IML 방법은 그 자체가 예측 및 설명 모두를 위해 직접적으로 사용되는 경우는 드물며, 그 대신 신경망과 같이 복잡성이 높은 머신러닝 모델을 예측을 위해 먼저 사용한 뒤, 여기에 post-hoc(model-agnostic)한 IML 방법을 부가적으로 적용하는 방식이 많이 채택됩니다. 또, post-hoc한 IML 방법들 중 global한 것보다는 local에 가까운 방법들이 더 많이 보고되었는데, 이는 복잡성이 높은 머신러닝 모델에 대한 효과적인 설명을 위해 국소적으로나마 그럴싸한 설명을 제시할 수 있도록 하는 데 집중한 결과로 볼 수 있습니다.


## 딥러닝을 위한 IML

지금까지 머신러닝 영역 전반에 대하여 연구되어 온 IML 방법론들을 몇 가지 주요한 기준에 의거하여 그룹 지어 소개하였습니다. 각 그룹에 포함되는 IML 방법론들에 대한 구체적인 설명은 글의 분량 관계 상 의도적으로 생략하였는데, 그 대신 여러분들이 더 많은 관심을 가지고 계실 딥러닝 모델에 대한 주요 IML 방법론들에 대하여 좀 더 구체적으로 소개해 드리고자 합니다.

- Amina Adadi et al.의 분류 체계를 따름
  - 보는 시각에 따라, 신경망을 위한 IML 방법론을 model-specific 또는 model-agnostic으로 보는 견해가 혼재하고 있으며, 서로 충돌하는 것이 사실임. 
    - 보다 엄밀하게, 여기에서 소개되는 IML 방법론들은 주로 범용성을 염두에 두고 model-agnostic하게 활용될 수 있도록 디자인된 것들이나, 실제로는 신경망에 초점을 맞춰 다소 model-specific하게 검증된 IML 방법론들이라고 할 수 있음.
    - 대부분 post-hoc이자 local한 방법론이라고 할 수 있음.

---

Criteria 1: Amina Adadi et al.'s examples (somewhat model-agnostic)

### Visualization

- Surrogate models
- Partial dependence plot (PDP)
- Individual conditional expectation (ICE)

### Knowledge Extraction 

- Rule extraction
- Model distillation

### Influence Methods

- Sensitivity analysis
- Layer-wise relevance propagation (LRP)
- Feature importance

### Example-based Explanation

- Prototypes and criticisms
- Counterfactuals explanations

---

Criteria 2: Christopher Molnar's examples (somewhat model-specific)

- Feature visualization
- Concepts
- Feature attribution
- Model distillation



## IML 실현과 관련된 현실적 이슈

### '설명'에 대한 평가 방법

- 참조: Amina Adadi et al. > XAI Measurement: Evaluating Explanations
  - Application-grounded: put the explanation into the application and let the end user (typically a domain expert) test it
  - Human-grounded: conduct simplified application-grounded evaluation, where experiments are run with lay humans (rather than domain experts)
  - Functionally-grounded: Does not involve humans


### '사람'을 고려한 '설명' 방법

- "How to produce explanations that simulate the human cognitive process" - 3가지 강조 사항
  - Contrastive
    - why-questions
  - Selective
    - in a biased manner
  - Social
    - Social conversation and interaction, for transfer of knowledge


## 결론

*TODO: 내용 작성*


## References

- Amina Adadi and Mohammed Berrada, Peeking inside the black-box: a survey on explainable artificial intelligence (XAI).
    - <a href="https://ieeexplore.ieee.org/iel7/6287639/6514899/08466590.pdf" target="_blank">Adadi, Amina, and Mohammed Berrada. "Peeking inside the black-box: A survey on Explainable Artificial Intelligence (XAI)." IEEE Access 6 (2018): 52138-52160.</a>
- Riccardo Guidotti et al., A survey of methods for explaining black box models.
    - <a href="https://dl.acm.org/ft_gateway.cfm?ftid=1997533&id=3236009" target="_blank">Guidotti, Riccardo, et al. "A survey of methods for explaining black box models." ACM computing surveys (CSUR) 51.5 (2018): 93.</a>
- Finale Doshi-Velez and Been Kim, Towards a rigorous science of interpretable machine learning.
    - <a href="https://arxiv.org/pdf/1702.08608" target="_blank">Doshi-Velez, Finale, and Been Kim. "Towards a rigorous science of interpretable machine learning." arXiv preprint arXiv:1702.08608 (2017).</a>
- Zachary C. Lipton, The mythos of model interpretability.
    - <a href="https://arxiv.org/pdf/1606.03490" target="_blank">Lipton, Zachary C. "The mythos of model interpretability." arXiv preprint arXiv:1606.03490 (2016).</a>
- Been Kim et al., Examples are not enough, learn to criticize! criticism for interpretability.
    - <a href="http://papers.nips.cc/paper/6300-examples-are-not-enough-learn-to-criticize-criticism-for-interpretability.pdf" target="_blank">Kim, Been, Rajiv Khanna, and Oluwasanmi O. Koyejo. "Examples are not enough, learn to criticize! criticism for interpretability." Advances in Neural Information Processing Systems. 2016.</a>
- Marco T. Ribeiro et al., Why should I trust you? Explaining the predictions of any classifier.
    - 이메일 분류 문제에서의 예측 결과에 대한 '설명' 제시 사례
    - <a href="https://arxiv.org/pdf/1602.04938.pdf?mod=article_inline" target="_blank">Ribeiro, Marco Tulio, Sameer Singh, and Carlos Guestrin. "Why should i trust you?: Explaining the predictions of any classifier." Proceedings of the 22nd ACM SIGKDD international conference on knowledge discovery and data mining. ACM, 2016.</a>
- 점술의 예시: 타로 카드(Tarot card)를 통한 예측
    - <a href="https://medium.com/@aritrin/all-about-tarot-everything-you-need-to-know-about-tarot-card-reading-60f1d03b675b" target="_blank">Ari Tri Nugroho. "All About Tarot: Everything You Need to Know about Tarot Card Reading." Medium, https://medium.com/@aritrin/all-about-tarot-everything-you-need-to-know-about-tarot-card-reading-60f1d03b675b. Accessed 6 August 2019.</a>
- 타로 카드의 기본적인 상징성에 기반한 설명
    - <a href="http://giphygifs.s3.amazonaws.com/media/5fBH6ztSjOdRgr46IWk/giphy.gif" target="_blank">Geek & Sundry, "Web series death gif" GIPHY, https://giphy.com/gifs/geekandsundry-lol-ashley-johnson-spooked-5fBH6ztSjOdRgr46IWk. Accessed 6 August 2019.</a>
- 알파고의 수를 복기하면서 힘겨워하는 이세돌 9단
    - <a href="http://news.chosun.com/site/data/html_dir/2016/03/11/2016031102984.html" target="_blank">차정승 기자, "이세돌 2연패 후 '밤샘 복기'…호텔방서 칩거하며 '알파고 파기 비법' 연구." 조선닷컴, http://news.chosun.com/site/data/html_dir/2016/03/11/2016031102984.html. Accessed 8 August 2019.</a>
- 알파고 등장 이후의 바둑 패러다임의 변화 기사
    - <a href="https://news.joins.com/article/21352123" target="_blank">정아람 기자, "알파고 쇼크 1년 ... 바둑 패러다임이 달라졌다." 중앙일보, https://news.joins.com/article/21352123. Accessed 8 August 2019.</a>
- 가장 간단한 의사 결정 나무 예시
    - <a href="https://becominghuman.ai/understanding-decision-trees-43032111380f" target="_blank">Egor Dezhic, "Understanding Decision Trees." Medium, https://becominghuman.ai/understanding-decision-trees-43032111380f, Accessed 20 August 2019.</a>
- Lasso
    - <a href="https://en.wikipedia.org/wiki/Lasso_(statistics)" target="_blank">Wikipedia contributors. "Lasso (statistics)." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 13 Aug. 2019. Web. 20 Aug. 2019. </a>
- Rule lists
    - <a href="http://www.jmlr.org/proceedings/papers/v38/wang15a.pdf" target="_blank">Wang, Fulton, and Cynthia Rudin. "Falling rule lists." Artificial Intelligence and Statistics. 2015.</a>