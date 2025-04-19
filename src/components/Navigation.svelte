<script>
  export let currentSection;
  export let goToSection;
  
  const sections = [
    { id: 'intro', name: '소개' },
    { id: 'beats', name: '비트 만들기' },
    { id: 'melody', name: '멜로디' },
    { id: 'harmony', name: '화성' },
    { id: 'structure', name: '곡 구성' }
  ];
</script>

<nav class="section-nav">
  <ul>
    {#each sections as section}
      <li class:active={currentSection === section.id}>
        <button on:click={() => goToSection(section.id)}>
          {section.name}
        </button>
      </li>
    {/each}
  </ul>
</nav>

<style>
  .section-nav {
    background-color: #2a2a2a;
    padding: 1rem;
  }
  
  ul {
    display: flex;
    list-style: none;
    gap: 1rem;
    margin: 0;
    padding: 0;
    justify-content: center;
    flex-wrap: wrap;
  }
  
  li {
    margin: 0;
  }
  
  button {
    background: none;
    border: none;
    color: #ccc;
    font-size: 1rem;
    padding: 0.5rem 1rem;
    cursor: pointer;
    border-radius: 4px;
    transition: all 0.2s;
  }
  
  button:hover {
    background-color: #3a3a3a;
    color: white;
  }
  
  .active button {
    background-color: #ff6b6b;
    color: white;
  }
</style>
