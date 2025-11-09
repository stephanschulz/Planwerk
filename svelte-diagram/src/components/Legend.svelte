<div class="legend-container">
  <h3>Legend</h3>
  
  <svg viewBox="0 0 1200 80">
    <!-- vBrain -->
    <rect x="50" y="20" width="30" height="20" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="90" y="32" class="legend-text">vBrain Controller</text>
    
    <!-- Light -->
    <rect x="230" y="20" width="30" height="20" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
    <text x="270" y="32" class="legend-text">Rectangular Light</text>
    
    <!-- AC Power Bus -->
    <line x1="420" y1="30" x2="470" y2="30" stroke="#000000" stroke-width="2" />
    <circle cx="420" cy="30" r="3" fill="#000000" />
    <text x="480" y="32" class="legend-text">AC Power Bus</text>
    
    <!-- Power T-branch -->
    <g>
      <line x1="620" y1="20" x2="620" y2="40" stroke="#000000" stroke-width="2" />
      <circle cx="620" cy="20" r="3" fill="#000000" />
      <text x="635" y="32" class="legend-text">Power T-branch</text>
    </g>
    
    <!-- Data -->
    <line x1="780" y1="30" x2="830" y2="30" stroke="#000000" stroke-width="1.5" />
    <circle cx="830" cy="30" r="3" fill="#000000" />
    <text x="840" y="32" class="legend-text">Data Daisy Chain</text>
    
    <!-- Control -->
    <line x1="1000" y1="30" x2="1040" y2="30" stroke="#000000" stroke-width="1" />
    <text x="1050" y="32" class="legend-text">Control to Lights</text>
  </svg>
  
  <div class="summary">
    <h4>System Summary</h4>
    <ul>
      <li><strong>400 total lights</strong> (2 lines × 200 lights each)</li>
      <li><strong>100 vBrain controllers</strong> (2 lines × 50 vBrains each)</li>
      <li><strong>4 groups</strong> of 25 vBrains (2 groups per line)</li>
      <li>All vBrains and lights <strong>evenly spaced</strong></li>
      <li>Power: <strong>Continuous bus with T-branches</strong></li>
      <li>Data: <strong>Daisy-chained</strong> through all units in each line</li>
    </ul>
  </div>
</div>

<style>
  .legend-container {
    background: white;
    padding: 24px;
    border: 2px solid #000000;
  }

  h3 {
    font-size: 20px;
    font-weight: 400;
    margin-bottom: 20px;
    color: #000000;
    letter-spacing: 0;
  }

  svg {
    width: 100%;
    height: auto;
    margin-bottom: 20px;
  }

  .legend-text {
    font-size: 11px;
    fill: #000000;
    dominant-baseline: middle;
    font-weight: 400;
  }

  .summary {
    margin-top: 24px;
    padding: 20px;
    background: #ffffff;
    border: 1px solid #000000;
  }

  h4 {
    font-size: 16px;
    font-weight: 400;
    margin-bottom: 12px;
    color: #000000;
  }

  ul {
    list-style: none;
    padding: 0;
  }

  li {
    font-size: 12px;
    color: #333333;
    margin: 8px 0;
    padding-left: 16px;
    position: relative;
    line-height: 1.5;
  }

  li::before {
    content: "•";
    position: absolute;
    left: 0;
    color: #000000;
    font-weight: normal;
    font-size: 14px;
  }

  strong {
    color: #000000;
    font-weight: 600;
  }
</style>
