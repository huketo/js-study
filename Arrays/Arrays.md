# 배열로 데이터 컬렉션을 관리하라

## 배열로 유연한 컬렉션을 생성하라

자바스크립트에서 데이터 컬렉션을 다루는 구조로는 `배열(Array)`과 `객체(Object)` 두 가지가 있었습니다. 이후 모던 자바스크립트에서 `Map`, `Set`, `WeakMap`, `WeakSet` 이 추가 되었습니다. 컬렉션을 선택할 때 어떤 작업을 할지 생각해봐야 합니다. 어떤 형태로든 조작(추가, 제거, 필터링, 정렬, 교체 등)해야 한다면 배열이 가장 적합한 컬렉션입니다.

```js
// Array Methods

// Sort()
const team = [
  'Joe',
  'Dyan',
  'Bea',
  'Theo',
];

function alphabetizeTeam(team) {
  return [...team].sort();
  // ['Bea', 'Dyan', 'Joe', 'Theo']
}

// filter()
const staff = [
  {
    name: 'Wesley',
    position: 'musician',
  },
  {
    name: 'Davis',
    position: 'engineer',
  },
];

function getMusicians(staff) {
  return staff.filter(member => member.position === 'musician');
  // [{name: 'Wesley', position: 'musician'}]
}
```

```js
// 배열을 활용해 객체를 순회 - Object.keys()
// 이터러블(iterable) - 컬렉션의 현재위치를 알고 있는 상태에서 컬렉션의 항목을 하나씩 처리하는 방법
function getTotalStats() {
  const game1 = {
    player: 'Jim Jonas',
    hits: 2,
    runs: 1,
    errors: 0,
  };

  const game2 = {
    player: 'Jim Jonas',
    hits: 3,
    runs: 0,
    errors: 1,
  };

  const total = {};

  const stats = Object.keys(game1);
  // [ 'player', 'hits', 'runs', 'errors' ]
  for (let i = 0; i < stats.length; i++) {
    const stat = stats[i];
    if (stat !== 'player') {
      total[stat] = game1[stat] + game2[stat];
    }
  }

  // {
  //   hits: 5,
  //   runs: 1,
  //   errors: 1
  // }

  // player를 제외한 값들로 출력.

  return total;
}
```

## includes()로 존재 여부를 확인하라

결론만 말해서 includes()를 사용하면 배열에 있는 값의 위치를 확인하지 않고도 존재 여부를 확인할 수 있다. 보통 indexOf() 메서드를 통해서 색인을 반환 받을 수 있는 데 해당 값이 존재하지 않으면 -1이 반환되는 것을 통해 존재 유무를 판단했다. 이는 까다롭고 불편하다.

```js
// problem
// indexOf() - 배열의 index를 반환, 값이 없을 경우 -1를 반환
const sections = ['contact', 'shipping'];

function displayShipping(sections) {
  return sections.indexOf('shipping') > -1;
}

// true
```

```js
// solve
// includes() - 배열에 값이 존재하는 지 확인, true/false 반환
const sections = ['contact', 'shipping'];

function displayShipping(sections) {
  return sections.includes('shipping');
}

// true
```

## 펼침 연산자(...)로 배열을 본떠라

배열의 수많은 메서드들로 인해 높은 수준의 유연성을 자바스크립트에서 지원되고 있습니다. 하지만 배열을 직접적으로 다루는 데 있어 생기는 문제들이 있습니다. 바로 원본 배열이 변경된다는 점입니다. 때문에 배열을 복제하여 사용하게 되는 데 이를 편하게 해주는 것이 `펼침 연산자[spread] (...)` 입니다. `spread(...)`는 배열뿐만 아니라 다양한 데이터 컬렉션에서 모두 사용이 가능합니다. 

```js
// problem
// 배열에서 항목을 제거하는 함수 <- 너무 길고 복잡
function removeItem(items, removable) {
  const updated = [];
  for (let i = 0; i < items.length; i++) {
    if (items[i] !== removable) {
      updated.push(items[i]);
    }
  }
  return updated;
}

// #1 splcie()
function removeItem(items, removable) {
  if (items.includes(removable)) {
    const index = items.indexOf(removable);
    items.splice(index, 1);
  }
  return items;
}

const books = ['practical vim', 'moby dick', 'the dark tower'];
const recent = removeItem(books, 'moby dick');
const novels = removeItem(books, 'practical vim');
// 원본 배열까지 변경해버리는 문제!

// #2 slice()
// items 배열, removeable 삭제할 값
function removeItem(items, removable) {
  if (items.includes(removable)) {
    const index = items.indexOf(removable);
    // index 삭제할 값의 색인(위치)
    return items.slice(0, index).concat(items.slice(index + 1));
    // 삭제할 값을 제외하고 새로운 배열 생성
  }
  return items;
}
// concat으로 두개의 배열을 병합해서 하나의 배열을 생성
```

```js
// solve
// spread 연산자 (...)
// items 배열, removeable 삭제할 값
function removeItem(items, removable) {
  if (items.includes(removable)) {
    const index = items.indexOf(removable);
    // index 삭제할 값의 색인(위치)
    return [...items.slice(0, index), ...items.slice(index + 1)];
  }
  return items;
}
```

```js
// spread 연산자를 사용한 배열 인수전달
const book = ['Reasons and Persons', 'Derek Parfit', 19.99];

function formatBook(title, author, price) {
  return `${title} by ${author} $${price}`;
}

formatBook(book[0], book[1], book[2]);
// 'Reasons and Persons by Derek Parfit $19.99'
formatBook(...book);
// 'Reasons and Persons by Derek Parfit $19.99'

// 동일한 결과
```

## push() 메서드 대신 spread 연산자로 원본 변경을 피하라

```js
// problem
// 조건 1) 3개 이상 주문할 경우 무료 선물을 준다.
// 조건 2) 할인 상품은 하나만 주문할 수 있다.
// 장바구니 합산하는 기능
const cart = [
  {
    name: 'The Foundation Triology',
    price: 19.99,
    discount: false,
  },
  {
    name: 'Godel, Escher, Bach',
    price: 15.99,
    discount: false,
  },
  {
    name: 'Red Mars',
    price: 5.99,
    discount: true,
  },
];

const reward = {
  name: 'Guide to Science Fiction',
  discount: true,
  price: 0,
};

function addFreeGift(cart) {
  if (cart.length > 2) {
    cart.push(reward);
    return cart;
  }
  return cart;
}

function summarizeCart(cart) {
  const discountable = cart.filter(item => item.discount);
  if (discountable.length > 1) {
    return {
      error: '할인 상품은 하나만 주문할 수 있습니다.',
    };
  }
  const cartWithReward = addFreeGift(cart);
  return {
    discounts: discountable.length,
    items: cartWithReward.length,
    cart: cartWithReward,
  };
}
```

이후 코드 수정으로 변수의 위치가 변경되었다면...?

```js
function summarizeCartUpdated(cart) {
  const cartWithReward = addFreeGift(cart);
  // 리워드의 discount 값이 true이기 때문에 error 발생할 수 있음.
  const discountable = cart.filter(item => item.discount);
  if (discountable.length > 1) {
    return {
      error: '할인 상품은 하나만 주문할 수 있습니다.',
    };
  }
  return {
    discounts: discountable.length,
    items: cartWithReward.length,
    cart: cartWithReward,
  };
}
```

```js
// solve
function addGift(cart) {
  if (cart.length > 2) {
    return [...cart, reward];
  }
  return cart;
}

function summarizeCartSpread(cart) {
  const cartWithReward = addGift(cart);
  // 리워드가 포함된 장바구니 배열을 새로 만듬
  const discountable = cart.filter(item => item.discount);
  if (discountable.length > 1) {
    return {
      error: '할인 상품은 하나만 주문할 수 있습니다.',
    };
  }
  return {
    discounts: discountable.length,
    items: cartWithReward.length,
    cart: cartWithReward,
  };
}
```