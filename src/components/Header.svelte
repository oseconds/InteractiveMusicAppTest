<script>
  // Header 컴포넌트
</script>

<header>
  <div class="logo">인터랙티브 음악 학습</div>
  <nav class="main-nav">
    <ul>
      <li><a href="#about">소개</a></li>
      <li><a href="https://github.com/oseconds/InteractiveMusicAppTest" target="_blank">GitHub</a></li>
    </ul>
  </nav>
</header>

<style>
  header {
    background-color: #1a1a1a;
    color: white;
    padding: 1rem 2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  }

  .logo {
    font-size: 1.5rem;
    font-weight: bold;
    color: #ff6b6b;
  }

  .main-nav ul {
    display: flex;
    list-style: none;
    gap: 2rem;
    margin: 0;
    padding: 0;
  }

  .main-nav a {
    color: white;
    text-decoration: none;
    transition: color 0.2s;
  }

  .main-nav a:hover {
    color: #ff6b6b;
  }
</style>
