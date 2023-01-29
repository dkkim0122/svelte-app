# 스벨트 튜토리얼

## Introduction

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

컴포넌트의 이름은 무조건 대문자로 시작하도록 한다. 그래야 HTML 태그와 컴포넌트를 구분지어줄 수 있다.

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

## Reactivity

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

## Props

### props

부모 컴포넌트에서 자식 컴포넌트로 prop을 내려주기 위해서는 자식 컴포넌트에서 prop을 선언해주어야 한다.

다음과 같이 export 문을 사용하여 prop `answer`를 선언해준다.

```jsx
// Nested.svelte
<script>
  export let answer;
</script>

<p>The answer is {answer}</p>
```

이를 부모에서는 다음과 같이 prop을 내려준다.

```jsx
// Main.svelte
<script>
  import Nested from './Nested.svelte'
</script>

<section>
  <Nested answer={42}/>
</section>
```

다음과 같은 결과가 화면에 나올 것이다!

```
The answer is 42
```

prop을 선언할 때 default 값을 할당해 주면 부모에서 prop을 내려주지 않아도 default 값을 보여준다. 

```jsx
<script>
  export let answer = 0;
</script>
```

만약 default가 없는 상태에서 prop을 받지 않는다면 `undefined`의 값을 가지게 된다.

```
The answer is undefined
```

vue에서는 다음과 같이 자식 컴포넌트에서 props 속성을 Vue 컴포넌트에서 선언해주어야 한다.

```jsx
export default {
  props: {
		propsA: {
			type: String,
			required: true
		},
		propsB: {
			type: Number,
			required: false,
			default: 0
		},
	}
}
```

부모 컴포넌트에서 props를 내려주기 위해서 바인딩을 사용한다.

```jsx
<Nested :props-a="I am prop from Vue" />
```

스벨트 내부에서도 마치 뷰에서의 바인딩과 비슷한 원리로 동작하는 것일텐데 원리가 궁금해지긴 한다.

### Spread Props

만약 여러 props를 한꺼번에 자식 컴포넌트에게 prop으로 주고 싶으면 다음과 같이 객체를 만들어 spread 시키는 방법이 있다.

```jsx
// Info.svelte
<script>
	export let name;
	export let version;
	export let speed;
	export let website;

  let props = $$props
</script>
<h1>{props.name}</h1>

<p>
	The <code>{name}</code> package is {speed} fast.
	Download version {version} from <a href="https://www.npmjs.com/package/{name}">npm</a>
	and <a href={website}>learn more here</a>
</p>
```

자식이 name, version, speed, website 총 4가지의 props를 필요로 할 때, 부모는 이렇게 props를 하나하나 내려줄 수도 있지만,

```jsx
// Main.svelte
<script>
  import Info from "./Info.svelte";

  const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev'
	};
</script>

<Info name={pkg.name} version={pkg.version} speed={pkg.speed} website={pkg.website}/>
```

다음과 같이 spread를 통해 간편하게 내려줄 수도 있다.

```jsx
<Info {...pkg} />
```

pkg 객체 안에 있는 각 속성들의 키가 props의 변수명이 되어 내려가게 된다.

### $$props

부모 컴포넌트가 내려준 모든 props들을 자식 컴포넌트에서 한 번에 확인할 수 있는 키워드가 $$props이다. 그런데 이 키워드는 자식 컴포넌트가 선언하지 않은 prop들도 모두 받아 올 수 있다.

```jsx
// Main.svelte
<script>
  import Info from "./Info.svelte";

  const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev',
    none: 'I have not been declared' // 자식 컴포넌트에서는 선언되지 않았음
	};
</script>

<Info {...pkg} />

// Info.svelte
<script>
	export let name;
	export let version;
	export let speed;
	export let website;

  let notDeclared = $$props.none
</script>

<h1>{notDeclared}</h1>
```

```
I have not been declared
```

> It's not generally recommended, as it's difficult for Svelte to optimise, but it's useful in rare cases.
> 

스벨트 자체에서 최적화하기 힘들어서 그런지 권장하지는 않는 것 같다.

## Logic

### if 블록

HTML 구문에 조건문을 붙이기 위해 스벨트는 다음과 같은 if 블록을 사용한다.

`#`는 블록의 시작을, `:`는 블록이 계속된다는 것을, `/`는 블록이 끝이 난다는 것을 알려주는 데에 사용되는 키워드이다.

```jsx
<script>
  let user = { loggedIn: false }

  function toggle() {
    user.loggedIn = !user.loggedIn
  }
</script>

{#if !user.loggedIn}
<button on:click={toggle}>Log In</button>
{/if}
{#if user.loggedIn}
<button on:click={toggle}>Log Out</button>
{/if}
```

다음과 같이 else 문을 사용할 수도 있다.

```jsx
{#if !user.loggedIn}
<button on:click={toggle}>Log In</button>
{:else}
<button on:click={toggle}>Log Out</button>
{/if}
```

else if 역시 가능하다.

```jsx
<script>
	let x = 7;
</script>

{#if x > 10}
	<p>{x} is greater than 10</p>
{:else if x < 5}
	<p>{x} is less than 5</p>
{:else}
	<p>{x} is between 5 and 10</p>
{/if}
```

Vue에서는 다음과 같이 HTML 요소에 v-if를 바인딩 해 주어 조건문에 따른 분기 처리를 할 수 있는데(**Conditional Rendering**), 스벨트는 아예 블록을 구분하여 분기 처리를 한다는 것이 흥미롭다.

```jsx
<p v-if="x > 10">{x} is greater than 10</p>
<p v-else-if="x < 5">{x} is less than 5</p>
<p v-else>{x} is between 5 and 10</p>
```

### each 블록

loop 문 역시 블록을 사용한다. each 블록을 사용하여 배열이나 객체를 순환할 수 있다.

```jsx
<script>
  let EDMArtists = [
    { name: 'Avicii', song: 'Sunset Jesus' },
    { name: 'Kygo', song: 'Freedom' },
    { name: 'Jonas Blue', song: 'Rise' },
  ]
</script>

{#each EDMArtists as artist, i}
  <li>{i}: {artist.name} => {artist.song}</li>
{/each}
```

### keyed each 블록

each 블록의 문제점은 순환 대상이 되는 배열 혹은 객체의 요소들에게 key 값을 부여해주지 않으면 배열의 업데이트 상태가 순환 블록 안에 제대로 전달되지 못한다는 것이다.

```jsx
// Artist.svelte
<script>
	const firstCharacters = {
    Avicii: 'A',
    Kygo: 'K',
    'Jonas Blue': 'JB'
	}

	// the name is updated whenever the prop value changes...
	export let name;

	// ...but the "firstCharacter" variable is fixed upon initialisation of the component
	const firstCharacter = firstCharacters[name];
</script>

<p>
	<span>The first character for { name } is { firstCharacter }</span>
</p>
```

```jsx
// Main.svelte
<script>
	import Artist from './Artist.svelte';

  let EDMArtists = [
    { id: 0, name: 'Avicii', song: 'Sunset Jesus' },
    { id: 1, name: 'Kygo', song: 'Freedom' },
    { id: 2, name: 'Jonas Blue', song: 'Rise' },
  ]

	function handleClick() {
		EDMArtists = EDMArtists.slice(1);
	}
</script>

<button on:click={handleClick}>
	Remove first Artist
</button>

{#each EDMArtists as artist}
	<Artist name={artist.name}/>
{/each}
```

`handleClick`으로 `EDMArtists`의 첫번째 요소를 없애주고, 해당 요소의 DOM 요소도 같이 없애주고 싶다. 그러나 버튼을 클릭하면 `firstCharacter`가 변하지 않고 그대로인 것을 알 수 있다.

이는 스벨트가 리스트에서 첫번째가 아닌 마지막 DOM 요소를 없애주기 때문이다. 즉 각각의 리스트 DOM 요소들은 변하지 않고 props로 내려주는 name만 업데이트되기 때문에 `Artist` 컴포넌트는 변경된 사항을 알 수 없다. 따라서 `firstCharacter` 변수 역시 그대로이기 때문에 변함이 없는 것이다.

애초에 의도한 대로 리스트의 첫번째 요소와 이에 해당하는 DOM 요소도 같이 없애주기 위해서는 스벨트에게 그 요소를 특정지어줄 수 있어야 한다. 그것을 가능하게 하는 것이 바로 key 속성이고, 다음과 같이 사용한다.

```jsx
{#each EDMArtists as artist (artist.id)}
	<Artist name={artist.name}/>
{/each}
```

### await 블록

만약 비동기적으로 데이터를 받아와야 한다면 어떻게 해야 할까?

다음과 같이 async await이나, Promise.then() 등 비동기 작업을 수행하도록 조치할 수 있는 방식은 많다.

```jsx
<script>
  function getRandomNumber() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const randomNumber = Math.floor(Math.random() * 100)
        if (typeof randomNumber === 'number') {
          resolve(randomNumber)
        } else {
          reject('Failed to get random number')
        }
      }, 500)
    })
  }

  let randomNumber = 0;

  async function handleClick() {
    try {
      randomNumber = await getRandomNumber()
    } catch (error) {
      throw new Error(error)
    }
  }

</script>

<button on:click={handleClick}>
	Get Random Number
</button>

<p>{randomNumber}</p>
```

그러나 스벨트에서는 다른 방식도 고려해볼 수 있다. 바로 await 블록을 이용하는 것이다.

```jsx
<script>
  function getRandomNumber() {
    return new Promise((resolve, reject) => {
			...
    })
  }

  let randomNumberPromise;

  async function handleClick() {
    randomNumberPromise = getRandomNumber()
  }

</script>

<button on:click={handleClick}>
	Get Random Number
</button>

{#await randomNumberPromise}
<p>...waiting</p>
{:then number}
<p>{number}</p>
{:catch error}
<p>{error}</p>
{/await}
```

await 블록을 이용하면 프로미스의 then과 catch 구문을 사용하는 것처럼 비동기 작업 처리를 해 줄 수 있다.

다음과 같이 프로미스가 resolve된 후의 값만을 렌더링해줄 수도 있다.

```jsx
{#await randomNumberPromise then number}
<p>{number}</p>
{:catch error}
<p>{error}</p>
{/await}
```