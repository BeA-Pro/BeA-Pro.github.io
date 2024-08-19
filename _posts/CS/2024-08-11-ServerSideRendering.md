---
title: 서버 사이드 렌더링 vs 클라이언트 사이드 렌더링
author: BeAPro
date: 2024-08-11 15:00:00 +0900
categories: [Study,Computer Science]
tags: [Rendering,Web,Computer Science]
image:
  path: /assets/img/title-image/computerScience.png
  alt: computerScience
---
## Client-Side Rendering (CSR)

### CSR이란?

CSR(클라이언트사이드 렌더링)은 브라우저가 빈 페이지를 로드한 후, JavaScript를 사용하여 그 페이지를 콘텐츠로 채우는 방식이다. HTML, JS, CSS 등을 서버로부터 받아서 클라이언트측에서 DOM 요소를 조작하여 내용을 출력한다. 이 경우 브라우저가 사용자 인터페이스를 생성하고 표시하는 데 더 적극적인 역할을 한다.

### CSR의 장점

- 향상된 인터랙티비티: CSR 애플리케이션은 페이지 전체를 다시 로드하지 않고도 사용자 인터페이스를 업데이트할 수 있어 더 인터랙티브한 경험을 제공한다.
- 적은 서버 부하: 부분적으로 데이터를 불러와서 UI를 업데이트 하기때문에, 상대적으로 서버의 부하가 적다.
- 낮은 초기 로드 시간: 애플리케이션의 골격만 브라우저로 전송되기 때문에 초기 페이지 로드가 더 빠를 수 있다.


## Server-Side Rendering (SSR)

### SSR이란?
SSR(서버사이드 렌더링)은 웹 페이지가 서버에서 생성된 후 브라우저로 전송되는 방식이다. 즉, 서버가 페이지의 논리와 구조를 처리하여 완전히 렌더링된 페이지를 사용자의 브라우저로 보낸다.

### SSR의 장점
- 향상된 SEO: 서버에서 렌더링된 페이지는 초기 HTML에 콘텐츠가 포함되어 있어 검색 엔진에 더 친화적이다.

CSR(클라이언트사이드 렌더링)에서는 브라우저가 초기 HTML을 받아오지만, 이 HTML은 거의 빈 상태이며, 자바스크립트를 통해 콘텐츠가 동적으로 로드된다. 하지만 SSR에서는 서버가 완전히 렌더링된 HTML 페이지를 생성하여 클라이언트(브라우저)에 전송한다. 이 HTML 페이지에는 모든 콘텐츠가 포함되어 있어, 검색 엔진 크롤러가 페이지를 방문할 때 이미 렌더링된 콘텐츠를 바로 읽고 인덱싱할 수 있다.

- 향상된 초기 성능: 사용자는 처음부터 렌더링된 페이지를 받기 때문에 콘텐츠를 더 빨리 볼 수 있다.


## 요약

SSR과 CSR은 웹 페이지를 렌더링하는 서로 다른 접근 방식이다. SSR은 SEO와 초기 성능을 개선하는 데 유리하며, CSR은 더 인터랙티브한 경험을 제공한다. 이들 중 어느 것을 선택할지는 애플리케이션의 목표와 특정 요구 사항에 따라 달라진다.

Next.js를 사용하여 잘 변경되지 않는 페이지는 SSG로, profile과 같이 변경되는 경우가 많은 페이지는 CSR/SSR을 하이브리드해서 만들 수 있다.

개발을 할 때 페이지의 목적에 따라 렌더링 방식을 정하면 더 높은 사용자 경험을 제공할 수 있다.