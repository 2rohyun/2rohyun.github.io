---
layout: post
title:  "Nomad Coder 님 영상 요약 #1.Concept of Blockchain"
summary: "what is blockchain"
author: 2rohyun
date: '2021-04-21 11:17:23 +0900'
category: blockchain
thumbnail: /assets/img/posts/blockchain.png 
keywords: blockchain, what is blockchain
permalink: /blog/what-is-blockchain/
usemathjax: false
---
# Concept of Blockchain

* 블록체인은 먼저 요즘 핫한 bitcoin, etherium, cardano, polkadot, cosmos, dogecoin 등을 돌아가게 하는 기반이다.

* 블록체인 = 블록들이 모여있는 체인이다. 

* 그렇다면 `블록`이란? 간단하게 말한다면, 블록은 추가가 가능하고 편집은 불가능한 DB 이다.

* 블록체인은 `탈중앙화 ( Decentralize )` 가 가능하다. 여기서 탈중앙화란, 특정 개인이 DB 를 관리할 수 없는 것을 뜻한다.

* 모두가 DB의 복제본을 가지고 있다. 즉 분산된 DB 이다. 이 이유로 크립토를 감시하거나 통제하기가 힘들다. 우리가 컴퓨터를 몽땅 꺼야 비트코인이 죽을 수 있다. 따라서 불가능하다. 덕분에 크립토 커런시가 정부의 감시나 통제에 대응할 수 있다.

* 블록을 구성하는 정보 : block hash, previous block hash, block data

* block data : 비트 코인의 경우 block data 는 트랜잭션 ( 거래 내역 ) 들이다.

* hash function : 해쉬는 블록들이 서로 연결된 방법이다. 즉 이것이 블록의 체인을 만든다. 해쉬는 one - way function, deterministic 의 특성을 가진다.
1. deterministic - 같은 인풋은 항상 같은 아웃풋을 가진다.
2. one - way function - 인풋으로 아웃풋을 얻을 수는 있지만, 아웃풋으로 인풋을 얻을 수 없다.

* 내가 블록이고 비트코인 블록체인에 추가되고 싶다면? 
1. 빗코의 트랜잭션( 결제 내역 ) 을 모은다 ( 데이터 부분 )
2. 이전 블록의 해시가 필요
3. 나의 데이터 + 이전 블록의 해시를 합쳐서 다시 해시 -> 나만의 해시가 생긴다.


## REFERENCE

https://www.youtube.com/watch?v=Ca7Meu4z-F4