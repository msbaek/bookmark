# Bookmark API Server

이 프로젝트에서는 API Server를 acceptance test와 unit test를 혼합하여 개발한 과정을 설명한다. 그 때 작업을 하면서 내가 하고 있는 방식이 [Growing Object-Oriented Software Guided by Tests](http://www.growing-object-oriented-software.com)에서 설명하는 방법과 유사하는다 생각이 들었다.

이 책에서 7~8 페이지 이런 아래와 같은 내용이 있다.

> 어플리케이션의 어떤 클래스에 대한 단위 테스트를 작성하면서 TDD를 시작하는 것이 좋다. 이런 방식은 테스트가 없는 것보다 좋다. 또, 우리가 모두 알고는 있지만 피하기 어려운 기본적인 프로그래밍 오류를 잡을 수 있다. 
> 그러나 단위 테스트만 있는 프로젝트는 TDD 프로세스의 중요한 이점을 놓치고 있다.
> 어디에서도 호출되지 않거나 나머지 시스템과 통합될 수없고 재작성되어야만 하는, 그러나 고품질의 잘 테스트된 코드를 가진 프로젝트로 될 수 있다.
> 
>>  TDD를 통해 test coverage는 높은 고품질 코드를 만들 수 있지만, TDD가 bottom-up이기에 실제로 필요치 않고, 시스템의 다른 부분과 통합되기 어려운 기능이 나올 수 있다는 의견으로 해석된다.
> 
> 어디서부터 코드를 작성할 지 어떻게 알 수 있을까 ? 이 보다 더 중요한 것은 언제 코딩이 완료되었는지 알 수 있을까 ? 황금율(golden rule)은 우리가 해야 할 일을 알려준다: 실패하는 테스트를 작성하라.
> 
> 기능(feature)을 구현할 때 우리가 구현하려는 기능을 보여주는 acceptance test를 작성함으로써 시작한다. acceptance test가 실패하는 동안에는 acceptance test는 그 기능이 아직 구현되지 않았음을 보여준다. 테스트가 성공하면 그 기능은 완료된 것이다. 기능을 구현할 때 acceptance test는 우리가 작성하려는 코드가 필요한 것인지 알려준다. 직접적으로 연관된(테스트를 성공시키기에 꼭 필요한) 코드만 작성하게 된다.
> 
> acceptance test를 구현하는 동안 내부에서 단위 테스트 기반의 TDD(test / 구현 / 리팩토링) 싸이클로 기능을 구현한다.
> 
> 전체 과정은 Figure 1.2와 같다.
> 
> ![](https://api.monosnap.com/rpc/file/download?id=dpRHlp9LyRwgDK716wHg6boQHYZ0UE)
> 
> 외부 테스트 루프(acceptance test)는 시연 가능한 진도(demonstrable progress)를 측정하는 척도이고, 이 테스트가 증가함에 따라 시스템을 변경할 때 회귀 테스트를 통해 시스템을 안전하게 보호한다.
> acceptance test를 성공시키기 위해 꽤 시간이 걸리는 경우가 종종 발생한다. 한번의 커밋으로 완성할 수 없는 경우가 많다. 그래서 일반적으로 지금 작업 중인 acceptance test의 feature branch와 완료된 acceptance test의 feature branch를 구분하여 관리/작업한다.
> 내부 루프(unit test)는 개발자들을 지원한다. 단위 테스트는 코드의 품질을 유지하는데 도움이 되며 테스트가 작성되면 바로 성공되어야 한다(acceptance test가 성공시키는데 꽤 시간이 걸리는데 반해). 실패하는 단위 테스트는 리파지토리에 커밋되면 안된다.

## 내가 진행한 방법

acceptance test는 위에서 언급한 것처럼 회귀 테스트로 동작하여 시스템에 가한 변경이 문제를 일으켰는지 알려주는 좋은 수단이 된다.

나는 이것보다 더 큰 의미를 부여했다. Acceptance Test 클래스들에 시스템에 필요한 기능이 식별될 때 마다 대응하는 acceptance test를 추가했다. 그리고 그 테스트들의 성공 비율을 통해 프로젝트의 전체 진도를 시각화 할 수 있었다.

그리고 Acceptance Test는 무엇부터 시작할지를 잘 보여주었다. 처읍부터 unit test를 작성하려면 무엇부터 추가해야 할지 망막할 것이다.

즉 Acceptance Test는

- 회귀 테스트로 안정성 보장
- 어떤 단위 테스트부터 시작해야 할 지 안내원 역할
- 전체 공정(테스트)의 완료율(성공율)을 통해 프로젝트의 진도를 가시화

하는 효과가 있었다.

그리고 각 Acceptance Test를 성공시키기 위해서는 필요한 만큼의 단위 테스트를 만들어서 진행했다.

이 글에서 본래 프로젝트를 간소화하여 그 과정을 설명하고자 한다.

## Static Domain Modeling

먼저 요구사항을 분석하여 주요한 클래스들, 속성 그리고 이들간의 관계를 추출한다.