---
layout: post
title:  "Nomad Coder 님 영상 요약 #5.Crypto currency wallet"
summary: "Crypto currency wallet"
author: 2rohyun
date: '2021-04-21 11:17:23 +0900'
category: blockchain
thumbnail: /assets/img/posts/wallet.png 
keywords: blockchain, crypto currency, wallet
permalink: /blog/crypto-currency-wallet/
usemathjax: false
---
# Crypto currency wallet 

비트코인을 전송한다는 건, 이메일이나 우편을 전송하는 것과는 다르다. 사실 아무 것도 전송하지 않는다.

왜냐면 사실 비트코인을 소유한다고 하면, 실제로 비트코인을 `소유` 하는 것이 아니기 때문이다. 비트코인이 네트워크에서 전송되고 하는 것이 아니다. 

비트코인은 비트코인 네트워크에 항상 언제나 끝까지 있다. 애초에 비트코인이 네트워크를 떠나지 않는다.

따라서 비트코인이 네트워크를 떠나지 않기 때문에 지갑에 넣는 것도 불가능하다.

그렇다면 어떻게 비트코인을 소유하는 걸까? 비트코인을 소유한다는 것의 의미는 무엇일까?

예를 들어, 집을 산다고 가정하면 집의 주소는 바뀌지 않는다. 그 집의 소유자가 누구인지에 대한 집문서가 바뀔 뿐이다.

비트코인과 같은 암호화폐도 마찬가지이다. 동일한 작업을 비트코인 DB 에서 한다.

비트코인의 지갑은 엄청나게 긴 텍스트이다. 특정 사람에게 비트코인을 보낼 때 필요한 것이 이 지갑이다.

따라서 영원히 잃어버린 비트코인의 경우 대부분 비트코인 지갑을 잘못 저장한 경우이다.

그렇다면 지갑은 어떻게 생성하는가?

지갑은 비대칭적 암호화 ( asymmetric encryption ) 을 이용하여 생성된다.

그렇다면 대칭적 암호화 ( symmetric encryption ) 이란?

![symmetric1](/assets/img/posts/symmetric1.png){: width="50%" height="50%"}

1. Sender 가 Receiver 에게 편지를 보내고싶어 안전한 금고에 넣어서 자신의 자물쇠를 채운 뒤 보낸다.
2. Receiver 는 자물쇠를 열 수 있는 key 가 없으므로 Sender 가 key 도 함께 보내줘야 한다.
3. 마찬가지로, Receiver 가 답장을 하려면 편지를 금고에 넣어서 Sender 의 자물쇠로 잠군 뒤, Sender 의 key 를 다시 보내줘야 한다.

이런 방식은 안전하지 않다. 한 개의 열쇠를 서로 주고 받아야하기 때문이다. 그래서 필요한 것이 `asymmetric encryption` 이다.

![asymmetric1](/assets/img/posts/asymmetric1.png){: width="50%" height="50%"}

이 방식에서는 각자 키를 2개씩 갖으므로 총 4개의 열쇠가 있다. 각자 public key, private key 가 있다. public key 를 자물쇠라고 생각해보자. 그리고 private key 를 그 자물쇠를 여는 진짜 key 라고 생각하자.

![asymmetric2](/assets/img/posts/asymmetric2.png){: width="50%" height="50%"}

![asymmetric3](/assets/img/posts/asymmetric3.png){: width="50%" height="50%"}

1. Person A 가 Person B 에게 편지를 보낼 때, Person B 의 자물쇠 ( person b's public key ) 를 요청한다.
2. 보낼 편지와 나의 자물쇠 ( person a's public key ) 를 모두 금고에 넣은 뒤, Person B 의 자물쇠로 잠군다.
3. Person B 는 자신의 열쇠 ( person b's private key ) 로 자물쇠를 열 수 있다.
4. Person B 는 답장과 자신의 자물쇠를 금고에 넣고 Person A 의 자물쇠로 잠구어 보낸다.

이 private/public key 방식이 암호회폐 세상에서의 `지갑`이다.

지갑을 만든다는 뜻은 private key 를 만든다는 것이고, private key 로 public key 를 생성한다. 그리고 긴 텍스트로 된 public key 가 비트코인을 보낸다고할 때에 공유해야할 주소인 것이다.

또한, 비트코인을 쓰거나, 누군가에게 보내고 싶다면 그 때는 항상 private key 가 필요하다. private key 가 있어야 public key 를 열 수 있으니까!

따라서 private key 는 비트코인 네트워크에서 해당 비트코인이 내 것임을, 주인임을 증명할 수 있는 열쇠인 것이다.

## REFERENCE

https://www.youtube.com/watch?v=4uBWSPw-Vd8
 