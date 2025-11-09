<script>
  // Detail view showing the topology
</script>

<div class="detail-container">
  <h2>Connection Topology Detail</h2>
  
  <svg viewBox="0 0 1200 300">
    <!-- AC Power Bus -->
    <line x1="80" y1="80" x2="1100" y2="80" stroke="#000000" stroke-width="3" />
    <circle cx="80" cy="80" r="4" fill="#000000" />
    <text x="80" y="65" class="label-bold">AC Power Bus (continuous)</text>
    
    <!-- vBrain 1 -->
    <g>
      <line x1="240" y1="80" x2="240" y2="120" stroke="#000000" stroke-width="2" />
      <circle cx="240" cy="80" r="4" fill="#000000" />
      <rect x="200" y="120" width="80" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="240" y="140" class="vbrain-label">vBrain 1</text>
      <path d="M 140 137 L 200 137" stroke="#000000" stroke-width="1.5" 
            fill="none" />
      <circle cx="140" cy="137" r="3" fill="#000000" />
      <text x="145" y="130" class="data-label">Data IN</text>
      
      <!-- Lights -->
      <rect x="195" y="175" width="22" height="15" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="220" y="175" width="22" height="15" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="245" y="175" width="22" height="15" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="270" y="175" width="22" height="15" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <line x1="240" y1="155" x2="206" y2="175" stroke="#000000" stroke-width="1" />
      <line x1="240" y1="155" x2="231" y2="175" stroke="#000000" stroke-width="1" />
      <line x1="240" y1="155" x2="256" y2="175" stroke="#000000" stroke-width="1" />
      <line x1="240" y1="155" x2="281" y2="175" stroke="#000000" stroke-width="1" />
      <text x="240" y="205" class="small-label">4 lights per vBrain</text>
    </g>
    
    <!-- vBrain 2 -->
    <g>
      <line x1="420" y1="80" x2="420" y2="120" stroke="#000000" stroke-width="2" />
      <circle cx="420" cy="80" r="4" fill="#000000" />
      <rect x="380" y="120" width="80" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="420" y="140" class="vbrain-label">vBrain 2</text>
      <path d="M 280 137 L 380 137" stroke="#000000" stroke-width="1.5" 
            fill="none" />
      <circle cx="280" cy="137" r="3" fill="#000000" />
      <text x="320" y="130" class="data-label">Data</text>
    </g>
    
    <!-- vBrain 3 -->
    <g>
      <line x1="600" y1="80" x2="600" y2="120" stroke="#000000" stroke-width="2" />
      <circle cx="600" cy="80" r="4" fill="#000000" />
      <rect x="560" y="120" width="80" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="600" y="140" class="vbrain-label">vBrain 3</text>
      <path d="M 460 137 L 560 137" stroke="#000000" stroke-width="1.5" 
            fill="none" />
      <circle cx="460" cy="137" r="3" fill="#000000" />
      <text x="500" y="130" class="data-label">Data</text>
    </g>
    
    <text x="700" y="137" class="dots">...</text>
    
    <!-- Explanation -->
    <text x="80" y="250" class="explanation">
      <tspan x="80" dy="0">• AC POWER: Single continuous bus with T-branch to each vBrain</tspan>
      <tspan x="80" dy="20">• DATA: Daisy-chained (exits one, enters next) through all vBrains</tspan>
      <tspan x="80" dy="20">• Each line has 50 vBrains (Groups A+B), each controlling 4 lights</tspan>
    </text>
  </svg>
</div>

<style>
  .detail-container {
    background: white;
    padding: 32px;
    border: 2px solid #000000;
    margin-bottom: 24px;
  }

  h2 {
    text-align: center;
    font-size: 20px;
    font-weight: 400;
    margin-bottom: 24px;
    color: #000000;
    letter-spacing: 0;
  }

  svg {
    width: 100%;
    height: auto;
  }

  .label-bold {
    font-size: 12px;
    font-weight: 400;
    fill: #000000;
  }

  .vbrain-label {
    font-size: 11px;
    fill: #000000;
    text-anchor: middle;
    dominant-baseline: middle;
    font-weight: 400;
  }

  .data-label {
    font-size: 10px;
    fill: #666666;
    font-weight: 400;
  }

  .small-label {
    font-size: 10px;
    fill: #666666;
    text-anchor: middle;
  }

  .dots {
    font-size: 20px;
    fill: #000000;
    font-weight: 400;
  }

  .explanation {
    font-size: 11px;
    fill: #333333;
    line-height: 1.5;
  }
</style>
