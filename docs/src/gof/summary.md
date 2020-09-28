---
layout: post
title:  "GoF 디자인 패턴 정리"
date:   2018-12-09
tags: [pattern]
category: 패턴
---

GoF(Gang of Four)는 **Design Patterns: Elements of Reusable Object-Oriented Software**를 집필한 저자 4명을 말한다.
에릭 감마(Erich Gamma), 리차드 헬름(Richard Helm), 랄프 존슨(Ralph Johnson), 존 블리시데스(John Vlissides)가 이 책의 저자들이다.

Design Patterns은 세 가지 카테고리로 분류되어 23개 패턴이 있다. 대부분 예제는 C++로 작성되었고, 일부는 Smalltalk로 작성되었다. 

#### 생성 패턴(Creational Pattern)
객체의 생성과 관련있다. 생성 패턴 대부분은 정확히 무엇을 생성하는지에 대한 정보 없이도 객체를 생성할 수 있도록 해준다.

- **싱글턴** : 오직 한 개의 인스턴스 보장, 전역적인 접근 제공
  - [Angular Service](https://angular.io/guide/dependency-injection#injector-hierarchy-and-service-instances)
- **팩토리 메서드** : 객체 생성 인터페이스 정의, 서브클래스에서 클래스 생성 결정
- **추상 팩토리** : 객체 생성과 표현을 분리
- **프로토타입** : 원형 복사해서 새로운 객체 생성
- **빌더** : 복잡한 객체 생성의 동일한 절차 제공

#### 구조 패턴(Structual Pattern)
프로그램의 구성에 관련있다. 구조 패턴은 프로그램 조직화에 대한 목적을 달성한다.

- **프록시** : 다른 객체에 대한 접근 제어
  - [ES6 Proxy](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- **퍼사드** : 인터페이스를 집합에 대한 획일화된 하나의 인터페이스 제공
- **데코레이터** : 객체 동적 새로운 책임 추가
  - [TypeScript Decorator](https://www.typescriptlang.org/docs/handbook/decorators.html)
- **컴포지트** : 하나의 추상 클래스로 부분과 전체의 계층을 표현
- **브릿지** : 구현에서 추상을 분리하여, 이들이 독립적으로 다양성을 가질 수 있도록 함
- **어뎁터** : 서로 일치하지 않는 인터페이스를 사용자가 기대하는 인터페이스 형태로 적응
- **플라이웨이트** : 많은 수의 객체를 효율적으로 제공하기 위해 공유개념을 이용

#### 행동 패턴(Behavioral Pattern)
객체 간의 행위에 관련있다.객체의 역할과 객체들간의 상호 작용하는 방식을 정의한다.

- **이터레이터** : 내부 표현부를 노출하지 않고 어떤 객체 집합에 속한 원소들을 순차적으로 접근할 수 있는 방법을 제공하는 패턴
  - [ES6 Iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)
- **옵져버** : 객체 사이에 일 대 다의 의존 관계를 정의해 두어, 어떤 객체의 상태가 변할 때 그 객체의 의존성을 가진 다른 객체들이 그 변화를 통지받고 자동으로 갱신될 수 있게 만드는 패턴
  - [MutationObserver](https://developer.mozilla.org/ko/docs/Web/API/MutationObserver)
  - [IntersectionObserver](https://developer.mozilla.org/ko/docs/Web/API/IntersectionObserver/IntersectionObserver)
- **상태** : 객체의 내부 상태에 따라 스스로 행동을 변경할 수 있게 끔 허가하는 패턴
- **전략** : 동일 계열의 알고리즘군을 정의하고, 각각의 알고리즘을 캡슐화하여, 이 들을 상호교환이 가능하도록 만드는 패턴
- **책임연쇄** : 하나의 요청에 대한 처리가 반드시 한 객체에거만 되지 않고, 여러 객체에서 그 처리 기회줌
- **커멘드** : 요청 자체 캡슐화, 요청 대기/로깅하여, 되돌릴 수 있는 연산을 지원
- **탬플릿 메소드** : 알고리즘의 뼈대만을 정의하고 각 단계에서 수행할 구체적 처리는 서브클래스쪽으로 미루는 패턴
- **방문자** : 객체 구조를 이루는 원소에 대해 수행할 연산을 표현하는 패턴으로, 연산을 적용할 원소의 클래스를 변경하지 않고도 새로운 연산을 정의할 수 있게 합니다.
- **미디에이터** : 한 집합에 속해있는 객체들의 상호작용을 캡슐화하는 중앙객체를 정의하는 패턴
- **메멘토** : 캡슐화를 위배하지 않는 채 어떤 객체의 내부상태를 잡아내고 신체화시켜, 이후에 해당 객체가 그 상태로 다시 되돌아올 수 있도록 하는 패턴
- **인터프리터** : 주어진 언어에 대해, 그 언어의 문법을 위한 표현 수단을 정의하고, 이와 어울러 그 표현수단을 사용하여 해당 언어로 작성된 문장을 해석하는 해석기를 정의하는 패턴이다.
