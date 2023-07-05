<script>
  import { onMount } from "svelte"
  import Layout from "../components/Layout.svelte"

  async function generateWords(n) {
    if (n) wordCount = n
    words = undefined

    const response = await fetch(`https://random-word-api.vercel.app/api?words=${wordCount}`)
    words = await response.json()
  }

  function onKeydown(e) {
    if (!started) {
      started = new Date().getTime()
    }

    if ((e.key === " " || e.key === "Enter") && e.target.value) {
      if (e.target.value === words[currentWord]) {
        correct++
      } else {
        incorrect++

        acc -= Math.round((1 / wordCount) * 100)
      }

      currentWord++
      e.target.value = currentValue = ""
      e.preventDefault()

      if (currentWord === words.length) {
        const elapsedTime = (new Date().getTime() - started) / 1000 / 60
        started = null

        alert(
          `Result\n\n${Math.round(
            wordCount / elapsedTime
          )} wpm\nAccuracy: ${acc}%\n${correct} correct words\n${incorrect} incorrect words`
        )

        acc = 100
        correct = 0
        incorrect = 0
        currentWord = 0

        generateWords()
      }
    }
  }

  function onInput(e) {
    currentValue = e.target.value

    if (e.target.value && e.target.value !== words[currentWord].slice(0, e.target.value.length)) {
      isCorrect = false
    } else {
      isCorrect = true
    }
  }

  let words
  let currentWord = 0
  let isCorrect = true
  let correct = 0,
    incorrect = 0,
    acc = 100,
    wordCount = 30
  let currentValue = ""
  let started

  onMount(generateWords)
</script>

<Layout>
  <div slot="nav-items" class="nav-items">
    {#each [15, 30, 60] as n}
      <button on:click={() => generateWords(n)} disabled={started}>{n} words</button>
    {/each}
  </div>
  <main slot="main">
    <h1>Typing Practice</h1>
    {#if !words}
      <p style="color: gray">Loading words...</p>
    {:else}
      <p>
        <b>{currentWord}</b> / {wordCount} words
      </p>
      <p style={{ fontsize: 12 }}>
        Correct: <span style="color: rgb(0, 150, 0)">{correct}</span><br />
        Incorrect: <span style="color: rgb(200, 0, 0)">{incorrect}</span><br />
        Accuracy: <b>{acc}</b>%
      </p>
      <div class="word">
        {words[currentWord]}
        <span class="next">
          {words[currentWord + 1] || ""}
        </span>
        <input on:keydown={onKeydown} on:input={onInput} />
        <p style={`min-height: 30px; color: ${isCorrect ? "black" : "rgb(200, 0, 0)"}`}>
          {currentValue}
        </p>
      </div>
      <div class="focus">Click here to focus</div>
    {/if}
  </main>
</Layout>

<style>
  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .word {
    position: relative;
    padding: 20px;
    font-size: 22px;
    filter: blur(1.5px);
    opacity: 0.7;
    text-align: center;
    width: 100vw;
    max-width: 300px;
    border: 1.5px solid black;
  }

  .focus {
    opacity: 1;
    position: relative;
    top: -82px;
    z-index: -1;
  }

  .next {
    position: absolute;
    right: 10px;
    color: gray;
  }

  .word:has(input:focus) {
    filter: none;
    opacity: 1;
  }

  .word:has(input:focus) ~ .focus {
    opacity: 0;
  }

  .word > input {
    all: unset;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
  }

  .nav-items button {
    margin: 0 5px;
  }
</style>
