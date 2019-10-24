<script>
  import { onMount } from "svelte";
  let data = [];
  let url = "http://redtweetbluetweet.ngrok.io/predict";
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
