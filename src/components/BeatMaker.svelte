<script>
  import { onMount, onDestroy } from 'svelte';
  import * as Tone from 'tone';
  // MidiWriter를 UMD 방식으로 가져오기
  import MidiWriter from 'midi-writer-js';
  
  // 비트 그리드 설정
  let rows = 4; // 드럼 종류
  let cols = 16; // 비트 수
  let grid = createEmptyGrid(rows, cols);
  let velocityGrid = createEmptyVelocityGrid(rows, cols);
  let currentStep = 0;
  let isPlaying = false;
  let tempo = 128; // BPM
  let drumSequencer;
  
  // 벨로시티 설정
  let velocityRandomization = true; // 벨로시티 랜덤화 활성화 상태
  let velocityRange = 0.3; // 벨로시티 변동 범위 (0-1)
  let baseVelocity = 0.7; // 기본 벨로시티 값

  // 드럼 소리 설정
  const drumSounds = [
    { name: '킥', sample: 'https://tonejs.github.io/audio/drum-samples/CR78/kick.mp3' },
    { name: '스네어', sample: 'https://tonejs.github.io/audio/drum-samples/CR78/snare.mp3' },
    { name: '하이햇', sample: 'https://tonejs.github.io/audio/drum-samples/CR78/hihat.mp3' },
    { name: '탐', sample: 'https://tonejs.github.io/audio/drum-samples/CR78/tom1.mp3' }
  ];
  
  // 드럼 샘플러 생성
  const drumSampler = new Tone.Sampler({
    urls: {
      C2: drumSounds[0].sample,
      D2: drumSounds[1].sample,
      E2: drumSounds[2].sample,
      F2: drumSounds[3].sample
    },
    onload: () => {
      console.log("드럼 샘플 로드 완료");
    }
  }).toDestination();
  
  // 비어있는 그리드 생성 함수
  function createEmptyGrid(rows, cols) {
    return Array(rows).fill().map(() => Array(cols).fill(false));
  }

  function createEmptyVelocityGrid(rows, cols) {
    return Array(rows).fill().map(() => Array(cols).fill(1));
  }
  
  // 셀 토글 함수
  function toggleCell(row, col) {
    grid[row][col] = !grid[row][col];
    grid = [...grid]; // Svelte 반응성을 위한 할당
  }
  
  // 재생 시작
  function startSequencer() {
    if (isPlaying) return;
    
    // Transport 설정
    Tone.Transport.bpm.value = tempo;
    currentStep = 0;
    
    // 시퀀서 생성
    drumSequencer = new Tone.Sequence((time, step) => {
      currentStep = step;
      
      // 활성화된 셀의 드럼 소리 재생
      grid.forEach((row, rowIndex) => {
        if (row[step]) {
          const note = ['C2', 'D2', 'E2', 'F2'][rowIndex];
          let velocity = baseVelocity;
          
          // 하이햇(2번 행)과 탐(3번 행)에만 벨로시티 랜덤화 적용
          if (velocityRandomization && (rowIndex === 2 || rowIndex === 3)) {
            velocity += Math.random() * velocityRange - velocityRange / 2;
            velocity = Math.max(0, Math.min(1, velocity)); // Clamp between 0 and 1
          }
          
          drumSampler.triggerAttackRelease(note, '16n', time, velocity);
        }
      });
      
    }, [...Array(cols).keys()], '16n');
    
    // 재생 시작
    drumSequencer.start(0);
    Tone.Transport.start();
    isPlaying = true;
  }
  
  // 재생 중지
  function stopSequencer() {
    if (!isPlaying) return;
    
    drumSequencer.stop();
    Tone.Transport.stop();
    isPlaying = false;
    currentStep = 0;
  }
  
  // BPM 변경 함수
  function updateTempo(e) {
    tempo = parseInt(e.target.value);
    Tone.Transport.bpm.value = tempo;
  }
  
  // 패턴 초기화
  function clearPattern() {
    grid = createEmptyGrid(rows, cols);
    currentStep = 0;
  }
  
  // 램덤 패턴 생성
  function randomPattern() {
    grid = Array(rows).fill().map(() => 
      Array(cols).fill().map(() => Math.random() > 0.8)
    );
  }
  
  // MIDI 파일 생성 및 다운로드 함수
  function exportToMidi() {
    // MIDI 트랙 생성
    const tracks = [];
    
    // GM 드럼 매핑 (일반 MIDI 표준)
    const drumMapping = [
      36, // Kick (Bass Drum 1)
      38, // Snare
      42, // Closed Hi-hat
      45  // Tom 1
    ];
    
    // 각 드럼 종류마다 별도의 트랙 생성
    for (let rowIndex = 0; rowIndex < rows; rowIndex++) {
      // 현재 행에 활성화된 셀이 있는지 확인
      if (grid[rowIndex].some(cell => cell)) {
        const track = new MidiWriter.Track();
        
        // 트랙 이름 설정
        track.addEvent(new MidiWriter.ProgramChangeEvent({instrument: 0})); // GM drum kit
        
        // 노트 이벤트 추가 (16분음표 기준)
        const notes = [];
        
        for (let colIndex = 0; colIndex < cols; colIndex++) {
          if (grid[rowIndex][colIndex]) {
            notes.push({
              pitch: drumMapping[rowIndex],
              duration: '16',
              startTick: colIndex * 16, // 16틱을 16분음표로 간주
              velocity: velocityRandomization && (rowIndex === 2 || rowIndex === 3) ? 
                Math.floor(baseVelocity * 127) : 100
            });
          }
        }
        
        // 드럼 노트 이벤트 추가 (채널 10은 GM 드럼 채널)
        if (notes.length > 0) {
          notes.forEach(note => {
            track.addEvent(new MidiWriter.NoteEvent({
              pitch: note.pitch,
              duration: note.duration,
              startTick: note.startTick,
              velocity: note.velocity,
              channel: 10
            }));
          });
          
          tracks.push(track);
        }
      }
    }
    
    // 모든 트랙이 비어있는지 확인
    if (tracks.length === 0) {
      alert('내보낼 비트 패턴이 없습니다. 먼저 비트를 만들어주세요.');
      return;
    }
    
    // 템포 설정을 위한 트랙 추가
    const tempoTrack = new MidiWriter.Track();
    tempoTrack.addEvent(new MidiWriter.TempoEvent({ bpm: tempo }));
    tracks.unshift(tempoTrack);
    
    // MIDI 파일 생성
    const write = new MidiWriter.Writer(tracks);
    
    // MIDI 파일 다운로드
    const midiBlob = new Blob([write.buildFile()], { type: 'audio/midi' });
    const downloadLink = document.createElement('a');
    downloadLink.href = URL.createObjectURL(midiBlob);
    downloadLink.download = `drum-pattern-${tempo}bpm.mid`;
    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
  }
  
  // 컴포넌트가 제거될 때 정리
  onDestroy(() => {
    if (isPlaying) {
      stopSequencer();
    }
  });
</script>

<section class="beat-maker">
  <h1>비트 메이커</h1>
  
  <div class="controls">
    <div class="playback-controls">
      {#if !isPlaying}
        <button on:click={startSequencer} class="play-btn">
          <span class="icon">▶</span> 재생
        </button>
      {:else}
        <button on:click={stopSequencer} class="stop-btn">
          <span class="icon">■</span> 정지
        </button>
      {/if}
    </div>
    
    <div class="tempo-control">
      <label for="tempo">
        BPM: {tempo}
      </label>
      <input 
        type="range" 
        id="tempo" 
        min="60" 
        max="200" 
        step="1" 
        bind:value={tempo} 
        on:input={updateTempo}
      />
    </div>
    
    <div class="pattern-controls">
      <button on:click={clearPattern} class="clear-btn">패턴 지우기</button>
      <button on:click={randomPattern} class="random-btn">랜덤 패턴</button>
      <button on:click={exportToMidi} class="export-btn">MIDI 내보내기</button>
    </div>
  </div>
  
  <div class="grid-container">
    <div class="labels">
      {#each drumSounds as sound, i}
        <div class="label">
          {sound.name}
          <button class="preview-btn" on:click={() => {
            const note = ['C2', 'D2', 'E2', 'F2'][i];
            drumSampler.triggerAttackRelease(note, '16n');
          }}>
            테스트
          </button>
        </div>
      {/each}
    </div>
    
    <div class="grid">
      {#each grid as row, rowIndex}
        <div class="row">
          {#each row as cell, colIndex}
            <div 
              class="cell {cell ? 'active' : ''} {colIndex === currentStep && isPlaying ? 'current' : ''} {colIndex % 4 === 0 ? 'beat-start' : ''}"
              on:click={() => toggleCell(rowIndex, colIndex)}
            ></div>
          {/each}
        </div>
      {/each}
    </div>
  </div>
  
  <div class="compact-velocity-controls">
    <details>
      <summary>하이햇 & 탐 다이나믹스 설정</summary>
      <div class="velocity-inner">
        <div class="velocity-toggle">
          <label class="switch">
            <input type="checkbox" bind:checked={velocityRandomization}>
            <span class="slider"></span>
          </label>
          <span>벨로시티 랜덤화 {velocityRandomization ? '켜짐' : '꺼짐'}</span>
        </div>
        
        <div class="velocity-settings" class:disabled={!velocityRandomization}>
          <div class="velocity-control">
            <label for="baseVelocity">
              기본 세기: {(baseVelocity * 100).toFixed(0)}%
            </label>
            <input 
              type="range" 
              id="baseVelocity" 
              min="0.1" 
              max="1" 
              step="0.05" 
              bind:value={baseVelocity} 
              disabled={!velocityRandomization}
            />
          </div>
          
          <div class="velocity-control">
            <label for="velocityRange">
              변동 폭: {(velocityRange * 100).toFixed(0)}%
            </label>
            <input 
              type="range" 
              id="velocityRange" 
              min="0" 
              max="0.9" 
              step="0.05" 
              bind:value={velocityRange}
              disabled={!velocityRandomization} 
            />
          </div>
        </div>
      </div>
    </details>
  </div>
  
  <div class="instructions">
    <h3>사용 방법</h3>
    <p>1. 그리드의 셀을 클릭하여 비트 패턴을 만듭니다.</p>
    <p>2. '재생' 버튼을 클릭하여 만든 비트를 들어보세요.</p>
    <p>3. BPM 슬라이더로 템포를 조절할 수 있습니다.</p>
    <p>4. '패턴 지우기'로 초기화하거나 '랜덤 패턴'으로 새로운 패턴을 시도해보세요.</p>
    <p>5. '하이햇 & 탐 다이나믹스 설정'에서 하이햇과 탐의 벨로시티 랜덤화를 조절할 수 있습니다.</p>
    <p>6. 'MIDI 내보내기' 버튼을 클릭하여 만든 비트 패턴을 MIDI 파일로 저장할 수 있습니다.</p>
  </div>
</section>

<style>
  .beat-maker {
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
  
  .controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
    flex-wrap: wrap;
    gap: 1rem;
  }
  
  .playback-controls button {
    font-size: 1.2rem;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  
  .play-btn {
    background-color: #4caf50;
    color: white;
  }
  
  .stop-btn {
    background-color: #f44336;
    color: white;
  }
  
  .tempo-control {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    min-width: 200px;
  }
  
  .tempo-control input[type="range"] {
    width: 100%;
  }
  
  .pattern-controls {
    display: flex;
    gap: 0.5rem;
  }
  
  .pattern-controls button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    background-color: #333;
    color: white;
  }
  
  .clear-btn:hover {
    background-color: #555;
  }
  
  .random-btn {
    background-color: #673ab7;
  }
  
  .random-btn:hover {
    background-color: #7e57c2;
  }
  
  .export-btn {
    background-color: #2196f3;
    color: white;
  }
  
  .export-btn:hover {
    background-color: #1976d2;
  }
  
  /* 벨로시티 컨트롤 스타일 */
  .compact-velocity-controls {
    background-color: #2a2a2a;
    border-radius: 8px;
    padding: 0.5rem;
    margin-bottom: 1rem;
    font-size: 0.9rem;
  }
  
  .compact-velocity-controls summary {
    cursor: pointer;
    padding: 0.5rem;
    font-weight: bold;
    color: #ff6b6b;
  }
  
  .compact-velocity-controls details[open] summary {
    margin-bottom: 0.5rem;
    border-bottom: 1px solid #444;
    padding-bottom: 0.5rem;
  }
  
  .velocity-inner {
    padding: 0.5rem;
  }
  
  .velocity-toggle {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  
  .switch {
    position: relative;
    display: inline-block;
    width: 50px;
    height: 24px;
  }
  
  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }
  
  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #555;
    transition: 0.4s;
    border-radius: 24px;
  }
  
  .slider:before {
    position: absolute;
    content: "";
    height: 18px;
    width: 18px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: 0.4s;
    border-radius: 50%;
  }
  
  input:checked + .slider {
    background-color: #ff6b6b;
  }
  
  input:checked + .slider:before {
    transform: translateX(26px);
  }
  
  .velocity-settings {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  
  .velocity-settings.disabled {
    opacity: 0.5;
  }
  
  .velocity-control {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .velocity-control input[type="range"] {
    width: 100%;
  }
  
  .grid-container {
    display: flex;
    margin-bottom: 2rem;
    overflow-x: auto;
  }
  
  .labels {
    display: flex;
    flex-direction: column;
    margin-right: 1rem;
  }
  
  .label {
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding-right: 0.5rem;
    min-width: 100px;
  }
  
  .preview-btn {
    font-size: 0.8rem;
    padding: 0.25rem 0.5rem;
    background-color: #333;
    border: none;
    border-radius: 4px;
    color: white;
    cursor: pointer;
  }
  
  .preview-btn:hover {
    background-color: #444;
  }
  
  .grid {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
  }
  
  .row {
    display: flex;
    height: 50px;
    margin-bottom: 4px;
  }
  
  .cell {
    width: 45px;
    height: 45px;
    margin-right: 4px;
    background-color: #333;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.1s ease;
  }
  
  .beat-start {
    border-left: 2px solid rgba(255, 255, 255, 0.3);
  }
  
  .cell:hover {
    background-color: #444;
  }
  
  .cell.active {
    background-color: #ff6b6b;
    box-shadow: 0 0 10px rgba(255, 107, 107, 0.5);
  }
  
  .cell.current {
    border: 2px solid white;
  }
  
  .instructions {
    background-color: #333;
    padding: 1.5rem;
    border-radius: 8px;
    margin-top: 2rem;
  }
  
  .instructions h3 {
    margin-top: 0;
    color: #ff6b6b;
  }
  
  @media (max-width: 768px) {
    .controls {
      flex-direction: column;
      align-items: flex-start;
    }
    
    .cell {
      width: 35px;
      height: 35px;
    }
  }
</style>
