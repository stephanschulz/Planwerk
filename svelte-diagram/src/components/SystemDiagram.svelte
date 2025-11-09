<script>
  let { onVBrainClick } = $props();
  
  // Generate vBrain positions for Group A
  const groupAVBrains = Array.from({length: 5}, (_, i) => ({
    id: `1a-${i+1}`,
    name: `vB${i+1}`,
    x: 70 + i * 110,
    y: 185,
    line: 1,
    group: 'A'
  }));
  
  // Generate vBrain positions for Group B  
  const groupBVBrains = Array.from({length: 3}, (_, i) => ({
    id: `1b-${i+26}`,
    name: `vB${i+26}`,
    x: 710 + i * 110,
    y: 185,
    line: 1,
    group: 'B'
  }));
  
  // Line 2 vBrains
  const line2AVBrains = Array.from({length: 3}, (_, i) => ({
    id: `2a-${i+1}`,
    name: `vB${i+1}`,
    x: 70 + i * 110,
    y: 425,
    line: 2,
    group: 'A'
  }));
  
  function handleClick(vbrain) {
    if (onVBrainClick) {
      onVBrainClick(vbrain);
    }
  }
</script>

<div class="diagram-container">
  <!-- Line 1 -->
  <svg viewBox="0 0 1360 300" class="line-svg">
    <!-- Background -->
    <rect x="40" y="100" width="1320" height="200" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="50" y="120" class="line-label">LINE 1 (50 vBrains, 200 lights)</text>
    
    <!-- Group A Label -->
    <text x="310" y="145" class="group-label">Group 1A (25 vBrains)</text>
    
    <!-- AC Power Bus Group A -->
    <line x1="50" y1="165" x2="570" y2="165" stroke="#000000" stroke-width="2" />
    <circle cx="50" cy="165" r="4" fill="#000000" />
    <text x="50" y="160" class="bus-label">AC Power</text>
    
    <!-- Group A vBrains -->
    {#each groupAVBrains as vbrain}
      <g class="vbrain-unit" 
         role="button" 
         tabindex="0"
         onclick={() => handleClick(vbrain)}
         onkeydown={(e) => e.key === 'Enter' && handleClick(vbrain)}>
        <!-- Power T-branch -->
        <line x1={vbrain.x + 25} y1="165" x2={vbrain.x + 25} y2={vbrain.y} 
              stroke="#000000" stroke-width="2" />
        <circle cx={vbrain.x + 25} cy="165" r="4" fill="#000000" />
        
        <!-- vBrain box -->
        <rect x={vbrain.x} y={vbrain.y} width="50" height="25" 
              fill="#ffffff" stroke="#000000" stroke-width="2" />
        <text x={vbrain.x + 25} y={vbrain.y + 17} class="vbrain-text">{vbrain.name}</text>
        
        <!-- Lights -->
        {#each Array(4) as _, i}
          <rect x={vbrain.x - 8 + i * 18} y={vbrain.y + 45} width="16" height="12" 
                fill="#ff5722" stroke="#000000" stroke-width="1.5" />
          <line x1={vbrain.x + 25} y1={vbrain.y + 25} x2={vbrain.x + i * 18} y2={vbrain.y + 45}
                stroke="#000000" stroke-width="1" />
        {/each}
      </g>
    {/each}
    
    <!-- Data daisy chain Group A -->
    <path d="M 50 197 L 95 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="95" cy="197" r="3" fill="#000000" />
    <path d="M 120 197 L 205 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="205" cy="197" r="3" fill="#000000" />
    <path d="M 230 197 L 315 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="315" cy="197" r="3" fill="#000000" />
    <path d="M 340 197 L 425 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="425" cy="197" r="3" fill="#000000" />
    <path d="M 450 197 L 535 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    
    <text x="450" y="202" class="dots-text">... (20 more)</text>
    
    <!-- Pass-through -->
    <line x1="570" y1="165" x2="690" y2="165" stroke="#000000" stroke-width="2" />
    <path d="M 560 197 L 690 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <text x="600" y="185" class="passthrough-label">Pass-through</text>
    
    <!-- Group B Label -->
    <text x="950" y="145" class="group-label">Group 1B (25 vBrains)</text>
    
    <!-- AC Power Bus Group B -->
    <line x1="690" y1="165" x2="1310" y2="165" stroke="#000000" stroke-width="2" />
    
    <!-- Group B vBrains -->
    {#each groupBVBrains as vbrain}
      <g class="vbrain-unit" 
         role="button" 
         tabindex="0"
         onclick={() => handleClick(vbrain)}
         onkeydown={(e) => e.key === 'Enter' && handleClick(vbrain)}>
        <!-- Power T-branch -->
        <line x1={vbrain.x + 25} y1="165" x2={vbrain.x + 25} y2={vbrain.y} 
              stroke="#000000" stroke-width="2" />
        <circle cx={vbrain.x + 25} cy="165" r="4" fill="#000000" />
        
        <!-- vBrain box -->
        <rect x={vbrain.x} y={vbrain.y} width="50" height="25" 
              fill="#ffffff" stroke="#000000" stroke-width="2" />
        <text x={vbrain.x + 25} y={vbrain.y + 17} class="vbrain-text">{vbrain.name}</text>
        
        <!-- Lights -->
        {#each Array(4) as _, i}
          <rect x={vbrain.x - 8 + i * 18} y={vbrain.y + 45} width="16" height="12" 
                fill="#ff5722" stroke="#000000" stroke-width="1.5" />
          <line x1={vbrain.x + 25} y1={vbrain.y + 25} x2={vbrain.x + i * 18} y2={vbrain.y + 45}
                stroke="#000000" stroke-width="1" />
        {/each}
      </g>
    {/each}
    
    <!-- Data daisy chain Group B -->
    <path d="M 735 197 L 845 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="845" cy="197" r="3" fill="#000000" />
    <path d="M 870 197 L 980 197" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    
    <text x="950" y="202" class="dots-text">... (22 more vBrains)</text>
    
    <text x="50" y="275" class="note-text">AC Power: Continuous bus with T-branches | Data: Daisy-chained between all 50 vBrains</text>
    
  </svg>

  <!-- Line 2 (simplified) -->
  <svg viewBox="0 0 1360 200" class="line-svg">
    <rect x="40" y="20" width="1320" height="180" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="50" y="40" class="line-label">LINE 2 (50 vBrains, 200 lights)</text>
    
    <!-- Group A Label -->
    <text x="310" y="65" class="group-label">Group 2A (25 vBrains)</text>
    
    <!-- AC Power Bus -->
    <line x1="50" y1="85" x2="570" y2="85" stroke="#000000" stroke-width="2" />
    <circle cx="50" cy="85" r="4" fill="#000000" />
    <text x="50" y="80" class="bus-label">AC Power</text>
    
    <!-- Group A vBrains (first 3) -->
    {#each line2AVBrains as vbrain}
      <g class="vbrain-unit" 
         role="button" 
         tabindex="0"
         onclick={() => handleClick(vbrain)}
         onkeydown={(e) => e.key === 'Enter' && handleClick(vbrain)}>
        <line x1={vbrain.x + 25} y1="85" x2={vbrain.x + 25} y2="105" 
              stroke="#000000" stroke-width="2" />
        <circle cx={vbrain.x + 25} cy="85" r="4" fill="#000000" />
        
        <rect x={vbrain.x} y="105" width="50" height="25" 
              fill="#ffffff" stroke="#000000" stroke-width="2" />
        <text x={vbrain.x + 25} y="122" class="vbrain-text">{vbrain.name}</text>
        
        {#each Array(4) as _, i}
          <rect x={vbrain.x - 8 + i * 18} y="150" width="16" height="12" 
                fill="#ff5722" stroke="#000000" stroke-width="1.5" />
          <line x1={vbrain.x + 25} y1="130" x2={vbrain.x + i * 18} y2="150" 
                stroke="#000000" stroke-width="1" />
        {/each}
      </g>
    {/each}
    
    <!-- Data chain -->
    <path d="M 50 117 L 95 117" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="95" cy="117" r="3" fill="#000000" />
    <path d="M 120 117 L 205 117" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    <circle cx="205" cy="117" r="3" fill="#000000" />
    <path d="M 230 117 L 315 117" stroke="#000000" stroke-width="1.5" 
          fill="none" />
    
    <text x="350" y="125" class="note-text">... continuing through Group 2A (vB4-25), then Group 2B (vB26-50)</text>
  </svg>
</div>

<style>
  .diagram-container {
    background: white;
    padding: 24px;
    border: 2px solid #000000;
    margin-bottom: 24px;
  }

  .line-svg {
    width: 100%;
    height: auto;
    margin-bottom: 24px;
  }

  .vbrain-unit {
    cursor: pointer;
    transition: opacity 0.15s ease;
  }

  .vbrain-unit:hover {
    opacity: 0.7;
  }

  .vbrain-unit:focus {
    outline: 2px solid #ff5722;
    outline-offset: 2px;
  }

  .line-label {
    font-size: 13px;
    font-weight: 400;
    fill: #000000;
    letter-spacing: 0;
  }

  .group-label {
    font-size: 11px;
    font-weight: 400;
    fill: #666666;
    text-anchor: middle;
  }

  .bus-label {
    font-size: 9px;
    fill: #666666;
    font-weight: 400;
  }

  .vbrain-text {
    font-size: 10px;
    fill: #000000;
    text-anchor: middle;
    dominant-baseline: middle;
    font-weight: 400;
  }

  .dots-text {
    font-size: 11px;
    fill: #999999;
    font-weight: 400;
  }

  .passthrough-label {
    font-size: 9px;
    fill: #666666;
    text-anchor: middle;
    font-style: normal;
  }

  .note-text {
    font-size: 10px;
    fill: #666666;
    line-height: 1.4;
  }
</style>

