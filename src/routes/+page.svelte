<script>
  import { onMount } from 'svelte';
  import { fade, fly } from 'svelte/transition';

  let showContent = false;
  let debateTopic = '';
  let isDebating = false;
  let showTopic = false;
  let audio;
  if (typeof window !== 'undefined') {
    audio = new Audio('/life.mp3');
  }
  let isPlaying = false;

  onMount(() => {
    setTimeout(() => {
      showContent = true;
    }, 2000);
  });

  function handleDebate() {
    if (!isDebating && debateTopic.trim()) {
      isDebating = true;
      showTopic = true;
      setTimeout(() => {
        showTopic = false;
        setTimeout(() => {
          isDebating = false;
        }, 500);
      }, 5000);
    } else {
      isDebating = false;
    }
  }

  function togglePlay() {
    isPlaying = !isPlaying;
    isPlaying ? audio.play() : audio.pause();
  }
</script>

<main>
  <div class="museum-scene">
    {#if !showContent}
      <div class="entrance" transition:fade={{ duration: 2000 }}>
        <div class="door">
          <h1>Enter The Philosopher's Chamber</h1>
        </div>
      </div>
    {:else}
      <div class="museum-interior" transition:fade={{ duration: 1000 }}>
        <div class="central-stage">
          <div class="busts">
            <div class="bust socrates" transition:fly={{ x: -200, duration: 1500, delay: 500 }}></div>
            <div class="bust aristotle" transition:fly={{ x: 200, duration: 1500, delay: 500 }}></div>
          </div>
          {#if showTopic}
            <div class="topic" transition:fade={{ duration: 500 }}>
              <h2>{debateTopic}</h2>
            </div>
          {/if}
          <div class="debate">
            <input type="text" bind:value={debateTopic} placeholder="Enter topic..." />
            <button on:click={handleDebate} class:stop={isDebating}>
              {isDebating ? 'Stop Debate' : 'Start Debate'}
            </button>
          </div>
          <button class="music" on:click={togglePlay}>
            {#if isPlaying} ⏸️ {:else} 🎶 {/if}
          </button>
        </div>
      </div>
    {/if}
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: 'Cinzel', serif;
    overflow: hidden;
  }

  .museum-scene {
    height: 100vh;
    width: 100vw;
    background: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.7));
    position: relative;
  }

  .museum-interior {
    height: 100%;
    width: 100%;
    position: relative;
    overflow: hidden;
  }

  .museum-interior::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-image: url('/images/bg.png');
    background-size: cover;
    background-position: center;
    filter: blur(3px);
    z-index: 1;
  }

  .museum-interior > * {
    position: relative;
    z-index: 2;
  }

  .central-stage {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 90%;
    height: 80vh;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .busts {
    display: flex;
    justify-content: space-between;
    width: 110%;
    height: 80%;
    padding: 2rem;
    margin-top: 60px;
    
  }

  .bust {
    width: 150%;
    height: 150%;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    filter: brightness(1.2);
  }

  .bust.socrates {
    background-image: url('/images/socrates.png');
  }

  .bust.aristotle {
    background-image: url('/images/aristotle.png');
  }

  .debate {
    position: fixed;
    bottom: 30rem;
    display: flex;
    gap: 1rem;
    width: 100%;
    max-width: 800px;
    padding: 0 2rem;
  }

  input[type="text"] {
    flex-grow: 1;
    padding: 1rem 1.5rem;
    font-family: 'Cinzel', serif;
    font-size: 1.2rem;
    border: 2px solid white;
    border-radius: 35px;
    background: transparent;
    color: white;
    box-shadow: 0 0px 0px rgba(255, 255, 255, 0.733);
  }

  input[type="text"]::placeholder {
    color: #ffffff;
  }

  button {
    padding: 1rem 2rem;
    font-size: 1.2rem;
    border: 2px solid white;
    border-radius: 25px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-family: 'Cinzel', serif;
    text-transform: uppercase;
    background: transparent;
    color: white;
  }

  button.stop:hover {
    background: white;
    color: black;
  }

  .music {
    position: fixed;
    top: 0;
    right: 10px;
    padding: 0.5rem 1rem;
    font-size: 1rem;
    border-radius: 5px;
    background-color: rgba(255, 255, 255, 0.26);
    color: white;
    cursor: pointer;
  }

  .music:hover {
    background-color: rgba(255, 255, 255, 0.3);
  }

  .topic h2 {
    font-size: 8rem;
    color: white;
    font-weight: 900;
    text-transform: uppercase;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  }
  .entrance {
    position: absolute;
    inset: 0;
    background: #000;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .door {
    text-align: center;
    color: #DAA520;
    cursor: pointer;
    padding: 2rem;
    border: 2px solid #DAA520;
  }

  .door h1 {
    font-size: 2.5rem;
    margin: 0;
    text-transform: uppercase;
    letter-spacing: 0.2em;
  }
</style>
