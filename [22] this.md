# 22. this

## 핵심 내용

1. 동작을 나타내는 메서드는 자신이 속한 객체의 상태, 즉 프로퍼티를 참조하고 변경할 수 있어야 한다. 이를 도와주는 것이 this이다.
2. this 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.
    1. 메서드 내부의 this는 메서드를 호출한 객체를 가리킨다.
        1. this 바인딩은 함수 호출 시점에 결정되기 때문에 가능한 일이다.
    2. 일반 함수 내부의 this는 전역 객체를 가리킨다.
        1. 콜백함수가 일반 함수로 호출된다면 콜백 함수 내부의 this에도 전역 객체가 바인딩된다.
        2. 메서드 내에서 정의한 중첩함수도 일반 함수로 호출되면 중첩함수 내부의 this에는 전역 객체가 바인딩된다.
        3. 위와 같은 현상은 메인 함수의 로직을 일부 수행하는 보조 함수(콜백함수, 중첩함수..)의 this가 메인 함수의 this와 다르다는 의미이므로, 바람직하지 않다.
        4. 생성자 함수도 일반 함수로 호출되면 this는 전역 객체를 가리킨다.
    3. 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    4. 화살표 함수 내부의 this는 상위 스코프의 this를 가리킨다.

## 참고

- 함수의 상위 스코프를 결정하는 방식인 렉시컬 스코프는 함수 정의가 평가되어 함수 객체가 생성되는 시점에 상위 스코프를 결정하는 반면, this 바인딩은 함수 호출 시점에 결정된다.