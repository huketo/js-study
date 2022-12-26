# 조건문을 깔끔하게 작성하라.

## 거짓 값이 있는 조건문을 축약하라

자바스크립트는 형변환이 자동으로 필요한 곳에서 이루어진다. Boolean 값이 필요한 자리에 자동으로 true 와 false 값으로 바뀌게 되는 데 이때 그 값들을 `Truthy`, `Falsy` 라고 부른다.

- `Falsy`: 0, -0, 0n, "", null, undefined, NaN (8가지)
- `Truthy`: 이외의 값들...

```js
const employee = {
  name: "Eric",
  equipmentTranding: true,
}

function listCerts(employee) {
  if (employee.equipmentTranding) {
    employee.certificates = ["Equipment"];
    // 조작 !
    delete employee.equipmentTranding;
  }
}

function checkAuthorization() {
  if (employee.equipmentTranging !== true) {
    return "기계를 작동할 권한이 없습니다.";
  }
  return `반갑습니다. ${employee.name} 님`;
}

listCerts(employee);
checkAuthorization(employee);
```

## 삼항 연산자로 빠르게 데이터를 확인하라

```js
// bad
if (active) {
  var display = "bold";
} else {
  var display = "normal";
}

// good
let display = active ? "bold" : "normal";
```

삼항연산자는 그저 단순하기만 한 것이 아니라 예측가능한 장점이 있습니다. 이를 통해 변수의 재할당을 줄일 수 있습니다.

```js
const title = "과장";
if (title === "과장") {
  const permissions = ["근로시간", "수당"];
} else {
  const permissions = ["근로시간"];
}

permissions;
// Uncaught ReferenceError: permissions is not defined
```

```js
const permissions = title === "과장" ? ["근로시간", "수당"] : ["근로시간"];
```

새로운 유형의 추가..

```js
// problem
const permissions = title === "과장" || title === "부장"
  ? title === "과장"
    ? ["근로시간, 초과근무승인", "수당"]
    : ["근로시간", "초과근무승인"]
  : [근로시간];
```

```js
// solve
function getTimePermissions({ title }) {
  if (title === "과장") {
    return ["근로시간", "초과근무승인", "수당"];
  }
  if (title === "부장") {
    return ["근로시간", "초과근무승인"];
  }
  return ["근로시간"];
}

const permissions = getTimePermissions({ title: "사원" });
// ["근로시간"]
```

복잡하고 중첩된 조건문으로 가야하는 경우 블록외부로 분리하여 독립적인 함수로 사용하는 것이 바람직하다.

## 단락평가를 이용해 효율성을 극대화하라

> **단락평가(short circuiting)**: &&, ||

```js
// problem
function getIconPath(icon) {
  const path = icon.path ? icon.path : "uploads/default.png";
  return `https://assets.foo.com/${path}`;
}

const icon = {
  path: "acme/bar.png",
}

getIconPath(icon);
// 'https://assets.foo.com/acme/bar.png'
```
`icon.path`를 두 번이나 확인하고 있음.
> 단락평가(short circuiting)을 사용하자.

```js
const name = "joe" || "I have no name";
name;
"joe"
```
```js
function getIconPath(icon) {
  const path = icon.path || "uploads/default.png";
  return `https://assets.foo.com/${path}`;
}
```

### 단락평가를 통해 오류를 방지

- example) 이미지 세트를 받아 아이콘 설정.

#### 이미지세트
```js
// 지정된 배열이 없음
const userConfig1 = {
}

// 내용이 없는 배열
const userConfig2 = {
  images: []
}

// 내용이 있는 배열
const userConfig3 = {
  images: [
    "me.png",
    "work.png"
  ]
}
```

#### Case 1) 속성이 정의되지 않은 경우
```js
const img = userConfig1.image[0] || "default.png";
// Uncaught TypeError: Cannot read properties of undefined (reading '0')
```

#### Case 2) 배열에 항목이 없는 경우
```js
// problem
function getFirstImage(userConfig) {
  let img = "default.png";
  if (userConfig.images) {
    img = userConfig.images[0];
  }
  return img;
}

getFirstImage(userConfig1);
// 'default.png'
getFirstImage(userConfig2);
// undefined
// 배열에 항목이 없기 때문에 undefined로 할당됨
getFirstImage(userConfig3);
// 'me.png'
```

```js
// solve
function getFirstImage(userConfig) {
  let img = "default.png";
  if (userConfig.images) {
    if (userConfig.images.length) {
      img = userConfig.images[0];
    }
  }
  return img;
}

getFirstImage(userConfig1);
// 'default.png'
getFirstImage(userConfig2);
// 'default.png'
getFirstImage(userConfig3);
// 'me.png'
```
=> 문제는 해결하였으나 코드가 복잡해졌다.

#### short-circuiting 으로 해결
```js
// userConfig.images 가 true 이고 userConfig.iamges.length > 0 => userConfig.images[0] 
// 이외 => "default.png"
function getImage(userConfig) {
  if (userConfig.images && userConfig.images.length > 0) {
    return userConfig.images[0];
  }
  return "default.png";
}
```

#### 삼항연산자와 함께 사용하여 더 간단하게
```js
function getImage(userConfig) {
  const images = userConfig.images;
  return images && images.length ? iamges[0] : "default.png";
}
```

*주의* 삼항연산자를 사용할 때 코드의 확장성을 생각했을 때 다루기 어려워 질 수 있다.

- example) gif 확장자를 체크하고 받지 않을려고 할 경우

```js
const images = userConfig.images;
```