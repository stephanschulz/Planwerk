<script>
  // Voice Corridor System Architecture with clean black/orange aesthetic
</script>

<div class="corridor-container">
  <h2>Voice Corridor - System Architecture</h2>
  <p class="subtitle">2 lines × 200 lights × 50 vBrains per line (25 vBrains per group)</p>
  
  <svg viewBox="0 0 1400 900" xmlns="http://www.w3.org/2000/svg">
    <!-- LINE 1 - Group A Container -->
    <rect x="50" y="50" width="650" height="220" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="65" y="75" font-family="ui-monospace, 'Courier New', monospace" font-size="13" font-weight="400" fill="#000000">LINE 1 - Group A (showing 4 of 25 vBrains)</text>
    
    <!-- AC Power Bus -->
    <line x1="70" y1="110" x2="600" y2="110" stroke="#000000" stroke-width="2" />
    <circle cx="70" cy="110" r="4" fill="#000000" />
    <text x="70" y="100" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">AC Power</text>
    
    <!-- vBrain 1 with 3D effect -->
    <g class="vbrain-unit">
      <!-- Shadow -->
      <rect x="88" y="143" width="70" height="35" fill="#cccccc" stroke="none" />
      <!-- Main box -->
      <rect x="85" y="140" width="70" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="120" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" text-anchor="middle" fill="#000000">vBrain 1</text>
      
      <!-- Power T-branch -->
      <line x1="120" y1="110" x2="120" y2="140" stroke="#000000" stroke-width="2" />
      <circle cx="120" cy="110" r="4" fill="#000000" />
      
      <!-- Lights -->
      <rect x="73" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="98" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="123" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="148" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      
      <!-- Connections to lights -->
      <line x1="120" y1="175" x2="84" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="120" y1="175" x2="109" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="120" y1="175" x2="134" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="120" y1="175" x2="159" y2="193" stroke="#000000" stroke-width="1" />
    </g>
    
    <!-- Data connection from vBrain 1 -->
    <line x1="155" y1="157" x2="220" y2="157" stroke="#000000" stroke-width="1.5" />
    <circle cx="220" cy="157" r="3" fill="#000000" />
    <text x="178" y="152" font-family="ui-monospace, 'Courier New', monospace" font-size="8" font-weight="400" fill="#666666">Data</text>
    
    <!-- vBrain 2 -->
    <g class="vbrain-unit">
      <rect x="233" y="143" width="70" height="35" fill="#cccccc" stroke="none" />
      <rect x="230" y="140" width="70" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="265" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" text-anchor="middle" fill="#000000">vBrain 2</text>
      
      <line x1="265" y1="110" x2="265" y2="140" stroke="#000000" stroke-width="2" />
      <circle cx="265" cy="110" r="4" fill="#000000" />
      
      <rect x="218" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="243" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="268" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="293" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      
      <line x1="265" y1="175" x2="229" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="265" y1="175" x2="254" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="265" y1="175" x2="279" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="265" y1="175" x2="304" y2="193" stroke="#000000" stroke-width="1" />
    </g>
    
    <line x1="300" y1="157" x2="365" y2="157" stroke="#000000" stroke-width="1.5" />
    <circle cx="365" cy="157" r="3" fill="#000000" />
    
    <!-- vBrain 3 -->
    <g class="vbrain-unit">
      <rect x="378" y="143" width="70" height="35" fill="#cccccc" stroke="none" />
      <rect x="375" y="140" width="70" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="410" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" text-anchor="middle" fill="#000000">vBrain 3</text>
      
      <line x1="410" y1="110" x2="410" y2="140" stroke="#000000" stroke-width="2" />
      <circle cx="410" cy="110" r="4" fill="#000000" />
      
      <rect x="363" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="388" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="413" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="438" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      
      <line x1="410" y1="175" x2="374" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="410" y1="175" x2="399" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="410" y1="175" x2="424" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="410" y1="175" x2="449" y2="193" stroke="#000000" stroke-width="1" />
    </g>
    
    <line x1="445" y1="157" x2="485" y2="157" stroke="#000000" stroke-width="1.5" />
    <text x="490" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="16" font-weight="400" fill="#000000">...</text>
    <line x1="515" y1="157" x2="555" y2="157" stroke="#000000" stroke-width="1.5" />
    
    <!-- vBrain 25 -->
    <g class="vbrain-unit">
      <rect x="568" y="143" width="70" height="35" fill="#cccccc" stroke="none" />
      <rect x="565" y="140" width="70" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="600" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" text-anchor="middle" fill="#000000">vBrain 25</text>
      
      <line x1="600" y1="110" x2="600" y2="140" stroke="#000000" stroke-width="2" />
      <circle cx="600" cy="110" r="4" fill="#000000" />
      
      <rect x="553" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="578" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="603" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="628" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      
      <line x1="600" y1="175" x2="564" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="600" y1="175" x2="589" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="600" y1="175" x2="614" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="600" y1="175" x2="639" y2="193" stroke="#000000" stroke-width="1" />
    </g>
    <circle cx="555" cy="157" r="3" fill="#000000" />
    
    <text x="70" y="255" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">• AC power bus with T-branches  • Data daisy chain (all 25 vBrains)</text>
    
    <!-- LINE 1 - Group B Container -->
    <rect x="730" y="50" width="620" height="220" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="745" y="75" font-family="ui-monospace, 'Courier New', monospace" font-size="13" font-weight="400" fill="#000000">LINE 1 - Group B (showing 4 of 25 vBrains)</text>
    <text x="745" y="95" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">← Pass-through from Group A</text>
    
    <!-- AC Power Bus continuing -->
    <line x1="730" y1="110" x2="1250" y2="110" stroke="#000000" stroke-width="2" />
    
    <!-- vBrain 26 -->
    <g class="vbrain-unit">
      <rect x="768" y="143" width="70" height="35" fill="#cccccc" stroke="none" />
      <rect x="765" y="140" width="70" height="35" fill="#ffffff" stroke="#000000" stroke-width="2" />
      <text x="800" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" text-anchor="middle" fill="#000000">vBrain 26</text>
      
      <line x1="800" y1="110" x2="800" y2="140" stroke="#000000" stroke-width="2" />
      <circle cx="800" cy="110" r="4" fill="#000000" />
      
      <rect x="753" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="778" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="803" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      <rect x="828" y="193" width="22" height="16" fill="#ff5722" stroke="#000000" stroke-width="1.5" />
      
      <line x1="800" y1="175" x2="764" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="800" y1="175" x2="789" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="800" y1="175" x2="814" y2="193" stroke="#000000" stroke-width="1" />
      <line x1="800" y1="175" x2="839" y2="193" stroke="#000000" stroke-width="1" />
    </g>
    
    <!-- Data from Group A -->
    <line x1="700" y1="157" x2="765" y2="157" stroke="#000000" stroke-width="1.5" />
    <circle cx="700" cy="157" r="3" fill="#000000" />
    
    <line x1="835" y1="157" x2="895" y2="157" stroke="#000000" stroke-width="1.5" />
    <circle cx="895" cy="157" r="3" fill="#000000" />
    
    <text x="920" y="162" font-family="ui-monospace, 'Courier New', monospace" font-size="14" font-weight="400" fill="#000000">... (continuing to vBrain 50)</text>
    
    <text x="745" y="255" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">• Data continues from Group A through all 25 vBrains in Group B</text>
    
    <!-- DETAIL: Single vBrain Unit -->
    <rect x="50" y="300" width="700" height="330" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="365" y="330" font-family="ui-monospace, 'Courier New', monospace" font-size="14" font-weight="400" text-anchor="middle" fill="#000000">DETAIL: Single vBrain Unit Connection</text>
    
    <!-- Detail AC Power Bus -->
    <line x1="100" y1="380" x2="700" y2="380" stroke="#000000" stroke-width="3" />
    <circle cx="100" cy="380" r="5" fill="#000000" />
    <text x="100" y="370" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" fill="#000000">AC Power Bus (continuous)</text>
    
    <!-- Detail vBrain with 3D effect -->
    <rect x="343" y="443" width="130" height="55" fill="#cccccc" stroke="none" />
    <rect x="340" y="440" width="130" height="55" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="405" y="473" font-family="ui-monospace, 'Courier New', monospace" font-size="13" font-weight="400" text-anchor="middle" fill="#000000">vBrain Controller</text>
    
    <!-- Power T-branch -->
    <line x1="405" y1="380" x2="405" y2="440" stroke="#000000" stroke-width="3" />
    <circle cx="405" cy="380" r="5" fill="#000000" />
    
    <!-- Data IN -->
    <line x1="200" y1="467" x2="340" y2="467" stroke="#000000" stroke-width="2" />
    <circle cx="200" cy="467" r="4" fill="#000000" />
    <text x="230" y="460" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">Data IN</text>
    
    <!-- Data OUT -->
    <line x1="470" y1="467" x2="610" y2="467" stroke="#000000" stroke-width="2" />
    <circle cx="610" cy="467" r="4" fill="#000000" />
    <text x="520" y="460" font-family="ui-monospace, 'Courier New', monospace" font-size="9" font-weight="400" fill="#666666">Data OUT</text>
    
    <!-- Detail Lights with 3D effect -->
    <rect x="218" y="553" width="90" height="30" fill="#cc6633" stroke="none" />
    <rect x="215" y="550" width="90" height="30" fill="#ff5722" stroke="#000000" stroke-width="2" />
    <text x="260" y="570" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" text-anchor="middle" fill="#ffffff">Light 1</text>
    
    <rect x="328" y="553" width="90" height="30" fill="#cc6633" stroke="none" />
    <rect x="325" y="550" width="90" height="30" fill="#ff5722" stroke="#000000" stroke-width="2" />
    <text x="370" y="570" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" text-anchor="middle" fill="#ffffff">Light 2</text>
    
    <rect x="438" y="553" width="90" height="30" fill="#cc6633" stroke="none" />
    <rect x="435" y="550" width="90" height="30" fill="#ff5722" stroke="#000000" stroke-width="2" />
    <text x="480" y="570" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" text-anchor="middle" fill="#ffffff">Light 3</text>
    
    <rect x="548" y="553" width="90" height="30" fill="#cc6633" stroke="none" />
    <rect x="545" y="550" width="90" height="30" fill="#ff5722" stroke="#000000" stroke-width="2" />
    <text x="590" y="570" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" text-anchor="middle" fill="#ffffff">Light 4</text>
    
    <!-- Connections to lights -->
    <line x1="405" y1="495" x2="260" y2="550" stroke="#000000" stroke-width="1.5" />
    <line x1="405" y1="495" x2="370" y2="550" stroke="#000000" stroke-width="1.5" />
    <line x1="405" y1="495" x2="480" y2="550" stroke="#000000" stroke-width="1.5" />
    <line x1="405" y1="495" x2="590" y2="550" stroke="#000000" stroke-width="1.5" />
    
    <!-- System Summary Box with 3D effect -->
    <rect x="783" y="303" width="550" height="180" fill="#cccccc" stroke="none" />
    <rect x="780" y="300" width="550" height="180" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="800" y="330" font-family="ui-monospace, 'Courier New', monospace" font-size="14" font-weight="400" fill="#000000">System Summary</text>
    
    <text x="800" y="360" font-family="ui-monospace, 'Courier New', monospace" font-size="11" font-weight="400" fill="#333333">
      <tspan x="800" dy="0">• 400 total lights (2 lines × 200 each)</tspan>
      <tspan x="800" dy="20">• 100 vBrain controllers (2 lines × 50 each)</tspan>
      <tspan x="800" dy="20">• 4 groups of 25 vBrains (2 groups per line)</tspan>
      <tspan x="800" dy="20">• AC Power: Continuous bus with T-branches</tspan>
      <tspan x="800" dy="20">• Data: Daisy-chained through all units</tspan>
      <tspan x="800" dy="20">• Each vBrain controls 4 rectangular lights</tspan>
    </text>
    
    <!-- Legend Box with 3D effect -->
    <rect x="783" y="513" width="550" height="117" fill="#cccccc" stroke="none" />
    <rect x="780" y="510" width="550" height="117" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="800" y="540" font-family="ui-monospace, 'Courier New', monospace" font-size="14" font-weight="400" fill="#000000">Legend</text>
    
    <!-- Legend items -->
    <rect x="803" y="563" width="35" height="22" fill="#cccccc" stroke="none" />
    <rect x="800" y="560" width="35" height="22" fill="#ffffff" stroke="#000000" stroke-width="2" />
    <text x="845" y="575" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" fill="#000000">vBrain Controller</text>
    
    <rect x="1003" y="563" width="35" height="22" fill="#cc6633" stroke="none" />
    <rect x="1000" y="560" width="35" height="22" fill="#ff5722" stroke="#000000" stroke-width="2" />
    <text x="1045" y="575" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" fill="#000000">Rectangular Light</text>
    
    <line x1="800" y1="600" x2="835" y2="600" stroke="#000000" stroke-width="2" />
    <circle cx="800" cy="600" r="4" fill="#000000" />
    <text x="845" y="605" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" fill="#000000">AC Power Bus</text>
    
    <line x1="1000" y1="600" x2="1035" y2="600" stroke="#000000" stroke-width="1.5" />
    <circle cx="1035" cy="600" r="3" fill="#000000" />
    <text x="1045" y="605" font-family="ui-monospace, 'Courier New', monospace" font-size="10" font-weight="400" fill="#000000">Data Daisy Chain</text>
  </svg>
</div>

<style>
  .corridor-container {
    width: 100%;
    max-width: 1500px;
    margin: 0 auto;
    padding: 24px;
    background: #ffffff;
    border: 2px solid #000000;
  }

  h2 {
    text-align: center;
    font-size: 22px;
    font-weight: 400;
    margin-bottom: 8px;
    color: #000000;
    font-family: ui-monospace, 'Courier New', monospace;
  }

  .subtitle {
    text-align: center;
    font-size: 12px;
    color: #666666;
    margin-bottom: 24px;
    font-family: ui-monospace, 'Courier New', monospace;
  }

  svg {
    width: 100%;
    height: auto;
  }

  .vbrain-unit {
    transition: opacity 0.2s ease;
  }

  .vbrain-unit:hover {
    opacity: 0.85;
  }
</style>


