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

        acc -= (1 / wordCount) * 100
        acc = Math.round(acc * 100) / 100
      }

      currentWord++
      values.push(currentValue)
      e.target.value = currentValue = ""
      e.preventDefault()

      if (currentWord === words.length) {
        const elapsedTime = (new Date().getTime() - started) / 1000 / 60
        const wpm = Math.round(wordCount / elapsedTime)
        started = null

        alert(
          `Result\n\n${wpm} wpm\nAccuracy: ${acc}%\n${correct} correct words\n${incorrect} incorrect words`
        )

        previousWpm = wpm
        previousAcc = Math.round(acc)

        acc = 100
        correct = 0
        incorrect = 0
        currentWord = 0
        values = []

        generateWords()
      }
    }
  }

  function onInput(e) {
    currentValue = e.target.value

    if (e.target.value && e.target.value !== words[currentWord].slice(0, e.target.value.length)) {
      isCorrect = false

      let incorrectRanges = []

      let incorrectStart = -1
      for (let i = 0; i < currentValue.length; i++) {
        const valueChar = currentValue.charAt(i)
        const wordChar = words[currentWord].charAt(i)

        if (incorrectStart === -1 && valueChar !== wordChar) {
          incorrectStart = i
        } else if (incorrectStart !== -1 && valueChar === wordChar) {
          incorrectRanges.push([incorrectStart, i])
          incorrectStart = -1
        }
      }

      if (incorrectStart !== -1) {
        incorrectRanges.push([incorrectStart, currentValue.length])
      }

      let offset = 0

      for (let incorrectRange of incorrectRanges) {
        const html = `<span style="color: red">${currentValue.substring(
          incorrectRange[0] + offset,
          incorrectRange[1] + offset
        )}</span>`

        currentValue =
          currentValue.substring(0, incorrectRange[0] + offset) +
          html +
          currentValue.substring(incorrectRange[1] + offset)
        offset += html.length - (incorrectRange[1] - incorrectRange[0])
      }
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
  let previousWpm = 0
  let previousAcc = 0
  let values = []

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
        <br /><br />
        <span class="prev-wpm">Previous: {previousWpm}wpm ({previousAcc}%)</span>
      </p>
      <div class="words">
        {#each words as word}
          <span
            class={"word" +
              (word === words[currentWord]
                ? " current"
                : words.indexOf(word) < currentWord
                ? " previous"
                : "")}
            >{word}&nbsp;
            <span class="value"
              >{@html word === words[currentWord]
                ? currentValue
                : values[words.indexOf(word)] || ""}</span
            >
          </span>
        {/each}
        <input type="text" on:input={onInput} on:keydown={onKeydown} />
      </div>
      <div class="focus">Click here to focus</div>
    {/if}
  </main>
</Layout>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Roboto+Mono&display=swap");

  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .focus {
    opacity: 1;
    position: relative;
    top: -180px;
    z-index: -1;
  }

  .prev-wpm {
    font-size: 12px;
  }

  .words {
    max-width: 600px;
    max-height: 200px;
    width: 90vw;
    height: 100vh;
    overflow: hidden;
    filter: blur(2px);
    opacity: 0.5;
    user-select: none;
    position: relative;
    font-family: "Roboto Mono", monospace;

    & input {
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      padding: 0;
      font: inherit;
      opacity: 0;
    }

    & .word {
      color: gray;
      position: relative;

      & .value {
        position: absolute;
        top: 0;
        left: 0;
        color: black;
      }

      &.current {
        color: lightgray;
      }

      &.previous {
        color: rgb(230, 230, 230);
      }
    }
  }

  .words:has(input:focus) {
    filter: none;
    opacity: 1;

    & ~ .focus {
      opacity: 0;
    }
  }

  .nav-items button {
    margin: 0 5px;
  }
</style>
