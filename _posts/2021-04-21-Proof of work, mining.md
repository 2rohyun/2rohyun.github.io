---
layout: post
title:  "Nomad Coder 님 영상 요약 #2.Proof of work, mining"
summary: "Proof of work, mining"
author: 2rohyun
date: '2021-04-21 13:17:23 +0900'
category: blockchain
thumbnail: /assets/img/posts/pow_mining.png 
keywords: blockchain, proof of work, mining 
permalink: /blog/proof-of-work-and-mining/
usemathjax: false
---
# Proof of work, mining 

비트코인, 이더리움 등 작업 증명 관련된 크립토 채굴을 위해서 좋은 그래픽이 필요하다 ( = 그래픽 카드의 수요가 급증한 이유 )

어떤 데이터가 블록체인에 추가될 수 있는가? 
* `진실`인 데이터 ( 거짓이 있으면 안됨. ) 이를 위해 `작업증명`이 필요하다. 
* 작업증명을 통해 비트코인, 이더리움 등 다양한 크립토들이 사기꾼, 악용사례 등으로 부터 블록체인을 보호하고 있다.

`채굴자( miner )` : 블록체인에 들어오는 데이터를 확인하는 일을 한다.`(check)` 왜냐면 바로 채굴자가 데이터를 블록안에 넣어서 `(input)` 블록체인에 보내는 역할을 하기 때문. `(send)`

* ex ) 비트코인 블록체인에서 비트코인을 친구에게 보내면 그 거래내용은 확인되지 않는다. 하지만 채굴자가 해당 거래내역을 보고, 확인하고 친구에게 돈을 보낸 모든 내역을 체크한다. 해당 내용의 사실 체크가 끝나면 그 데이터를 블록 안에 넣고, 이러한 체크를 여러 개 해서 블록이 꽉 차면 블록을 닫고 블록체인에 올린다.

채굴자들 덕분에 블록체인에 올라가야하는 데이터들을 모두 체크할 수 있다. 이를 탈중앙화 방법으로 한다. 즉 누구나 원한다면 채굴자가 될 수 있다는 것. ( 누구나 데이터를 검증할 수 있다는 것 )

채굴자들이 채굴을 하는 이유? 돈을 주니까! 채굴자들이 트랜잭션 컨펌을 하면 돈을 받게된다. ( transfer fee 라고하는 일종의 수수료 ) 덕분에 검증 작업이 활발하게 이루어져 거짓말, 사기꾼으로부터 네트워크를 보호할 수 있는 것이다.

그렇다면 개꿀이네? 아님. 블록체인에 블록을 올리는 게 매우 x 100 `어렵다.` 아래의 이미지는 채굴자가 네트워크 질문에 대한 답을 찾는 과정을 보여준다.

![proof_of_work](/assets/img/posts/proof_of_work.png){: width="50%" height="50%"}

1. 작업증명이 채굴자에게 질문을 한다. 그것은 네트워크 전체가 다 아는 질문이다. ( send puzzle to miner )
2. 채굴자들은 그에 대한 답을 해야하고 답을 찾으면 그때서야 블록을 올릴 수가 있다. ( get it to network )
3. 채굴자에게 커미션을 준다 ( commission + bitcoin ) -> 이 때 비트코인이 생성된다. ( 사람들이 블록을 블록체인에 올리는 순간 생성 )

결국 채굴자는 보상을 두 번 받게 된다. 
1. 첫 번째 - 거래내역을 컨펌하면서, 
2. 두 번째 - 질문의 답을 찾아 블록을 체인에 올리면서 

채굴자가 블록을 체인에 올릴 때마다 coinbase transaction 이 생성되고 이 coinbase transaction 이 비트코인이 생성되는 순간이다.

원래 생성되어있는 것이 아니라, 블록을 체인에 올릴 때마다 새롭게 생성되는 것이다.

처음엔 블록을 올릴 때마다 50개의 코인이 생성되었다. 하지만 4년마다 반감기가 와서 그 생산량이 반으로 줄어든다. 50 -> 25 -> 12.5 -> 6.25 ( 현재 )

왜 반감기가 있지? 비트코인은 무한정이 아니기 때문이다. 총 2100만개의 생산량이 정해져있다.

그렇다면 채굴자들이 답해야하는 `질문`이라는 것이 무엇일까? 

![nicocoin](/assets/img/posts/nikocoin.png){: width="50%" height="50%"}

`nonce` = 한 번만 쓰인 숫자 ( 채굴자가 바꿀 수 있는 유일한 값 )

1. `0` 은 난이도를 뜻한다. 얼마나 블록을 찾는 것이 어려운지. 채굴자는 블록에 들어갈 데이터를 바꿀 수 없다. 오직 검증만 할 수 있음. 이 것은 네트워크가 채굴자에게 물어보는 "질문"에 연결된다.
2. 질문 : "채굴자님, 여기 nonce 에 넣어야하는 값이 뭘까요? 참고로 그 해시는 3개의 0으로 시작해야해요!" == "3개의 0으로 시작하는 해시를 만들려면 어떤 nonce 를 가져야하나요?"
3. 왜 3이냐? 위의 사진을 보면 난이도가 3이기 때문이다.
4. 채굴자는 nonce 를 바꾸어가며 숫자 맞추기 게임을 한다고 생각하면 된다. 노가다다. ( 3개의 0으로 시작하는 해시를 찾기 위해 )

이러한 이유로 블록 생성은 매우 어렵다. 하지만 검증하는 절차 자체는 쉽다. 이것은 작업증명의 매직! `질문`은 아주 복잡한 답변을 가지고 있지만, 답은 검증하기가 아주 쉽다!
이것이 `작업증명`에 대한 간략한 설명이다.

비트코인은 매 10분마다 새롭게 블록을 생성한다. ( 사진의 block 의 data ) 블록이 너무 빠르게 생성되몀 난이도를 올리기도 한다.

결국 네트워크가 원하는 답을 찾아 nonce 를 바꾸면 된다! 현재 비트코인은 19개의 0으로 시작해야한다고한다.

참고로 이건 비트코인의 난이도를 계산하기 위한 정확한 방법은 아니다. 0의 숫자는 난이도에 따른 결과일 뿐이고, 난이도 자체는 bits 라는 것을 통해 계산한다고 한다.

이제 이해되지? 그래픽 카드가 좋을수록 빠르게 여러 개의 nonce 를 시도해볼 수 있다! 예를 들어 6천만 논스를 1초에 계산한다.

155000000000000000000 -> 비트코인 네트워크에서 1초마다 도원되는 해시의 숫자 ^^.. 그만큼 채굴이 어렵다는 뜻이다. 10분마다 이 해시 중 하나는 nonce 를 찾아서 블록에 추가된다.

## REFERENCE

https://www.youtube.com/watch?v=ElGBP90XZWE