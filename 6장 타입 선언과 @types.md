# Effective Typescript

## 6장 타입 선언과 @types

### 아이템 45. devDependencies에 typescript와 @types 추가하기

- npm은 세 가지 의존성을 구분해서 관리하며, 각각의 의존성은 package.json 파일 내의 별도 영역에 들어 있다.
- dependencies
  - 현재 프로젝트를 실행하는 데 필수적인 라이브러리들이 포함된다. 프로젝트의 런타임에 loadash가 사용된다면 dependencies에 포함되어야 한다. 프로젝트를 npm에 공개하여 다른 사용자가 해당 프로젝트를 설치한다면, dependencies에 들어 있는 라이브러리도 함께 설치될 것이다. 이러한 현상을 전이(transitive) 의존성이라고 한다.
- devDependencies
  - 현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요 없는 라이브러리들이 포함된다. 예를 들어, 프로젝트에서 사용 중인 테스트 프레임워크가 devDependencies에 포함될 수 있는 라이브러리이다. 프로젝트 npm에 공개하여 다른 사용자가 해당 프로젝트를 설치한다면, devDependencies에 포함된 라이브러리들은 제외된다는 것이 dependencies와 다른 점이다.
- peerDependencies
  - 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함된다. 단적인 예로 플러그인을 들 수 있다. 제이쿼리의 플러그인은 다양한 버전의 제이쿼리와 호환되므로 제이쿼리의 버전을 플러그인에서 직접 선택하지 않고, 플러그인이 사용되는 실제 프로젝트에서 선택하도록 만들 때 사용한다.
- 타입스크립트는 devDependencies에 넣는 것이 좋다.
  - 팀원들이 항상 정확한 버전의 타입스크립트를 설치할 수 있다.
- 사용하려는 라이브러리에 타입 선언이 포함되어 있지 않더라도 DefinitelyTyped에서 타입 정보를 얻을 수 있다.

#### 요약

- 타입스크립트를 시스템 레벨로 설치하면 안 된다. 타입스크립트를 프로젝트 devDependencies에 포함시키고 팀원 모두가 동일한 버전을 사용하도록 해야 한다.
- @types 의존성 dependencies가 아니라 devDependencies에 포함시켜야 한다. 런타임에 @types가 필요한 경우라면 별도의 작업이 필요할 수 있다.

### 아이템 46. 타입 선언과 관련된 세 가지 버전 이해하기

- 타입스크립트는 의존성 관리를 오히려 복잡하게 만든다.
  - 타입스크립트를 사용하면 라이브러리, 타입 선언(@types)의 버전, 타입스크립트 버전을 추가로 고려해야 하기 때문이다.
- 타입스크립트에서 일반적으로 특정 라이브러리를 dependencies로 설치하고, 타입 정보를 devDependencies로 설치한다.
- 실제 라이브러리와 타입 정보의 버전이 별도로 관리되는 방식은 다음 네 가지 문제점이 있다.
  - 라이브러리를 업데이트했지만 실수로 타입 선언은 업데이트하지 않는 경우
  - 라이브러리보다 타입 선언의 버전이 최신인 경우
  - 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립트 버전이 최신인 경우
  - @types의 의존성이 중복될 경우
- 자체적인 타입 선언은 보통 package.json의 types 필드에서 .d.ts 파일을 가리키도록 되어 있다.
- 번들링 방식의 문제점 네 가지
  - 번들링 타입 선언에 보강 기법으로 해결할 수 없는 오류가 있는 경우, 또는 공개 시점에는 잘 동작했지만 타입스크립트 버전이 올라가면서 오류가 발생하는 경우
  - 프로젝트 내의 타입 선언이 다른 라이브러리의 타입 선언에 의존한 경우
  - 프로젝트의 과거 버전에 있는 타입 선언에 문제가 있는 경우
  - 타입 선언의 패치 업데이트를 자주 하기 어려움
- 라이브러리를 공개하려는 경우, 타입 선언을 자체적으로 포함하는 것과 타입 정보만 분리하여 DefinitelyTyped에 공개하는 것의 장단점을 비교해 봐야 한다.
  - 공식적인 권장사항은 라이브러리가 타입스크립트로 작성된 경우만 타입 선언을 라이브러리에 포함하는 것이다.

#### 요약

- @types 의존성과 관련된 세 가지 버전이 있다. 라이브러리 버전, @types 버전, 타입스크립트 버전이다.
- 라이브러리를 업데이트하는 경우, 해당 @types 역시 업데이트해야 한다.
- 타입 선언을 라이브러리에 포함하는 것과 DefinitelyTyped에 공개하는 것 사이의 장단점을 이해해야 한다. 타입스크립트로 작성된 라이브러리라면 타입 선언을 자체적으로 포함하고, 자바스크립트로 작성된 라이브러리라면 타입 선언을 DefinitelyTyped에 공개하는 것이 좋다.

### 아이템 47. 공개 API에 등장하는 모든 타입을 익스포트하기

- 라이브러리 제작자는 프로젝트 초기에 타입 익스포트부터 작성해야 한다.
- 함수의 선언에 이미 타입 정보가 있다면 제대로 익스포트되고 있는 것이며, 타입 정보가 없다면 타입을 명시적으로 작성해야 한다.

#### 요약

- 공개 메서드에 등장한 어떤 형태의 타입이든 익스포드한다. 어차피 라이브러리 사용자가 추출할 수 있으므로, 익스포트하기 쉽게 만드는 것이 좋다.

### 아이템 48. API 주석에 TSDoc 사용하기

```typescript
/**
 * 인사말을 생성합니다.
 * @param name 인사할 사람의 이름
 * @param title 그 사람의 칭호
 * @returns 사람이 보기 좋은 형태의 인사말
 */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

- 위의 예시처럼 JSDoc 스타일로 주석을 작성하면, 대부분의 편집기가 주석을 툴팁으로 표시해 준다.
- 만약 공개 API에 주석을 붙인다면 JSDoc 형태로 작성해야 한다.
- TSDoc 주석은 마크다운 형식으로 꾸며지므로 굵은 글씨, 기울임 글씨, 글머리 기호 목록을 사용할 수 있다.

#### 요약

- 익스포트된 함수, 클래스, 타입에 주석을 달 때는 JSDoc/TSDoc 형태를 사용한다. JSDoc/TSDoc 형태의 주석을 달면 편집기가 주석 정보를 표시해 준다.
- @param, @returns 구문과 문서 서식을 위해 마크다운을 사용할 수 있다.
- 주석에 타입 정보를 포함하면 안 된다.

### 아이템 49. 콜백에서 this에 대한 타입 제공하기

- let이나 const로 선언된 변수는 렉시컬 스코프인 반면, this는 다이나믹 스코프이다.
  - 다이나믹 스코프는 정의된 방식이 아니라 호출된 방식에 따라 달라진다.

```typescript
class C {
  vals = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val * val);
    }
  }
}

const c = new C();
const method = c.logSquares;
method(); // error
```

- 위의 예시에서 this의 값은 undefined로 설정된다.
- call을 사용하면 명시적으로 this를 바인딩하여 문제를 해결할 수 있다.

```typescript
class ResetButton {
  render() {
    return makeButton({ text: 'Reset', onClick: this.onClick });
  }

  onClick() {
    alert(`Reset ${this}`);
  }
}
```

- 위의 코드는 this 바인딩 문제로 동작하지 않는다.

```typescript
// solve 1
class ResetButton {
  constructor() {
    this.onClick = this.onClick.bind(this);
  }

  render() {
    return makeButton({ text: 'Reset', onClick: this.onClick });
  }

  onClick() {
    alert(`Reset ${this}`);
  }
}

// solve 2
class ResetButton {
  render() {
    return makeButton({ text: 'Reset', onClick: this.onClick });
  }

  onClick = () => {
    alert(`Reset ${this}`);
  };
}
```

#### 요약

- this 바인딩이 동작하는 원리를 이해해야 한다.
- 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시해야 한다.

### 아이템 50. 오버로딩 타입보다는 조건부 타입을 사용하기

```typescript
function double(x: number | string): number | string;
function double(x: any) {
  return x + x;
}
```

- 위의 예시는 string을 넣으면 string이 나오고, number를 넣으면 number를 반환해야 한다.

```typescript
function double<T extends number | string>(x: T): T;
function double(x: any) {
  return x + x;
}
```

- 위의 예시는 타입을 구체적으로 만들어 주지만 과한면이 있다.

```typescript
function double(x: string): string;
function double(x: number): number;
function double(x: any) {
  return x + x;
}
```

- 위의 예시는 타입이 명확해졌지만, 유니온 타입 관련해서 문제가 발생한다.
- 조건부 타입은 타입 공간의 if 구문과 가탇.

```typescript
function double<T extends number | string>(
  x: T
): T extends string ? string : number;
function double(x: any) {
  return x + x;
}
```

- 오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 저 정확해진다.

#### 요약

- 오버로딩 타입보다 조건부 타입을 사용하는 것이 좋다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원할 수 있다.

### 아이템 51. 의존성 분리를 위해 미러 타입 사용하기

- 작성 중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것(미러링)을 고려해 보는 것이 좋다.
- 다른 라이브러리의 타입이 아닌 구현에 의존하는 경우에도 동일한 기법을 적용할 수 있고 타입 의존성을 피할 수 있다.

#### 요약

- 필수가 아닌 의존성을 분리할 때는 구조적 타이핑을 사용하면 된다.
- 공개한 라이브러리를 사용하는 자바스크립트 사용자가 @types 의존성을 가지지 않게 해야 한다.

### 아이템 52. 테스팅 타입의 함정에 주의하기

- 타입 선언 파일을 테스팅할 때는 이 테스트 코드처럼 단순히 함수를 실행만 하는 방식을 일반적으로 적용한다.
  - 라이브러리 구현체의 기존 테스트 코드를 복사하면 간단히 만들 수 있기 때문이다.
- 테스팅을 위한 할당을 사용하는 방법에는 두 가지 문제점이 있다.
  - 불필요한 변수를 만들어야 한다.
    - 헬퍼 함수를 정의한다.
  - 타입이 동일한지 체크하는 것이 아니라 할당 가능성을 체크하고 있다.

#### 요약

- 타입을 테스트할 때는 특히 함수 타입의 동일성(equality)과 할당 가능성(assignability)의 차이점을 알고 있어야 한다.
- 콜백이 있는 함수를 테스트할 떄, 콜백 매개변수의 추론된 타입을 체크해야 한다. 또한 this가 API의 일부분이라면 역시 테스트해야 한다.
- 타입 관련된 테스트에서 any를 주의해야 한다. 더 엄격한 테스트를 위해 dtslint 같은 도구를 사용하는 것이 좋다.
