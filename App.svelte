<script>
  import { createHmac } from "crypto";
  import { onMount } from "svelte";

  let serverSeed = "";
  let clientSeed = "";
  let nonce = "";
  let cursor = 0;
  let count = 1;
  let results = [];
  let display_bytes = [];
  let DisplayResult = [];

  const generateRandomNumbers = () => {
    results = generateFloats({ serverSeed, clientSeed, nonce, cursor, count });
  };
  const generateDisplayBytes = () => {
    display_bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
  };
  const generateDisplayResult = () => {
    DisplayResult = generateResult({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
  };

  function generateResult({ serverSeed, clientSeed, nonce, cursor, count }) {
    if (activeTab === "limbo") {
      let endresult =
        generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) *
        16777216;
      endresult = (16777216 / (endresult + 1)) * (1 - 0.01);

      console.log(endresult.toFixed(2));
    }
  }

  function generateBytes({ serverSeed, clientSeed, nonce, cursor, count }) {
    // Setup cursor variables
    let currentRound = Math.floor(cursor / 32);
    let currentRoundCursor = cursor;
    currentRoundCursor -= currentRound * 32;

    // Generate bytes until count is fulfilled
    const bytes = [];
    while (bytes.length < count) {
      // HMAC function used to output provided inputs into bytes
      const hmac = createHmac("sha256", serverSeed);
      hmac.update(`${clientSeed}:${nonce}:${currentRound}`);
      const buffer = hmac.digest();

      // Update cursor for next iteration of loop
      while (currentRoundCursor < 32 && bytes.length < count) {
        bytes.push(buffer[currentRoundCursor]);
        currentRoundCursor += 1;
      }
      currentRoundCursor = 0;
      currentRound += 1;
    }

    return bytes;
  }

  function generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) {
    const bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count: count * 4
    });

    // Convert bytes to floats
    const floats = [];
    for (let i = 0; i < bytes.length; i += 4) {
      let partial = 0;
      for (let j = 0; j < 4; j++) {
        partial += bytes[i + j] / 256 ** (j + 1);
      }
      floats.push(partial);
    }

    return parseFloat(floats);
  }

  let activeTab = "limbo";
</script>

<nav>
  
<a class:active={activeTab === 'limbo'} href="#" on:click={() => activeTab = 'limbo'}>Limbo</a>
  <a class:active={activeTab === 'dice'} href="#" on:click={() => activeTab = 'dice'}>Dice</a>
  <a class:active={activeTab === 'mines'} href="#" on:click={() => activeTab = 'mines'}>Mines</a>
  <a class:active={activeTab === 'keno'} href="#" on:click={() => activeTab = 'keno'}>Keno</a>  
</nav>

<label>
  Server Seed:
  <input bind:value={serverSeed} />
</label>

<br>

<label>
  Client Seed:
  <input bind:value={clientSeed} />
</label>

<br>


<label>
  Nonce:
  <input bind:value={nonce} />
</label>
<br>

<div style="margin-left:1rem">

<button class="button" on:click={generateDisplayResult} on:click={generateRandomNumbers} on:click={generateDisplayBytes}>Verify</button>

</div>

{#if display_bytes.length > 0}
  <ul>
    
      <li>{results}</li>
    
  </ul>
{/if}
{#if display_bytes.length > 0}
<ul>
{display_bytes}
</ul>
{/if}


<style>
a {
  margin: 4px;
}

nav {
  margin-bottom: 1rem;
}

.active {
  color: #ff3d03;
  font-weight: 700;
}

.button {
  margin: 1rem;
}

label {
  display: inline-block;
  width: 210px;
  text-align: center;
}
</style>