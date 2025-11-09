<script>
  // Grid-based diagram component
</script>

<div class="grid-container">
  <svg viewBox="0 0 800 600" xmlns="http://www.w3.org/2000/svg">
    <!-- Main grid structure -->
    <!-- Outer border -->
    <rect x="50" y="50" width="700" height="500" fill="none" stroke="#000000" stroke-width="3" />
    
    <!-- Vertical divider (left column separator) -->
    <line x1="425" y1="50" x2="425" y2="550" stroke="#000000" stroke-width="3" />
    
    <!-- Horizontal divider (top row separator) -->
    <line x1="50" y1="300" x2="750" y2="300" stroke="#000000" stroke-width="3" />
    
    <!-- Middle vertical divider (bottom left section) -->
    <line x1="237.5" y1="300" x2="237.5" y2="550" stroke="#000000" stroke-width="3" />
    
    <!-- Bottom right section horizontal dividers -->
    <line x1="425" y1="383" x2="750" y2="383" stroke="#000000" stroke-width="3" />
    <line x1="425" y1="466" x2="750" y2="466" stroke="#000000" stroke-width="3" />
    
    <!-- TOP LEFT: Nested squares with asterisk -->
    <g class="nested-squares">
      <!-- Outer square -->
      <rect x="85" y="85" width="305" height="180" fill="none" stroke="#000000" stroke-width="2" />
      <!-- Middle square -->
      <rect x="115" y="115" width="245" height="120" fill="none" stroke="#000000" stroke-width="2" />
      <!-- Inner square -->
      <rect x="145" y="145" width="185" height="60" fill="none" stroke="#000000" stroke-width="2" />
      
      <!-- Asterisk/flower symbol -->
      <g transform="translate(237.5, 175)">
        <!-- 8 petals in a circle -->
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(0)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(45)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(90)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(135)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(180)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(225)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(270)" />
        <ellipse cx="0" cy="-18" rx="12" ry="20" fill="#ff5722" transform="rotate(315)" />
        <!-- Center circle -->
        <circle cx="0" cy="0" r="8" fill="#ff5722" />
      </g>
    </g>
    
    <!-- TOP RIGHT: Tree structure with dots -->
    <g class="tree-structure">
      <!-- Vertical line -->
      <line x1="490" y1="85" x2="490" y2="265" stroke="#000000" stroke-width="2" />
      
      <!-- Branch 1 (top) -->
      <line x1="490" y1="100" x2="540" y2="100" stroke="#000000" stroke-width="2" />
      <circle cx="540" cy="100" r="8" fill="#000000" />
      
      <!-- Branch 2 -->
      <line x1="490" y1="145" x2="540" y2="145" stroke="#000000" stroke-width="2" />
      <circle cx="540" cy="145" r="8" fill="#000000" />
      
      <!-- Branch 3 -->
      <line x1="490" y1="190" x2="540" y2="190" stroke="#000000" stroke-width="2" />
      <circle cx="540" cy="190" r="8" fill="#000000" />
      
      <!-- Branch 4 -->
      <line x1="490" y1="235" x2="540" y2="235" stroke="#000000" stroke-width="2" />
      <circle cx="540" cy="235" r="8" fill="#000000" />
      
      <!-- Branch 5 (bottom) -->
      <line x1="490" y1="265" x2="540" y2="265" stroke="#000000" stroke-width="2" />
      <circle cx="540" cy="265" r="8" fill="#000000" />
    </g>
    
    <!-- BOTTOM LEFT: Circle with play button -->
    <g class="play-button">
      <circle cx="143.75" cy="425" r="65" fill="none" stroke="#000000" stroke-width="2" />
      <!-- Play triangle -->
      <polygon points="125,400 125,450 165,425" fill="#ff5722" stroke="none" />
    </g>
    
    <!-- BOTTOM MIDDLE: Empty box (already created by grid) -->
    
    <!-- BOTTOM RIGHT: Three rows with labels -->
    <g class="label-rows">
      <!-- Row 1: "one" with orange accent -->
      <text x="450" y="355" font-family="ui-monospace, 'Courier New', monospace" font-size="38" font-weight="400" fill="#000000">one</text>
      <text x="490" y="355" font-family="ui-monospace, 'Courier New', monospace" font-size="38" font-weight="400" fill="#ff5722">I</text>
      
      <!-- Row 2: "two" -->
      <text x="450" y="438" font-family="ui-monospace, 'Courier New', monospace" font-size="38" font-weight="400" fill="#000000">two</text>
      
      <!-- Row 3: "three" -->
      <text x="450" y="521" font-family="ui-monospace, 'Courier New', monospace" font-size="38" font-weight="400" fill="#000000">three</text>
    </g>
  </svg>
</div>

<style>
  .grid-container {
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

  .nested-squares,
  .tree-structure,
  .play-button,
  .label-rows {
    transition: opacity 0.2s ease;
  }

  .nested-squares:hover,
  .tree-structure:hover,
  .play-button:hover {
    opacity: 0.85;
  }
</style>

