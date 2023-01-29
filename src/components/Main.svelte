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

  let randomNumberPromise;

  async function handleClick() {
    randomNumberPromise = getRandomNumber()
  }

</script>

<button on:click={handleClick}>
	Get Random Number
</button>

{#await randomNumberPromise then number}
<p>{number}</p>
{:catch error}
<p>{error}</p>
{/await}