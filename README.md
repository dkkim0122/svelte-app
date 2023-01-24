# 스벨트 튜토리얼

### 변수 사용하기

Vue의 mustache처럼 script 태그 안에서 선언된 변수들을 포함한 expression들을 `{}`에 넣을 수 있음. 간단한 표현식만 가능한듯 하다.

```jsx
// App.svelte

<script>
	let name = 'world'
	let birthYear = 1996
</script>

<h1>Hello {name.toUpperCase()}!</h1>
<h2>
	Your age is {new Date().getFullYear() - birthYear}
</h2>
```

### HTML 태그 안의 Attribute로 사용하기

```jsx
<script>
	let name = 'Dokyung Kim'
	let src = '/tutorial/image.gif';
</script>

<img src={src} alt="{name} dances" />
<img {src} alt="{name} dances" />
```

밑의 img 태그와 같이 속성 이름과 변수명이 같을 때 속성 선언 구문을 생략할 수 있는 특성을 `shorthand property`라 한다.

### 스타일 사용

```jsx
<p>This is a paragraph.</p>

<style>
	p {
		color: violet;
		font-size: 2rem;
		font-family: 'Comic Sans MS', cursive;
	}
</style>
```

다음과 같이 style 태그 안에 CSS 구문들을 넣어서 스타일을 만들어 줄 수 있다. Vue와 동일하게 **`scoped` 스타일 구문을 사용**한다.

### 컴포넌트 사용

컴포넌트의 이름은 무조건 대문자로 시작하도록 한다. HTML 태그와 컴포넌트를 구분지어줄 수 있다.

```jsx
<p>This is a paragraph.</p>
<Nested />

<script>
import Nested from './Nested.svelte'
</script>

<style>
	p {
		color: purple;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>
```

### text 안의 HTML 태그 적용

마치 Vue의 v-html과 똑같은 역할을 해 준다. 마찬가지로 sanitize는 사용자가 직접 적용해야 하는 부분으로, XSS 공격을 받을 수 있으니 사용자 입력이 들어오는 부분에는 사용하지 말 것(아니면 sanitize 후 사용할 것).

```jsx
<script>
	let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<p>{string}</p>
<p>
	{@html string}
</p>
```

### 이벤트 핸들러 부착

다음과 같이 `on:이벤트명={이벤트 핸들러}`로 이벤트 핸들러를 부착해줄 수 있다.

```jsx
<script>
  let count = 0

  function incrementCount() {
    count += 1
  }
</script>

<section>
  <button on:click={incrementCount}>
    Clicked {count} {count === 1 ? 'time' : 'times'}
  </button>
</section>
```

### reactive decleration

다음과 같이, 동기적으로 데이터가 변화함에 따라 반응형으로 변화하는 데이터를 선언해줄 수 있다. 

```jsx
let count = 0
$: doubled = count * 2
```

달러 표시(`$`)를 사용하면, 이 변수가 참조하고 있는 데이터가 변경되면 해당 로직을 언제든지 재실행하게 된다. 뷰에서는 `computed` 변수가 이 친구와 똑같은 역할을 하는 거 같다.

```jsx
setup() {
	const count = ref(0)
	const doubled = computed(() => count * 2)
}
```

### reactive statements

어떤 값만을 반응성 있게 반환할 수 있는 것은 아니다. 어떤 action을 수행하는 statement(선언문) 역시 반응성 있게 호출할 수 있다.

```jsx
$: if (count >= 10) {
  alert('Count is dangerously high!!')
  count = 9
}
```

### 배열과 객체 업데이트하기

스벨트에서 반응성은 할당에 의해 일어난다. 따라서 배열이나 객체를 변경하는 함수를 호출한다고 해도 새로 할당된 것이 아니면 반응성이 제대로 동작하지 않는다. 밑의 예제에서 버튼을 클릭하여 `addNumber` 메서드를 호출해도, `sum`이라는 반응성 데이터의 값은 재호출되어 계산되지 않는다. `numbers`라는 배열이 재할당된 것이 아니기 때문에 변경이 일어났다는 것을 파악하지 못하기 때문.

```jsx
<script>
  let numbers = [1, 2, 3, 4]

  function addNumber() {
    numbers.push(numbers.length + 1)
  }

  $: sum = numbers.reduce((acc, cur) => acc + cur, 0)
</script>

<section>
  <p>{numbers.join(' + ')} = {sum}</p>
  <button on:click={addNumber}>
    Add a number
  </button>
</section>
```

`numbers`에 새 요소들이 push되지만, `sum`은 `numbers`의 값이 변경되었다는 것을 알 수 없다.

이 때 반응성을 가지고 배열을 변화시키기 위해서는 두 가지가 있다.

- 직접 새 배열을 할당하는 것.

```jsx
function addNumber() {
	numbers.push(numbers.length + 1)
	numbers = numbers
}

function addNumber() {
  numbers = [...numbers, numbers.length + 1]
}
```

`push` 뿐만 아니라, `pop`, `shift`, `splice`, `Map.set`, Set.add 등의 배열과 객체 메서드들에게 모두 해당되는 이야기이다.

- 배열이나 객체의 새 속성에 값을 부여하는 것.

```jsx
function addNumber() {
  numbers[numbers.length] = numbers.length + 1
}
```

객체의 경우, 다음과 같이 원래 있었던 속성에 재할당하는 경우는 당연히 반응성이 부여된다.

```jsx
let obj = { a: 1 }

function addNumber() {
	obj.a += 1
}
```

그러나 다음과 같이 indirect한 할당의 경우에는 반응성이 끊긴다.

```jsx
function addNumber() {
	let a = obj.a
	a += 1
}
```