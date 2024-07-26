---
title: Githru-vscode-ext 파헤치기
author: BeAPro
date: 2024-07-25 16:00:00 +0900
categories: [Githru-vscode-ext]
tags: [Githru-vscode-ext, OSSCA]
image:
  path: /assets/img/title-image/githru.png
  alt: githru
---
## **Githru-vscode-ext란?**

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-01.png)

Githru-vscode-ext는 GitHub의 복잡한 commit 히스토리를 **Stem**, **Context Preserving Squash Merge**, **Commit Clustering** 기법을 사용하여 단순화하여 시각적 분석을 도와주는 VSCode 익스텐션이다.

Github 링크 : <https://github.com/githru>

### Stem
Stem은 Git 저장소의 DAG(Directed Acyclic Graph) 표현을 간소화하기 위한 개념이다.
Stem은 특정 커밋의 조상 노드 리스트로, 다중 부모 노드가 있을 때 하나의 부모만 포함합니다. 이는 Git의 첫 번째 부모 옵션(first-parent option)과 유사하다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-02.png)

위와 같이 복잡한 구조의 commit 내역을 Stem 기술을 사용하면 4가지 Stem으로 나눌 수 있다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-03.png)

기본적으로 마스터 브랜치에서 시작하여 다른 브랜치의 첫 번째 부모 커밋만 남기고 나머지 부모 노드를 제거한다. 이를 통해 각 브랜치의 단순한 경로만 남게 된다. 이를 통해 개발자는 중요한 브랜치와 커밋에 집중할 수 있으며, 전체 개발 히스토리를 개괄적으로 파악할 수 있다.



### Context Preserving Squash Merge (CSM)

CSM은 Git 저장소의 복잡한 DAG 구조를 단순화하면서도 중요한 컨텍스트를 보존하기 위해 제안된 기술이다.
여러 Stem 간의 정보를 축약하여 전체적인 그래프의 복잡성을 줄이고, 각 커밋 간의 중요한 문맥을 보존한다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-04.png)

특정 머지 커밋(CSM-base)과 관련된 모든 부모 커밋(CSM-source)을 하나의 노드로 합치면서 CSM-base에 CSM-source의 정보(작성자, 커밋 메시지, 변경된 파일 목록 등)를 포함시킨다. CSM은 주 브랜치부터 시작하여 가장 최근의 커밋 순으로 적용된다. 이를 통해 중요한 브랜치의 토폴로지를 먼저 보존할 수 있다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-05.png)

CSM을 적용시킨 결과는 위와 같다.

### Commit Clustering

Commit Clustering은 유사한 커밋들을 하나의 클러스터로 묶는 기법으로, 각 클러스터는 여러 커밋을 대표하는 하나의 노드로 표현된다.
CSM이 여러 Stem간의 정보를 축약했다면 Commit Clustering은 단일 Stem 내의 커밋들을 유사성에 따라 분류하여 복잡성을 줄인다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-06.png)

커밋의 유사성을 측정하기 위해 `Simple Additive Weighting (SAW)` 모델을 사용했다.

> Githru에는 CSM까지만 적용되었다. Commit Clustering은 [데모 사이트](https://githru.github.io/realm-java-demo/)에서 확인 할 수 있다.
{: prompt-info}


## **Githru-vscode-ext 코드 분석**

