---
layout: post
title:  "Nomad Coder 님 영상 요약 #3.Smart contract, etherium"
summary: "Smart contract, etherium"
author: 2rohyun
date: '2021-04-21 11:17:23 +0900'
category: blockchain
thumbnail: /assets/img/posts/smart_contracts.png 
keywords: blockchain, Smart contract, etherium
permalink: /blog/what-is-blockchain/
usemathjax: false
---
# Smart contract, etherium

스마트 컨트랙트는 이 때까지 들어본 모든 엄청난 프로젝트의 주춧돌 같은 역할이다. NFT, ICO ( Initial Coin Offering ), DEX ( Decentralized exchanges ) etc ..

비트코인이 지루한 이유? 다른 사람과 교류할 수 없기 때문이다. 비트코인을 주고받는 것 외에 할 수 있는 것이 없다.

스마트 컨트랙트를 통하면 다른 사람과 교류할 수 있고, 또한 코드로 소통할 수 있다. 그 중 하나는 이를 통해 `탈중앙화 앱`을 만들 수 있다는 것이다. 

뭔 뜻이냐? 개발자로서 코딩을 해서 이것을 `공유 네트워크`에 올릴 수 있다. ( 전체 네트워크가 검증하고 실행하는 네트워크에 )

보통 코딩해서 AWS 에 올리잖아? 근데 Amazon server 가 터지면? 내 애플리케이션도 사라진다. 

하지만 내 코드를 공유 네트워크 ( smart contract platform ) 에 올리면? 그 곳에 영원히 있는거야. 사람들은 볼 수 있지만 변경할 수는 없다.

> 정리하자면, 전세계 사람들이 쓰는 어마어마한 공유 네트워크에 내가 코딩을 해서 그 코드를 주인이 없는 백엔드에 올린다. 이것이 어마어마한 블록체인의 안정성을 구축한 채 나의 코드를 검증하고 실행하게 된다. 

무엇을 만들 수 있나? 예를 들면 너만의 은행 계좌를 만들 수 있다. 여기서 두 가지 방침이 있는 스마트 컨트랙트를 사용한다.

![smartcontract](/assets/img/posts/smart_contract.png){: width="50%" height="50%"}

1. 돈을 저축하고 거기에 보관한다.
2. 시간이 흐른 30년 후의 나에게 돌려준다.
이 것을 은행의 도움 없이 스마트 컨트랙트가 할 수 있다. 스마트 컨트랙트를 사용하여 블록체인에 올린다. 그럼 잘 보관되어 있겠지? 아무도 수정을 할 수 없으니까! 차곡차곡 스마트 컨트랙트를 통해 30년동안 저축을 하고, 시간이 흘러서 코드가 실행되고 그 돈은 다시 너에게 돌아올 것이다.

한 가지 예를 더 들어보자. 에어비앤비!
방의 주인과 방을 빌리는 사람은 에어비앤비를 중개인으로하여 방을 빌려주고, 빌린다. 방의 주인과 방을 빌리는 사람 모두 에어비앤비를 신뢰하고 있다. 이것을 스마트 컨트랙트로 대체할 수 있다.

![smartairbnb](/assets/img/posts/smart_airbnb.png){: width="50%" height="50%"}

1. 방을 빌리려는 사람은 에어비앤비 대신 스마트 컨트랙트에 돈을 보낸다.
2. 스마트 컨트랙트는 아두이노와 같은 IoT 와 같은 기술을 활용하여 대문이나 TV 같은 것들과 연결한다.
3. 스마트 컨트랙트는 TV 를 켜고, 문을 열어준다. 정해진 기간동안!

스마트 컨트랙트의 단점? 원하는 소스를 모두 활용할 수 없다. 결국 해당 ( 블록체인 ) 네트워크 위에서만 실행이 가능하다.
![smartlimit](/assets/img/posts/smart_limit.png){: width="50%" height="50%"}

ex ) 친구와 내가 내일 비가 온다고 내기를 한다. 스마트 컨트랙트와 비 감지 센서를 연결하고 비가 온다면 돈을 따는 것이다. 문제는 스마트 컨트랙트는 비 감지 센서를 믿어야한다는 것이다. 이 거래는 "신뢰 기반" 의 거래가 되는 것이다. 탈중앙화 네트워크 외부의 무언가에 의존, 혹은 신뢰를 해야한다는 것.

따라서 이러한 단점 때문에 누군가가 조작할 수 있는 인풋은 허용할 수 없는 신뢰 기반이 아닌 특수한 네트워크, 환경을 요구하게 된다. 

![smartoracle](/assets/img/posts/smart_oracle.png){: width="50%" height="50%"}

이 단점을 보완하기 위해서 "oracle" 이라는 것이 존재한다. oracle 은 스마트 컨트랙트에 인풋을 제공하는데, 신뢰 기반의 데이터를 이러한 특수 네트워크에 가져온다.

> 총 정리 : 스마트 컨트랙트는 나의 코드를 가져다 모두가 공유, 검증, 실행하지만 수정은 할 수 없는 ( 블록체인의 특징 ) 백엔드에 올린다. 덕분에 중개인이 필요없게 되고 규칙을 빠짐없이 실행할 스마트 컨트랙트를 만들고, 정부나 단체에 조종 당할 염려없이 네트워크에 의하여 빈틈없이 실행된다.

스마트 컨트랙트를 지원하는 블록체인 : etherium, kusama, polkadot, cardano, cosmos etc..

## REFERENCE

https://www.youtube.com/watch?v=3I5_D-deQT0
