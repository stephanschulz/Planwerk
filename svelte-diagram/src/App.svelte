<script>
  import SystemDiagram from './components/SystemDiagram.svelte';
  import DetailView from './components/DetailView.svelte';
  import Legend from './components/Legend.svelte';
  import BlockHierarchy from './components/BlockHierarchy.svelte';
  import GridDiagram from './components/GridDiagram.svelte';
  import VoiceCorridorDiagram from './components/VoiceCorridorDiagram.svelte';
  import DiagramBuilder from './components/DiagramBuilder.svelte';
  
  let selectedVBrain = $state(null);
  let currentView = $state('builder'); // 'builder', 'corridor', 'grid', 'hierarchy' or 'system'
  
  function handleVBrainClick(vbrain) {
    selectedVBrain = vbrain;
  }
</script>

<main>
  <header>
    <h1>System Architecture Diagrams</h1>
    <p class="subtitle">Visual documentation for lighting control systems</p>
    <div class="view-toggle">
      <button 
        class:active={currentView === 'builder'} 
        onclick={() => currentView = 'builder'}>
        Diagram Builder
      </button>
      <button 
        class:active={currentView === 'corridor'} 
        onclick={() => currentView = 'corridor'}>
        Voice Corridor
      </button>
      <button 
        class:active={currentView === 'grid'} 
        onclick={() => currentView = 'grid'}>
        Grid Diagram
      </button>
      <button 
        class:active={currentView === 'hierarchy'} 
        onclick={() => currentView = 'hierarchy'}>
        Block Hierarchy
      </button>
      <button 
        class:active={currentView === 'system'} 
        onclick={() => currentView = 'system'}>
        vBrain System
      </button>
    </div>
  </header>

  {#if currentView === 'builder'}
    <DiagramBuilder />
  {:else if currentView === 'corridor'}
    <VoiceCorridorDiagram />
  {:else if currentView === 'grid'}
    <GridDiagram />
  {:else if currentView === 'hierarchy'}
    <BlockHierarchy />
  {:else}
    <SystemDiagram onVBrainClick={handleVBrainClick} />
    <DetailView />
    <Legend />
  {/if}
  
  {#if selectedVBrain}
    <div class="selected-info">
      <h3>Selected: {selectedVBrain.name}</h3>
      <button onclick={() => selectedVBrain = null}>Close</button>
      <p>Line: {selectedVBrain.line}</p>
      <p>Group: {selectedVBrain.group}</p>
      <p>Controls: 4 rectangular lights</p>
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 1800px;
    margin: 0 auto;
  }

  header {
    text-align: center;
    margin-bottom: 40px;
    padding: 32px;
    background: white;
    border: 2px solid #000000;
  }

  h1 {
    font-size: 28px;
    font-weight: 400;
    margin-bottom: 8px;
    color: #000000;
    letter-spacing: 0;
  }

  .subtitle {
    font-size: 14px;
    color: #666666;
    line-height: 1.4;
    margin-bottom: 20px;
  }

  .view-toggle {
    display: flex;
    gap: 12px;
    justify-content: center;
  }

  .view-toggle button {
    padding: 10px 24px;
    background: #ffffff;
    color: #000000;
    border: 2px solid #000000;
    cursor: pointer;
    font-weight: 400;
    font-size: 13px;
    font-family: ui-monospace, 'Courier New', monospace;
    transition: all 0.2s ease;
  }

  .view-toggle button:hover {
    background: #f0f0f0;
  }

  .view-toggle button.active {
    background: #ff5722;
    color: #ffffff;
    border-color: #ff5722;
  }

  .selected-info {
    position: fixed;
    bottom: 32px;
    right: 32px;
    background: white;
    padding: 20px;
    border: 2px solid #000000;
    z-index: 1000;
  }

  .selected-info h3 {
    margin-bottom: 12px;
    font-size: 16px;
    font-weight: 400;
    color: #000000;
  }

  .selected-info button {
    margin-top: 12px;
    padding: 8px 16px;
    background: #ff5722;
    color: white;
    border: none;
    cursor: pointer;
    font-weight: 400;
    transition: background 0.2s ease;
  }

  .selected-info button:hover {
    background: #e64a19;
  }

  .selected-info p {
    margin: 6px 0;
    font-size: 13px;
    color: #333333;
    line-height: 1.4;
  }
</style>

