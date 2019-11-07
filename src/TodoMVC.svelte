<script>
  const ENTER_KEY = 13;
  const ESCAPE_KEY = 27;

  let currentFilter = "all";
  let items = [];
  let editing = null;

  try {
    items = JSON.parse(localStorage.getItem("todos-svelte")) || [];
  } catch (err) {
    items = [];
  }

  const updateView = () => {
    currentFilter = "all";
    if (window.location.hash === "#/active") {
      currentFilter = "active";
    } else if (window.location.hash === "#/completed") {
      currentFilter = "completed";
    }
  };

  window.addEventListener("hashchange", updateView);
  updateView();

  function clearCompleted() {
    items = items.filter(item => !item.completed);
  }

  function remove(index) {
    items = items.slice(0, index).concat(items.slice(index + 1));
  }

  function toggleAll(event) {
    items = items.map(item => ({
      id: item.id,
      description: item.description,
      completed: event.target.checked
    }));
  }

  async function createNew(event) {
    if (event.which === ENTER_KEY) {
      items = items.concat({
        id: uuid(),
        description: event.target.value,
        completed: false,
        score: await getSentiment(event.target.value).then(r =>
          makePrediction(r)
        )
      });
      event.target.value = "";
    }
  }

  async function getSentiment(draft_tweet) {
    const response = await fetch("https://redtweetbluetweet.appspot.com/predict", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text: draft_tweet })
    });
    const json = await response.json();
    return json;
  }

  function makePrediction(r) {
    const [d_score, r_score] = r[0];
    const prediction =
      d_score > r_score
        ? `${Math.round(d_score * 100)}% D`
        : `${Math.round(r_score * 100)}% R`;
    return prediction;
  }

  function handleEdit(event) {
    if (event.which === ENTER_KEY) event.target.blur();
    else if (event.which === ESCAPE_KEY) editing = null;
  }

  function submit(event) {
    items[editing].description = event.target.value;
    editing = null;
  }

  function uuid() {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
      var r = (Math.random() * 16) | 0,
        v = c == "x" ? r : (r & 0x3) | 0x8;
      return v.toString(16);
    });
  }

  $: filtered =
    currentFilter === "all"
      ? items
      : currentFilter === "completed"
      ? items.filter(item => item.completed)
      : items.filter(item => !item.completed);

  $: numActive = items.filter(item => !item.completed).length;

  $: numCompleted = items.filter(item => item.completed).length;

  $: try {
    localStorage.setItem("todos-svelte", JSON.stringify(items));
  } catch (err) {
    // noop
  }
</script>

<header class="header">
  <h1>rtbt</h1>
  <input
    class="new-todo"
    on:keydown={createNew}
    placeholder="Enter your draft tweet text here:"
    autofocus />
</header>

{#if items.length > 0}
  <section class="main">
    <input
      id="toggle-all"
      class="toggle-all"
      type="checkbox"
      on:change={toggleAll}
      checked={numCompleted === items.length} />
    <label for="toggle-all">Mark all as complete</label>

    <ul class="todo-list">
      {#each filtered as item, index (item.id)}
        <li
          class="{item.completed ? 'completed' : ''}
          {editing === index ? 'editing' : ''}">
          <div class="view">
            <input
              class="toggle"
              type="checkbox"
              bind:checked={item.completed} />
            <label on:dblclick={() => (editing = index)}>
              {item.description} {item.score}
            </label>
            <button on:click={() => remove(index)} class="destroy" />
          </div>

          {#if editing === index}
            <input
              value={item.description}
              id="edit"
              class="edit"
              on:keydown={handleEdit}
              on:blur={submit}
              autofocus />
          {/if}
        </li>
      {/each}
    </ul>

    <footer class="footer">
      <span class="todo-count">
        <strong>{numActive}</strong>
        {numActive === 1 ? 'item' : 'items'} left
      </span>

      <ul class="filters">
        <li>
          <a class={currentFilter === 'all' ? 'selected' : ''} href="#/">All</a>
        </li>
        <li>
          <a
            class={currentFilter === 'active' ? 'selected' : ''}
            href="#/active">
            Active
          </a>
        </li>
        <li>
          <a
            class={currentFilter === 'completed' ? 'selected' : ''}
            href="#/completed">
            Completed
          </a>
        </li>
      </ul>

      {#if numCompleted}
        <button class="clear-completed" on:click={clearCompleted}>
          Clear completed
        </button>
      {/if}
    </footer>
  </section>
{/if}
