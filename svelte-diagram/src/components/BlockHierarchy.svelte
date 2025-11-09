<script>
  // Hierarchical block structure
  const blocks = [
    { label: "Blocks", type: "primary", size: "large" },
    { label: "within", type: "secondary", size: "large" },
    { label: "Blocks", type: "primary", size: "medium" },
    { label: "within", type: "secondary", size: "medium" },
    { label: "...", type: "primary", size: "small" }
  ];
</script>

<div class="hierarchy-container">
  <svg viewBox="0 0 800 600" xmlns="http://www.w3.org/2000/svg">
    <!-- Vertical line -->
    <line x1="80" y1="80" x2="80" y2="520" stroke="#000000" stroke-width="3" />
    
    <!-- Block 1: Large orange "Blocks" -->
    <g class="block-item">
      <!-- Connection dot -->
      <circle cx="80" cy="100" r="12" fill="#000000" />
      <!-- Horizontal line -->
      <line x1="80" y1="100" x2="220" y2="100" stroke="#000000" stroke-width="3" />
      
      <!-- 3D Block -->
      <!-- Back face -->
      <polygon points="220,70 700,70 730,50 250,50" fill="#cc6633" stroke="#000000" stroke-width="2" />
      <!-- Top face -->
      <polygon points="700,70 730,50 730,130 700,150" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Front face -->
      <rect x="220" y="70" width="480" height="80" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Text -->
      <text x="460" y="120" font-family="ui-monospace, 'Courier New', monospace" font-size="48" font-weight="700" text-anchor="middle" fill="#000000">Blocks</text>
    </g>
    
    <!-- Block 2: Large white "within" -->
    <g class="block-item">
      <!-- Connection dot -->
      <circle cx="80" cy="200" r="12" fill="#000000" />
      <!-- Horizontal line -->
      <line x1="80" y1="200" x2="260" y2="200" stroke="#000000" stroke-width="3" />
      
      <!-- 3D Block -->
      <!-- Back face -->
      <polygon points="260,170 690,170 720,150 290,150" fill="#e0e0e0" stroke="#000000" stroke-width="2" />
      <!-- Top face -->
      <polygon points="690,170 720,150 720,230 690,250" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <!-- Front face -->
      <rect x="260" y="170" width="430" height="80" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <!-- Text -->
      <text x="475" y="220" font-family="ui-monospace, 'Courier New', monospace" font-size="48" font-weight="700" text-anchor="middle" fill="#000000">within</text>
    </g>
    
    <!-- Block 3: Medium orange "Blocks" -->
    <g class="block-item">
      <!-- Connection dot -->
      <circle cx="80" cy="300" r="12" fill="#000000" />
      <!-- Horizontal line -->
      <line x1="80" y1="300" x2="300" y2="300" stroke="#000000" stroke-width="3" />
      
      <!-- 3D Block -->
      <!-- Back face -->
      <polygon points="300,270 670,270 695,255 325,255" fill="#cc6633" stroke="#000000" stroke-width="2" />
      <!-- Top face -->
      <polygon points="670,270 695,255 695,330 670,345" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Front face -->
      <rect x="300" y="270" width="370" height="75" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Text -->
      <text x="485" y="318" font-family="ui-monospace, 'Courier New', monospace" font-size="42" font-weight="700" text-anchor="middle" fill="#000000">Blocks</text>
    </g>
    
    <!-- Block 4: Medium white "within" -->
    <g class="block-item">
      <!-- Connection dot -->
      <circle cx="80" cy="400" r="12" fill="#000000" />
      <!-- Horizontal line -->
      <line x1="80" y1="400" x2="340" y2="400" stroke="#000000" stroke-width="3" />
      
      <!-- 3D Block -->
      <!-- Back face -->
      <polygon points="340,370 640,370 665,355 365,355" fill="#e0e0e0" stroke="#000000" stroke-width="2" />
      <!-- Top face -->
      <polygon points="640,370 665,355 665,430 640,445" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <!-- Front face -->
      <rect x="340" y="370" width="300" height="75" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <!-- Text -->
      <text x="490" y="418" font-family="ui-monospace, 'Courier New', monospace" font-size="42" font-weight="700" text-anchor="middle" fill="#000000">within</text>
    </g>
    
    <!-- Block 5: Small orange "..." -->
    <g class="block-item">
      <!-- Connection dot -->
      <circle cx="80" cy="500" r="12" fill="#000000" />
      <!-- Horizontal line -->
      <line x1="80" y1="500" x2="380" y2="500" stroke="#000000" stroke-width="3" />
      
      <!-- 3D Block -->
      <!-- Back face -->
      <polygon points="380,470 580,470 600,458 400,458" fill="#cc6633" stroke="#000000" stroke-width="2" />
      <!-- Top face -->
      <polygon points="580,470 600,458 600,520 580,532" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Front face -->
      <rect x="380" y="470" width="200" height="62" fill="#ff5722" stroke="#000000" stroke-width="2" />
      <!-- Text -->
      <text x="480" y="512" font-family="ui-monospace, 'Courier New', monospace" font-size="36" font-weight="700" text-anchor="middle" fill="#000000">. . .</text>
    </g>
  </svg>
</div>

<style>
  .hierarchy-container {
    width: 100%;
    max-width: 900px;
    margin: 0 auto;
    padding: 24px;
    background: #ffffff;
    border: 2px solid #000000;
  }

  svg {
    width: 100%;
    height: auto;
  }

  .block-item {
    transition: opacity 0.2s ease;
  }

  .block-item:hover {
    opacity: 0.85;
  }
</style>

