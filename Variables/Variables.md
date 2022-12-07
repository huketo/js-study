# 변수 할당으로 의도를 표현하라

## const로 변하지 않는 값을 표현하라

기본적으로 const를 사용하여 변수를 선언하고 할당이 변경되는 경우 드물게 let을 사용하여 변수의 변경여부에 따라 분류하여 표현하자.

```js
// problem
// const로 할당한 값은 불변값이 아니다.
function mutableDiscount(cart) {
  const discountable = [];

  for (let i = 0; i < cart.length; i++) {
    if (cart[i].discountAvailable) {
      discountable.push(cart[i]);
    }
  }
  // 하지만 이렇게 될 경우 값을 예측하기 어렵다. 
  return discountable;
}
```

```js
// solve
function discountable(cart) {
  const discountable = cart.filter(item => item.discountAvailable);

  return discountable;
}
```

## let과 const로 유효 범위 충돌을 줄여라

기본적으로 const를 사용하면서 재할당을 피하는 것이 오류를 피하고 코드의 의도를 표현하기 좋다. 하지만 반드시 재할당을 해야하는 경우에는 블록유효범위를 가지는 const와 let을 사용하자.

```js
// problem
function getLowestPrice(item) {
var count = item.inventory;
  var price = item.price;

  if (item.salePrice) {
    var count = item.saleInventory;
    if (count > 0) {
      price = item.salePrice;
    }
  }

  if (count) {
    return price;
  }

  return 0;
}
```

```js
// solve
function getLowestPrice(item) {
  const count = item.inventory;
  let price = item.price;

  if (item.salePrice) {
    const saleCount = item.saleInventory;
    if (saleCount > 0) {
      price = item.salePrice;
    }
  }

  if (count) {
    return price;
  }

  return 0;
}
```

## 블록 유효 범위 변수로 정보를 격리하라.

- 블록이란?
- 호이스팅?

```js
// problem
function addClick(items) {
  for (var i = 0; i < items.length; i++) {
    items[i].onClick = function () { return i; };
  }
  return items;
}
const example = [{}, {}];
const clickSet = addClick(example);
clickSet[0].onClick(); // --> 2
clickSet[1].onClick(); // --> 2
// 배열의 어느 항목을 선택해도 같은 값이 나옴
```

```js
// solve
// var -> let
function addClick(items) {
  for (let i = 0; i < items.length; i++) {
    items[i].onClick = function () { return i; };
  }
  return items;
}
const example = [{}, {}];
const clickSet = addClick(example);
clickSet[0].onClick(); // --> 0
clickSet[1].onClick(); // --> 1
```

## 템플릿 리터럴로 변수를 읽을 수 있는 문자열로 변환하라

`+` 쓰지말고 `${}`으로 문자열 표현하자.

```js
// problem
function getProvider() {
  // stub for get provider
  return 'pragprog.com/cloud';
}

function generateLink(image, width) {
  const widthInt = parseInt(width, 10);
  return 'https://' + getProvider() + '/' + image + '?width=' + widthInt;
}
```

```js
// solve
function getProvider() {
  // stub for get provider
  return 'pragprog.com/cloud';
}

function generateLink(image, width) {
  return `https://${getProvider()}/${image}?width=${parseInt(width, 10)}`;
}
```

> ### 간단 정리
> 1. 변수 할당할 때 `const`, `let` 써라
> 2. 문자열에 변수넣을 때 `${variable}` 템플릿 리터럴 써라