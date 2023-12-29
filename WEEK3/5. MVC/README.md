
# 5. MVC

## MVC란?

- MVC는 소프트웨어 디자인 패턴 중 하나
- MVC는 소프트웨어를 **Model, View, Controller**로 나누고 각각 **역할** 부여
- **모든 문제를 해결할 수 있을 만큼 상세하게 정의**하고 있지 않음

### Model

- 소프트웨어에서의 정보 및 데이터, 이에 대한 조작
- Controller에게 받은 데이터를 조작하는 역할

### View

- 사용자가 보게되는 인터페이스
- Controller에게 받은 Model의 데이터를 사용자에게 시각적으로 보여주기 위한 역할

### Controller

- Model과 View를 잇는 중간 매개체
- Controller는 Model과 View 사이에서 데이터 흐름을 제어하는 역할

## MVC의 흐름

![Untitled](https://github.com/sj7699/Backend-Study/assets/26706925/7f839dc3-8de1-40b5-bb1d-43b39b7fd1ee)


1. **User**: 사용자가 웹 사이트에 접속한다.
2. **Manipulates**: Controller는 사용자가 요청한 웹 페이지를 보여주기 위해 Model을 호출한다.
3. **Updates**: Model은 비즈니스 로직을 통해 DB 및 파일과 같은 데이터를 제어한 후 결과를 반환한다. 이후 Controller는 Model에게 반환받은 결과를 View에 반영한다.
4. **Sees**: 데이터를 받아온 View가 사용자에게 웹 페이지를 출력하여 보여준다.

## MVC가 만들어진 이유

### 등장 배경

- GUI 시스템을 개발하는 과정에서 **특정 도메인의 문제를 해결하기 위해 만들어진 설계 사례**
- 즉, **관심사의 분리를 통한 상호 독립성 보장**
- 자세히는 **도메인(모델) 과 UI(뷰)의 분리**를 위한 해결책

### MVC로 얻을 수 있는 것들

1. **관심사의 분리**
2. **코드의 재사용성 및 확장성 증가**
3. **유지보수 용이**
4. **테스트 용이**
5. **개발자간의 협업 용이**

## MVC의 역할과 규칙

1. 모델은 컨트롤러나 뷰에 의존하면 안된다.
    - 모델 내부에 컨트롤러 및 뷰와 관련된 코드가 있으면 안된다.
2. 뷰는 모델에만 의존해야 하고, 컨트롤러에는 의존하면 안된다.
    - 뷰 내부에 모델의 코드만 있을 수 있고, 컨트롤러의 코드가 있으면 안된다.
3. 뷰가 모델로부터 데이터를 받을 때는 사용자마다 다르게 보여주어야 하는 데이터에 한해서만 받아야 한다.
4. 컨트롤러는 모델과 뷰에 의존해도 된다.
    - 컨트롤러 내부에는 모델과 뷰의 코드가 있을 수 있다.
5. 뷰가 모델로부터 데이터를 받을 때는 반드시 컨트롤러에서 받아야 한다.

## MVC는 왜 프론트엔드에서 잘 안쓰일까?

### 너무 큰 컨트롤러(Massive-View-Controller)
![Untitled 2](https://github.com/sj7699/Backend-Study/assets/26706925/b4b7a04a-3157-4de3-ad13-acb4a72b51d7)

![Untitled 1](https://github.com/sj7699/Backend-Study/assets/26706925/7f22e279-3268-4559-87a7-b005d2e74483)


- 원인은 여러 개의 View와 여러 개의 Model로 인한 **복잡한 상호작용**
- 사용자 인터페이스의 복잡성 + View에서 보여줘야하는 동적인 요소 증가
- Controller 코드의 비대함으로 인한 재사용성, 확장성, 테스트 용이성, 유지보수 용이성 하락
- Controller는 Model과 View의 변경에 취약해짐
wdW

### 프론트의 주요 문제

- **복잡한 View - Model 관계 단순화**
- **계층적인 View**를 통한 **효율적인 DOM 처리**

### 프론트엔드의 솔루션

- MVVM
    - Model View ViewModel
    - 컨트롤러의 반복적인 기능이 **선언적인 방식**으로 개선
        - **템플릿**과 **바인딩**을 이용하여 선언적인 방식으로 변경
    - • **Model과 View의 관점을 분리하려 하지 않고 하나의 템플릿으로 관리하려는 방식**으로 발전
- Data Binding
    - View가 변경되거나 Model이 변경되었을 때 **다른 쪽도 자동 변경**
        - Observer 패턴,proxyAPI,defineProperty를 통해 구현

**[출처]** [M-V-Whatever 정리 - 1.MVC](https://blog.naver.com/jukrang/221414570067)

**[출처]** [https://velog.io/@langoustine/여기도-MVC-저기도-MVC-MVC-패턴이-뭐야](https://velog.io/@langoustine/%EC%97%AC%EA%B8%B0%EB%8F%84-MVC-%EC%A0%80%EA%B8%B0%EB%8F%84-MVC-MVC-%ED%8C%A8%ED%84%B4%EC%9D%B4-%EB%AD%90%EC%95%BC)
