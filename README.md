# Effective TypeScript Study

## item14: 타입 연산과 제네릭 사용으로 반복 줄이기

### 1. 명명된 타입 시그니처 사용해 반복 코드 줄이기

- interface 혹은 type을 생성해서 타입 정의

- 함수가 같은 타입 시그니처를 공유할 경우 타입 시그니처를 명명하고 재사용

- 인터페이스 확장 interface extends, type &

- 공통 필드만 분리해내 클래스로 추출

- 인덱싱하여 속성의 타입에서 중복 제거

- 매핑된 타입

  - 배열의 필드를 루프 도는 것과 같은 방식
  - 타입을 확장하거나 새로운 타입을 만들 때, 특정 필드만 가져올 때, 타입을 options로 바꿀 때 사용가능hg

    - 타입 확장

      ```ts
          type Person = {
              name: string;
              email: string;
              age: number;
              gender: string;
              birthDay: Date
          };


          type VIP ={
              [K in keyof Person]: Person[K],
              isVip: boolean;
          }
      ```

    - 새로운 타입 생성

      ```ts
      type NewPerson = {
        [K in keyof Person]: boolean;
      };
      ```

    - 특정 필드만 가져올 때
    - Pick 메서드로도 구현가능
    - 제네릭 두 개의 매개변수를 받아 호출

      ```ts
      type VipState = {
        [K in "isVip" | "name"]: Person[K];
      };

      type Pick<T, K extends keyof T> = {
        [P in K]: T[P];
      };

      type VipState = Pick<Person, "isVip" | "name">;

      // Pick 활용 예시
      export interface BrandSearchValue {
        rank: number;
        arrow: string;
        arrow_value: number;
        is_new: boolean;
        brand_nm: string;
        srch_cnt: number;
        comp_arrow: string;
        comp_rto: string;
      }

      export interface BrandWeightGrid {
        grid_headers: Headers[];
        results: BrandWeightValue;
      }

      type BrandWeightValue = Pick<
        BrandSearchValue,
        "rank" | "arrow" | "arrow_value" | "is_new" | "brand_nm"
      > & {
        srch_weht: string;
      };
      ```

    - 타입을 선택으로 바꿀 때
    - Partial 타입은 T의 모든 필드를 선택적으로 만듦

      ```ts
      // 타입을 options로 바꿀 때
      type Partial<T> = {
        [P in keyof T]?: T[P];
      };

      type OptionalPerson = {
        [K in keyof Person]?: Person[K];
      };
      type PartialPerson = Partial<Person>;
      ```

- 값의 형태에 해당하는 타입을 정의

  ```ts
  const person = {
      name: 'oppenheimer',
      email: 'bomb@trinity.com
      age: 32,
  }
  type Person = typeof person;
  ```

- 함수나 메서드의 반환값에 해당하는 타입을 정의

  ```ts
  function get(key: string) {
    return localStorage.getItem(key);
  }

  type Get = ReturnType<typeof get>;
  ```

- 제네릭 타입에서 매개변수를 제한하는 방법

  - extends를 활용
  - extends는 확장이라기 보다는 부분집합

  ```ts
  interface Person {
    name: string;
    age: number;
    location: string;
  }

  type Persons<T extends Person> = [T, T];

  const persons: Persons<Person> = [
    {
      name: "oppenheimer",
      age: 32,
      location: "los alamos",
    },
    {
      name: "einstein",
      age: 42,
      location: "princeton",
    },
  ];

  // error
  const wrongPersons: Persons<{ name: string; age: number }> = [
    {
      name: "straus",
      age: 35,
    },
    {
      name: "robert",
      age: 64,
    },
  ];
  ```

## Item15: 동적 데이터에 인덱스 시그니처 사용하기

- 타입스크립트는 '인덱스 시그니처를' 사용해 동적 데이터를 표현할 수 있음
- `[key: string]: string`과 같은 형태로 사용, key는 참고정보로 아무값이나 써도 됨
- 다만 인덱스로 string 사용이 너무 광범위 하다고 느껴지면 Record와 매핑된 타입을 사용할 수 있음

  ```ts
  interface Person {
    [key: string]: string;
    name: string;
    email: string;
    gender: string;
  }
  ```

  1.  Record

      - 키 타입에 유연성을 제공하는 제네릭 타입
      - `Record<K, T>`는 K 타입의 키를 가지고 T 타입의 값을 가지는 객체를 생성

  2.  매핑된 타입

      - 조건부로 key 타입을 제한할 수 있음, extends는 확장보다는 제한의 의미를 가짐

      ```ts
          Type Person = {[k in 'name' | 'email' | 'age']: k extends 'age' ? number : string} = {
          name: string
          email: string
          age: number
          }
      ```

## Item16: number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 자바스크립트의 객체는 문자열이나 심볼을 키로 사용할 수 있음, number는 문자열로 변환되어 사용됨
- 배열의 인덱스는 숫자로 접근 가능함, 인덱스는 문자열로 변환됨
- 타입스크립트는 Array에 대해서 선언할 때 number 인덱스 시그니처를 사용함
- 런타임에서는 의미 없는 가상의 코드이지만, 타입 체크 시점에서 오류를 잡을 수 있어 유용함
- ? 인덱스 시그니처에 number를 사용하는 것은 피하고, Array, 튜플, ArrayLike를 사용하는 것이 좋음

## Item17: 변경 관련된 오류 방지를 위해 readonly 사용하기

- 값이 변경되면 안되는 경우 readonly를 사용해 오류를 방지할 수 있음
- readonly number[]와 number[]의 차이

  - 배열 자체 read, write 불가능
  - length를 읽을 순 있지만, 변경은 불가능
  - 배열 변경 메소드 사용 불가능(push, pop, splice 등)
  - readonly 배열은 보통 배열의 할당 가능 반대는 불가능

- 함수의 매개변수가 함수 내에서 변경되지 않는다면 명시적으로 readonly를 사용하는 것이 좋음

  - 의도치 않은 변경 가능
  - 더 넓은 타입을 사용할 수 있음(item 29)

    ```ts
    function sum(numbers: readonly number[]) {
      return numbers.reduce((total, n) => total + n, 0);
    }
    ```

  - readonly는 얕게(shallow) 동작함
  - readonly를 깊게(deeply) 사용하려면 Readonly 제네릭을 사용해야 함

    ```ts
    type DeepReadonly<T> = {
      readonly [P in keyof T]: DeepReadonly<T[P]>;
    };

    const readonlyPerson: DeepReadonly<Person> = {
      name: "oppenheimer",
      email: "bomb1945@trinity.com":
      age: 32,
    };
    ```

## Item18: 매핑된 타입을 사용하여 값을 동기화하기

- 리액트가 state를 동기화 및 최적화 하듯
- 매핑된 타입으로 변경사항 발생시 동기화 하는 최적화 가능

# 3장 타입 추론

## Item19: 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입스크립트는 단순한 문자열이나 숫자 부터 객체까지도 타입을 추론함
- 따라서 불필요한 타입 구문을 작성하지 않도록 해야함
- 타입스크립트의 변수는 일반적으로 처음 등장할 때 타입이 결정됨

> 타입 할당하지 않아도 되는 경우

- 함수의 파라미터에 대한 타입 정의 => 비구조화 할당문으로 변경

  - 함수/메서드 시그니처에 타입O but, 함수 내 지역변수 타입 구문X가 이상적

  ```ts
  interface User {
    id: string;
    name: string;
    email: string;
    age: number;
    gender: string;
  }

  function logUser(user: User) {
    const id: string = user.id;
    const name: string = user.name;
    const email: string = user.email;
    const age: number = user.age;
    const gender: string = user.gender;
    console.log(id, name, email, age, gender);
  }

  // 비구조화 할당문
  function logUser(user: User) {
    const { id, name, email, age, gender } = user;
    console.log(id, name, email, age, gender);
  }
  ```

- 함수의 매개변수의 기본값이 있는 경우

> 타입이 추론됨에도 명시적으로 적는 경우

- 객체 리터럴 -> `잉여 속성 체크` 동작

  - 선택적 속성이 있는 타입의 오타 체크 도와줌
  - 변수가 사용되는 시점이 아닌 할당하는 시점에 오류가 표시

- 함수의 반환 타입
  - 정확한 위치 오류 표시가능
  - 입력과 출력을 명확히 할 수 있음
  - 명명된 타입으로 직관적으로 표현 가능

## Item20: 다른 타입에는 다른 변수 사용하기

- 변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다!

  ```ts
  let x = 10;
  x = "hello"; // error
  ```

> 타입을 바꾸는 방법

    - 범위를 좁히기 -> 유니온 타입 사용

## Item21: 타입 넓히기

- 런타임에 모든 변수는 유일한 값을 가지지만 코드 제크하는 정적 분석 시점에 변수는 `가능한` 값들의 집합 타입을 가짐
- 타입체커는 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야함!
- 이 과정을 `타입 넓히기`라고 함

- 타입 체커가 타입을 정하기는 상당해 모호함
- 넓히기 과정을 제어할 수 있는 방법

  - 변수 선언시 const를 사용 -> const는 재할당이 불가능함으로 더 좋은 범위로 타입을 한정할 수 있음

    ```ts
    const x = "x";
    x; // type: 'x' not string
    ```

  - 그러나 const는 객체나 배열일 때 문제가됨
  - 객체의 경우 타입스크립트 넓히기 알고리즘은 변수를 let으로 선언한 것과 같이 동작함

> 타입 추론의 강도 직접제어

1. 명시적 타입 구문 제공
2. 타입 체커에 추가적인 문맥 제공 (-> 함수의 매개변수로 값을 전달)
3. const 단언문을 사용(-> as const)

   - 타입스크립트는 최대한 좁은 타입으로 타입 추론
     ```ts
     const x = {
       a: "a",
       b: "b",
     } as const;
     x; // type: {readonly a: string, readonly b: string}
     ```

## Item22: 타입 좁히기

- 대표적으로 조건문을 활용한 null 체크

1. instanceof

2. 속성 체크(in 사용)

3. 내장 함수 사용(typeof, Array.isArray, Number.isNaN 등)

4. 명시적 태그 붙이기(태그된 유니온, 구별된 유니온 패턴)

5. 커스텀 함수(사용자 정의 타입 가드)

- 타입을 좁힐 때는 실수를 조심해야함
  - typeof null은 'object'임

## Item23: 한꺼번에 객체 생성하기

- 일반적으로 타입은 변경되지 않음

- 객체 생성시 한꺼번에 생성해야 타입추론에 유리
  - 객체 전개 연산자를 통해 큰 객체를 한거번에 생성 가능
  - 안전하게 객체 추가하려면 전개 연산자를 사용하는 것이 좋음

## Item24: 일관성 있는 별칭 사용하기

- 제어 흐름을 방해하지 않도록 타입 별칭을 일관성 있게 사용해야함
- 할 수 있다면 별칭 보다는 비구조화 할당을 사용하는 것이 가독성이 좋음

- - 타입스크립트는 함수가 타입정제를 무효화하지 않는다고 가정함 그러나 실제로 무효화될 수 있음

## Item25: 비동기 코드에는 콜백 대신 async 함수 사용하기

- 콜백보다는 프로미스나 async/await를 사용하자

  - 타입 추론 용이
  - 코드 작성 용이

- 콜백 API 혹은 함수를 래핑하는 경우 프로미스보다 async/await를 사용하자
  - async 함수는 항상 프로미스를 반환하도록 강제됨 -> 함수가 프로미스를 반환한다면 async 사용!
  - 콜백이나 프로미스 사용의 경우 반동기 코드를 작성할 수 있음
  - async 함수는 항상 비동기 코드로 작성됨
  - async 함수 안에서 프로미스 반환시 또 다른 프로미스로 래핑되지 않음 -> 타입 정보 명확히 드러남

## Item26: 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입스크립트는 할당 시점에 타입을 추론

  - 발생하는 문제

    ```ts
    type Country = "korea" | "china" | "japan";
    function setLocation(country: Country) {
      console.log(country);
    }

    setLocation("korea");

    let country = "korea"; // 할당 시점에 타입추론 -> ts는 country를 string으로 추론
    setLocation(country); // 'string' 형식의 인수는 'Country' 형식의 매개변수에 할당될 수 없습니다.
    ```

  - 해결방법

  1.  타입 선언에서 country의 가능한 값 제거
      ```ts
      let country: Country = "korea";
      setLocation(country);
      ```
  2.  함수에 파라미터로 넣을 인수를 const 상수로 선언

           ```ts
           const country = "korea"; // 상수로 변경할 수 없는 값으로 선언 -> 타입스크립트는 더 구체적인 타입으로 추론
           setLocation(country);
           ```

      > 튜플 사용시 문맥으로부타 값을 분리했을 때 주의점

      ```ts
      function SearchWords(words: [string, number]) {
        const [word, times] = words;
        return { word, times };
      }

      SearchWords(["korea", 10]); // 정상

      <!-- 문맥으로부터 값 분리시 에러 피하기 -->
      <!-- 첫번째 방법 -->
      <!-- 타입 선언 제공 -->

      const words: [string, number] = ["korea", 10];
      SearchWords(words);

      <!-- 두번째 방법 -->
      <!-- as const 타입 단언 사용 -->

      const words = ["korea", 10] as const; // 상수 문맥 제공
      SearchWords(words);

      // const는 얕은 상수
      // as const는 그 내부까지 상수라는 사실을 타스에게 알려줌
      // 다만, as const를 사용할 경우 readonly가 되기 때문에 주의해야함
      // -> 정의한 곳이 아니라 사용한 곳에서 오류가 발생함

      ```

## Item27: 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 내장된 함수형 기법과 로대시 같은 유툴리티 라이브러리를 쓰자
  - 가독성 높임
  - 타입 흐름 개선
  - 명시적인 타입 구문의 필요성 줄어듬
