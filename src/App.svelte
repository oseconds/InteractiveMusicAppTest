<script>
  import { onMount } from 'svelte';
  import * as Tone from 'tone';
  import BeatMaker from './components/BeatMaker.svelte';
  import Navigation from './components/Navigation.svelte';
  import Header from './components/Header.svelte';
  import Introduction from './components/Introduction.svelte';
  
  let currentSection = 'intro';
  let audioStarted = false;
  
  // Tone.js 초기화
  async function startAudio() {
    await Tone.start();
    audioStarted = true;
    console.log('오디오 컨텍스트가 시작되었습니다.');
  }
  
  function goToSection(section) {
    currentSection = section;
  }
</script>

<main>
  <Header />
  <Navigation {currentSection} {goToSection} />
  
  <div class="content">
    {#if !audioStarted}
      <div class="start-overlay">
        <h2>인터랙티브 음악 학습 경험을 시작하려면 클릭하세요</h2>
        <button on:click={startAudio}>학습 시작하기</button>
      </div>
    {:else}
      {#if currentSection === 'intro'}
        <Introduction />
      {:else if currentSection === 'beats'}
        <BeatMaker />
      {/if}
    {/if}
  </div>
</main>

<style>
  :global(body) {
    background-color: #121212;
    font-family: 'Inter', 'Helvetica Neue', sans-serif;
    margin: 0;
    padding: 0;
    color: #eee;
    min-height: 100vh;
  }

  main {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }

  .content {
    flex-grow: 1;
    position: relative;
  }

  .start-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.7);
    z-index: 10;
  }

  .start-overlay h2 {
    color: white;
    margin-bottom: 2rem;
    text-align: center;
    max-width: 80%;
  }

  .start-overlay button {
    background-color: #ff6b6b;
    color: white;
    border: none;
    font-size: 1.2rem;
    padding: 1rem 2rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .start-overlay button:hover {
    background-color: #ff8787;
  }

  @media (max-width: 768px) {
    .start-overlay h2 {
      font-size: 1.5rem;
    }
  }
</style>
