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
  - 타입을 확장하거나 새로운 타입을 만들 때, 특정 필드만 가져올 때, 타입을 options로 바꿀 때 사용가능

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
