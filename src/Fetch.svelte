<script>
  import { onMount } from "svelte";
  let data = [];
  let url = "http://127.0.0.1:52335/predict";
  onMount(async function() {
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data)
    });
    const json = await response.json();
    data = json;
  });
</script>
