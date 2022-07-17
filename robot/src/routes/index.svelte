
<script lang='ts'>
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js?client';

  if (browser) {  // client-only code here
    const toto = new (window as any).KMMotorOneWebBLE();
    console.warn(toto)
  }

  let count = 0;
</script>

<input type=number bind:value={count}>
<button on:click="{() => count += 1}">count</button>
