# 특수한 컬렉션을 이용해 코드 명료성을 극대화하라

## 객체를 이용해 정적인 키-값을 탐색하라

- 값을 표현하기 위해서 객체를 사용하면 좋다.

```js
const colorArray = ["#d10202", "#19d836", "#0e33d8"];

console.log(colorArray[0]);
// #d10202

const colorObject = {
  red: "#d10202",
  green: "#19d836",
  blue: "#0e33d8",
};

console.log(colorObject.red);
// #d10202
```

- 설정파일 같은 Key-Value 정보를 저장하하고 정적인 정보에 적합하다. 계속해서 갱인, 반복, 대체, 정렬해야할 정보에는 객체는 적합하지 않습니다.

```js
export const config = {
  endpoint: "http://pragprog.com",
  key: "secretkey",
};
```

- 함수의 인자로 객체를 전달 받을 떄 객체의 구조를 미리 알고 있다면 변수를 이용해 키를 설정하지 않습니다. 또한 인자로 객체를 받았을 때 각각의 함수에서 새로운 객체를 생성하여 사용합니다.

```js
function getBill(item) {
  return {
    name: item.name,
    due: twoWeeksFromNow(),
    total: calculateTotal(item.price),
  };
}

const bill = getBill({ name: "객실 청소", price: 30 });

function displayBill(bill) {
  return `${bill.name} 비용은 ${bill.total} 달러이며 납부일은 ${bill.due}입니다.`;
}
```

## Object.assign()으로 조작없이 객체를 생성하라

객체에 새로운 필드를 추가하거나 외부 API에서 데이터를 가져와 현재 데이터 모델에 연결해야 하는 경우 - 기본값을 설정하면서 원래의 데이터를 유지하는 새로운 객체를 생성해야 할땐 어떻게 할까?

```js
const defaults = {
  author: "",
  title: "",
  year: 2017,
  rating: null,
};

const book = {
  author: "Joe Morgan",
  title: "Simplifying JavaScript",
};

// 객체의 키값을 index로 배열을 사용하여 해결, 너무 장황
function addBookDefaults(book, defaults) {
  const fields = Object.keys(defaults);
  const updated = {};
  for (let i = 0; i < fields.length; i++) {
    const field = fields[i];
    updated[field] = book[field] || defaults[field];
  }
  return updated;
}
```

Object.assign()을 사용하여 다른 객체의 키값으로 객체의 필드를 생성하고 갱신할 수 있다.

```js
// problem
const book = {
  author: "Joe Morgan",
  title: "Simplifying JavaScript",
};

const defaults = {
  author: "",
  title: "",
  year: 2017,
  rating: null,
};

Object.assign(defaults, book);

// {
//   author: 'Joe Morgan',
//   title: 'Simplifying JavaScript',
//   year: 2017,
//   rating: null,
// }

const anotherBook = {
  title: "Another book",
  year: 2016,
};

Object.assign(defaults, anotherBook);

// {
//   author: 'Joe Morgan',
//   title: 'Another book',
//   year: 2016,
//   rating: null,
// }

console.log(defaults);
// {
//   author: 'Joe Morgan',
//   title: 'Another book',
//   year: 2016,
//   rating: null
// }

// ! 원본 객체도 변경되어 버리는 문제
```

```js
// solve
const defaults = {
  author: "",
  title: "",
  year: 2017,
  rating: null,
};

const book = {
  author: "Joe Morgan",
  title: "Simplifying JavaScript",
};

// 빈 객체에 새로운 값이 갱신되어 반환
const updated = Object.assign({}, defaults, book);
// {
//   author: 'Joe Morgan',
//   title: 'Simplifying JavaScript',
//   year: 2017,
//   rating: null
// }
console.log(defaults);
// {
//   author: '',
//   title: '',
//   year: 2017,
//   rating: null,
// };

// 원본 조작이 발생하지 않음
```

중첩된 객체를 복사할 때 문제

```js
const defaultEmployee = {
  name: {
    first: "",
    last: "",
  },
  years: 0,
};

const employee = Object.assign({}, defaultEmployee);

employee.name.first = "Joe";

console.log(employee);
// {
//   name: {
//     first:'Joe',
//     last: '',
//   },
//   years: 0
// }

console.log(defaultEmployee);
// {
//   name: {
//     first:'Joe',
//     last: '',
//   },
//   years: 0
// }

// employee에서 중첩된 객체에 대한 참조를 담고 있다. (shallow copy)
// 중첩된 객체도 복사하는 방법 (deep copy)
```

```js
// solve
const defaultEmployee = {
  name: {
    first: "",
    last: "",
  },
  years: 0,
};

const employee2 = Object.assign({}, defaultEmployee, {
  name: Object.assign({}, defaultEmployee.name),
});

employee2.name.first = "Joe";

console.log(emplyee2);
// {
//   name: {
//     first: 'Joe',
//     last: ''
//   },
//   years: 0
// }

console.log(defaultEmployee);
// {
//   name: {
//     first: '',
//     last: '',
//   },
//   years: 0,
// }

// 원본 객체가 변경되지 않음
```

## 객체 펼침 연산자로 정보를 갱신하라

```js
const book1 = {
  title: "Reasons and Persons",
  author: "Derek Parfit",
};

// 새로운 값 추가
const update1 = { ...book1, year: 1984 };
// {
//   title: 'Reasons and Persons',
//   author: 'Derek Parfit',
//   year: 1984
// }

const book2 = {
  title: "Reasons and Persons",
  author: "Derek Parfit",
};

// 값 변경
const update2 = { ...book2, title: "Reasons & Persons" };
// { 
//   title: 'Reasons & Persons',
//   author: 'Derek Parfit'
// }
```

중첩된 객체 복사

```js
const defaultEmployee = {
  name: {
    first: "",
    last: "",
  },
  years: 0,
};

const employee = {
  name: { ...defualtEmployee.name },
  years: defaultEmployee.years,
};

employee.name.first = "Joe";
employee.years = 2022;

console.log(employee);
// {
//   name: {
//     first: "Joe",
//     last: "",
//   },
//   years: 2022,
// }

console.log(defaultEmployee);
// {
//   name: {
//     first: "",
//     last: "",
//   },
//   years: 0,
// }
```

## `Map`으로 명확하게 `key-value` 데이터를 갱신하라

`Map`은 `Object`와 비슷하게 `key-value` 방식의 Collection 이다.

### `Object`와 `Map` 비교
- `Object`의 키는 `String`, `Map`의 키는 모든 값(`any`)을 가질 수 있다.
- `Object`는 크기를 수동으로 추적해야하지만, `Map`은 크기를 쉽게 얻을 수 있다.

### `Map`을 사용하는 경우
- `key-value` 쌍이 자주 추가 되거나 삭제되는 경우
- `key`가 문자열이 아니거나, 모든 `key`가 동일한 타입이며 모든 `value`가 동일한 타입일 경우

### example: 강아지 컬렉션

```js
const dogs = [
  {
    이름: "맥스",
    크기: "소형견",
    견종: "보스턴테리어",
    색상: "검정색",
  },
  {
    이름: "도니",
    크기: "대형견",
    견종: "레브라도레트리버",
    색상: "검정색",
  },
  {
    이름: "섀도",
    크기: "중형견",
    견종: "레브라도레트리버",
    색상: "갈색",
  },
]
```

### `Object`로 강아지 콜렉션 다루기

```js
let filters = {};

function addFilters(filters, key, value) {
  filters[key] = value;
}

addFilters(filters, "견종", "레브라도레트리버");
addFilters(filters, "크기", "대형견");
addFilters(filters, "색상", "갈색");

filters["크기"];
// "대형견"

function deleteFilters(filters, key) {
  delete filters[key];
}

deleteFilters(filters, "크기");
filters;
// {
//   "견종": "레브라도레트리버",
//   "색상": "갈색",
// }

function clearFilters(filters) {
  filters = {};
  return filters;
}

clearFilters(filters);
// {}
```

### `Map` 데이터 다루기

```js
// Map 데이터 추가 1
const filters1 = new Map()
  .set("견종", "레브라도레트리버")
  .set("크기", "대형견")
  .set("색상", "갈색");
filters1;
// Map(3) { '견종' => '레브라도레트리버', '크기' => '대형견', '색상' => '갈색' }
filters1.get("크기");
// "대형견"

// Map 데이터 추가 2
const filters2 = new Map(
  [
    ["견종", "레브라도레트리버"],
    ["크기", "대형견"],
    ["색상", "갈색"],
  ]
)
filters2;
// Map(3) { '견종' => '레브라도레트리버', '크기' => '대형견', '색상' => '갈색' }
filters2.get("색상");
// "갈색"

// Map 데이터 삭제
filters2.delete("색상");
// true
filters2.get("색상");
// undefined

// Map 모든 데이터 삭제
filters2.clear();
filters2;
// Map(0) {}

```

### `Map`으로 강아지 컬렉션 함수를 재구성

```js
const petFilters = new Map();
function addFilters(filters, key, value) {
  filters.set(key, value);
}

function deleteFilters(filters, key) {
  filters.delete(key);
}

function clearFilters(filters) {
  filters.clear();
}
```

### 숫자를 `Key`로 사용한 콜렉션을 다룰때

#### `Obeject`를 이용하여 숫자 `Key` 다루기

```js
// error code: Number
const errors = {
  100: "이름이 잘못되었습니다.",
  110: "이름에는 문자만 입력할 수 있습니다.",
  200: "색상이 잘못되었습니다."
};

function isDataVaild(data) {
  if (data.length < 10) {
    // return errors.100 ==> error
    return errors[100];
  }
  return true;
}
isDataValid("123456789");
// "이름이 잘못되었습니다."
Object.keys(errors);
// [ '100', '110', '200' ]
```

#### `Map`을 이용하여 숫자 `Key` 다루기

```js
const errors = new Map([
  [100, "이름이 잘못되었습니다."],
  [110, "이름에는 문자만 입력할 수 있습니다."],
  [200, "색상이 잘못되었습니다."],
]);

errors.get(100);
// "이름이 잘못되었습니다."

errors.keys();
// [Map Iterator] { 100, 110, 200 }
// Map Iterator를 활용하여 Map을 순회할 수 있습니다.
```

## `Map`과 `Spread 연산자(...)`로 `Key-Value` 데이터를 순회하라

### `Object`를 활용하여 데이터 순회

```js
// Object
const filters = {
  색상: "검정색",
  견종: "레브라도레트리버",
}

function getAppliedFilters(filters) {
  const keys = Object.keys(filters);
  keys.sort();
  const applied = [];
  for (const key of keys) {
    applied.push(`${key}: ${filters[key]}`);
  }
  return `선택한 조건은 ${applied.join(', ')} 입니다.`
}

getAppliedFilters(filters);
// '선택한 조건은 견종: 레브라도레트리버, 색상: 검정색 입니다.'
```
> `for...of` 반복문: 컬렉션의 각값을 하나 씩 반환한다. `Key` 값이 `[Symbol.iterable]` 속성을 가지고 있어야 한다.

```js
// Map
// const filters = new Map()
//   .set("색상", "검정색")
//   .set("견종", "레브라도레트리버");

const filters = new Map(
  [
    ["색상", "검정색"],
    ["견종", "레브라도레트리버"],
  ]
);

function checkFilters(filters) {
  for (const entry of filters) {
    console.log(entry);
  }
}

checkFilters(filters);
// [ '색상', '검정색' ]
// [ '견종', '레브라도레트리버' ]

function getAppliedFilters(filters) {
  const applied = [];
  for (const [key, value] of filters) {
    applied.push(`${key}: ${value}`);
  }
  return `선택한 조건은 ${applied.join(", ")} 입니다.`;
}
// '선택한 조건은 색상: 검정색, 견종: 레브라도레트리버 입니다.'

// Map의 경우 배열처럼 sort() 메서드가 없다.
// => Spread 연산자(...)를 활용.

[...filters];
// [ [ '색상', '검정색' ], [ '견종', '레브라도레트리버' ] ]

function sortByKey(a, b) {
  return a[0] > b[0] ? 1 : -1;
}

function getSortedAppliedFilters(filters) {
  const applied = [];
  for (const [key, value] of [...filters].sort(sortByKey)) {
    applied.push(`${key}: ${value}`);
  }
  return `선택한 조건은 ${applied.join(", ")} 입니다.`;
}

getSortedAppliedFilters(filters);
// '선택한 조건은 견종: 레브라도레트리버, 색상: 검정색 입니다.'
```

### 간단하게 만들기

```js
// let applied = []; 같은 수집을 위한 배열을 굳이 만들지 말고 map을 활용하자.
function getAppliedFilters(filters) {
  const applied = [...filters].map(([key, value]) => {
    return `${key}: ${value}`;
  })
  return `선택한 조건은 ${applied.join(", ")} 입니다.`
}

getAppliedFilters(filters);
'선택한 조건은 색상: 검정색, 견종: 레브라도레트리버 입니다.'

function sortByKey(a, b) {
  return a[0] > b[0] ? 1 : -1;
}

function getSortedAppliedFilters(filters) {
  const applied = [...filters]
    .sort(sortByKey)
    .map(([key, value]) => `${key}: ${value}`)
    .join(', ');
  return `선택한 조건은 ${applied} 입니다.`
}

getSortedAppliedFilters(filters);
// '선택한 조건은 견종: 레브라도레트리버, 색상: 검정색 입니다.'
```

### `Map`을 활용한 컬렉션 순회

> 1. `Map`을 `Array`로 변환합니다.
> 2. `Array`를 정렬(`sort`)합니다.
> 3. `Array`에 담긴 `Key-Value`쌍을 "key: value" 형식의 문자열로 변환합니다.
> 4. `Array`의 항목을 연결해서 문자열을 만듭니다.
> 5. 템플릿 리터럴을 이용해서 다른 정보와 함께 문자열로 병합합니다.

## `Map` 생성 시 부수 효과를 피하라

```js
const defaults = new Map(
  [
    ["색상", "갈색"],
    ["견종", "비글"],
    ["지역", "부산"],
  ]
)

const filters = new Map().set("색상", "검정색");

function applyDefaults(map, defaults) {
  for (const [key, value] of defaults) {
    if (!map.has(key)) {
      map.set(key, value);
    }
  }
}

applyDefaults(filters, defaults);
filters;
// Map(3) { '색상' => '검정색', '견종' => '비글', '지역' => '부산' }
```

복사본을 만들어서 조작하는 것으로 변경 

```js
const defaults = new Map(
  [
    ["색상", "갈색"],
    ["견종", "비글"],
    ["지역", "부산"],
  ]
)

const filters = new Map().set("색상", "검정색");

function applyDefaults(map, defaults) {
  const copy = new Map([...map]);
  for (const [key, value] of defaults) {
    if (!copy.has(key)) {
      copy.set(key, value);
    }
  }
  return copy;
}

applyDefaults(filters, defaults);
// Map(3) { '색상' => '검정색', '견종' => '비글', '지역' => '부산' }
```

### `Spread 연산자(...)`로 더 간단하게 `Map` 병합하기

```js
// example
const exFilters = new Map().set("color", "black").set("color", "brown");
exFilters.get("color");
// "brown"

const filters1 = new Map().set("색상", "검정색");
const filters2 = new Map().set("색상", "갈색");

const update = new Map([...filters1, ...filters2]);
update.get("색상");
// "갈색"

// 코드 개선
function applyDefaults(map, defaults) {
  return new Map([...defaults, ...map]);
}
```

## `Set`를 이용해 고윳값을 관리하라

`Set`는 각 고유 항목을 하나씩만 갖는 특화된 배열입니다.

```js
const dogs = [
  {
    이름: "맥스",
    크기: "소형견",
    견종: "보스턴테리어",
    색상: "검정색"
  },
  {
    이름: "도니",
    크기: "대형견",
    견종: "래브라도레트리버",
    색상: "검정색"
  },
  {
    이름: "섀도",
    크기: "중형견",
    견종: "래브라도레트리버",
    색상: "갈색"
  },
];

// 색상 목록만 수집할려고 할 때
// # 1
function getColors(dogs) {
  return dogs.map(dog => dog["색상"]);
}
getColors(dogs);
// ! 중복값이 존재

// 중복을 제거하는 함수
function getUnique(attributes) {
  const unique = [];
  for (const attribute of attributes) {
    if (!unique.includes(attribute)) {
      unique.push(attribute);
    }
  }
  return unique;
}

const colors = getColors(dogs);
getUnique(colors);
// [ '검정색', '갈색' ]
```


```js
// # 2
// Set를 사용한 고유값 다루기
const colorsArray = ["검정색", "검정색", "갈색"];
const unique = new Set(colorsArray);
// Set(2) { '검정색', '갈색' }

function getUnique(attributes) {
  return [...new Set(attributes)];
}
getUnique(colorsArray);
// [ '검정색', '갈색' ]
```

### `Set` 데이터 추가

```js
let names = new Set();
names.add("joe");
// Set(1) { 'joe' }
names.add("bea");
// Set(2) { 'joe', 'bea' }
names.add("joe");
// Set(2) { 'joe', 'bea' }
```

```js
function getUniqueColors(dogs) {
  const unique = new Set();
  for (const dog of dogs) {
    unique.add(dog.색상);
  }
  return [...unique];
}

getUniqueColors(dogs);
// [ '검정색', '갈색' ]
```


