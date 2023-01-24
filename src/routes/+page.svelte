<script>
  let numbers = [1, 2, 3, 4]
  const obj = { a: 1 }

  function addNumber() {
    numbers[numbers.length] = numbers.length + 1
  }

  function addNumberObj(obj) {
    obj.a += 1
  }

  $: sum = numbers.reduce((acc, cur) => acc + cur, 0)
</script>

<section>
  <p>{obj.a}</p>
  <button on:click={addNumberObj(obj)}>
    Add a number
  </button>
</section>

