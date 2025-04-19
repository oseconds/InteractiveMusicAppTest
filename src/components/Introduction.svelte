<script>
  // Introduction 컴포넌트
</script>

<section class="introduction">
  <h1>인터랙티브 음악 학습 경험에 오신 것을 환영합니다</h1>
  
  <div class="intro-content">
    <div class="intro-text">
      <p>이 웹사이트는 Svelte와 Tone.js를 사용하여 구현된 인터랙티브 음악 학습 플랫폼입니다.</p>
      <p>Ableton Learning Music에서 영감을 받아 제작되었으며, 실습을 통해 음악의 기초를 배울 수 있습니다.</p>
      <p>상단의 네비게이션을 통해 다양한 음악 요소들을 실험해보세요:</p>
      
      <ul>
        <li><strong>비트 만들기</strong> - 리듬과 드럼 패턴을 만들고 실험해봅니다.</li>
        <li><strong>멜로디</strong> - 음표를 배치하여 멜로디를 만들어봅니다.</li>
        <li><strong>화성</strong> - 화음과 코드 진행을 학습합니다.</li>
        <li><strong>곡 구성</strong> - 음악의 구조와 형식을 배웁니다.</li>
      </ul>
    </div>
    
    <div class="intro-visual">
      <div class="pulse-visual">
        <!-- 간단한 시각적 요소 추가 -->
        <div class="pulse"></div>
        <div class="pulse delay-1"></div>
        <div class="pulse delay-2"></div>
      </div>
    </div>
  </div>
  
  <div class="cta">
    <p>네비게이션 메뉴에서 "비트 만들기"를 클릭하여 시작해보세요!</p>
  </div>
</section>

<style>
  .introduction {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
    color: #eee;
  }
  
  h1 {
    font-size: 2.5rem;
    margin-bottom: 2rem;
    color: #ff6b6b;
    text-align: center;
  }
  
  .intro-content {
    display: flex;
    gap: 2rem;
    margin-bottom: 2rem;
  }
  
  .intro-text {
    flex: 1;
  }
  
  .intro-visual {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 300px;
  }
  
  .pulse-visual {
    position: relative;
    width: 200px;
    height: 200px;
  }
  
  .pulse {
    position: absolute;
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: rgba(255, 107, 107, 0.2);
    animation: pulse 3s infinite;
  }
  
  .delay-1 {
    animation-delay: 1s;
  }
  
  .delay-2 {
    animation-delay: 2s;
  }
  
  @keyframes pulse {
    0% {
      transform: scale(0.5);
      opacity: 1;
    }
    100% {
      transform: scale(1.5);
      opacity: 0;
    }
  }
  
  ul {
    margin: 1rem 0;
    padding-left: 1.5rem;
  }
  
  li {
    margin-bottom: 0.5rem;
  }
  
  .cta {
    background-color: #333;
    padding: 1.5rem;
    border-radius: 8px;
    text-align: center;
    font-size: 1.2rem;
  }
  
  @media (max-width: 768px) {
    .intro-content {
      flex-direction: column;
    }
  }
</style>
