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
