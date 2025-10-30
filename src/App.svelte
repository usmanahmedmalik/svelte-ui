<script>
  let count = 0;

  function increment() {
    count += 1;
  }
</script>

<main>
  <h1>Hello from Svelte 👋</h1>
  <p>Count: {count}</p>
  <button on:click={increment}>Add +1</button>
</main>

<style>
  main {
    text-align: center;
    padding: 2rem;
    font-family: system-ui, sans-serif;
  }
  button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
  }
</style>
