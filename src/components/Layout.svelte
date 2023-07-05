<div class="wrapper">
  <header>
    <nav>
      <a href="/" class="brand">Typing Practice</a>
      <slot name="nav-items"></slot>
    </nav>
  </header>
  <slot name="main" />
  <footer>
    <p>Made by <a href="https://github.com/OhBeomho">OhBeomho</a></p>
    <p>
      Source on <a href="https://github.com/OhBeomho/typing-practice">GitHub</a>
    </p>
    <p>Using <a href="https://random-word-api.vercel.app/">Rando</a> to generate words</p>
  </footer>
</div>

<style>
  :global(body) {
    margin: 0;
    height: 100vh;
  }

  :global(a) {
    text-decoration: none;
    color: rgb(0, 100, 255);
  }

  :global(a:hover) {
    text-decoration: underline;
  }

  .wrapper {
    height: 100%;
    display: flex;
    flex-direction: column;
  }

  nav {
    padding: 10px;
    box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.15);
    background-color: black;
    color: white;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  nav .brand {
    color: white;
    font-size: 25px;
  }

  footer {
    background-color: gray;
    text-align: center;
  }
</style>
