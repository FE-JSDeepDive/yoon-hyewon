# Q1. 정규표현식으로 문자열에서 HTML 태그를 제거하려면 어떻게 하나요?

# A1.

- <[^>]+>: <로 시작해서 >로 끝나는 태그 전체를 의미함.

```js
const html = "<p>Hello <strong>world</strong></p>";
const text = html.replace(/<[^>]+>/g, ""); // → "Hello world"
```

# Q2. Symbol을 사용하여 외부 접근을 막을 수 있나요? 완전한 캡슐화가 가능한가요?

# A2.

- Symbol은 객체의 key로 사용되면 외부에서 우연히 접근하기 어렵게 만들어 줄 수는 있지만, 완전한 캡슐화(정보 은닉) 수단은 아닙니다.
- Symbol은 고유한 값이기 때문에, 객체의 속성 key로 사용하면 우연히 이름이 겹쳐서 속성이 덮어씌워지는 일을 방지할 수 있습니다. 하지만, 완전히 숨겨진 값은 아니고 Object.getOwnPropertySymbols()로 접근 가능합니다.

**🔐 진짜 캡슐화가 필요하면?**

- #privateField (ES2022 이후) 또는 클로저 기반 패턴을 사용해야 함.
- 클래스 내부 전용으로 # prefix를 붙이면 외부 접근이 완전 차단됨.

# Q3. Symbol을 for...in이나 Object.keys()로 접근할 수 없는 이유는 무엇인가요?

# A3.

- for...in이나 Object.keys()는 **“열거 가능한 문자열 키”**만 대상으로 하기 때문입니다.
  Symbol은 **열거 불가능(enumerable: false)**하며, 문자열 키가 아니기 때문에 기본 반복문에서 제외됩니다.

```js
const sym = Symbol("hidden");
const obj = {
  visible: "보이는 값",
  [sym]: "숨겨진 값",
};

for (let key in obj) {
  console.log(key); // visible
}

console.log(Object.keys(obj)); // ['visible']
```
