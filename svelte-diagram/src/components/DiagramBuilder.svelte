<script>
  import { onMount } from 'svelte';
  
  let elements = $state([]);
  let selectedElement = $state(null);
  let tool = $state('select'); // 'select', 'box', 'line', 'text'
  let dragStart = $state(null);
  let isDragging = $state(false);
  let isResizing = $state(false);
  let resizeHandle = $state(null); // 'nw', 'ne', 'sw', 'se', 'start', 'end'
  let canvasElement = $state(null);
  let editingText = $state(null);
  let editingLineLabel = $state(false);
  let savedDesigns = $state([]);
  let currentDesignName = $state('Untitled');
  
  // Template for new elements
  function createBox(x, y) {
    return {
      id: Date.now(),
      type: 'box',
      x: x - 60,
      y: y - 25,
      width: 120,
      height: 50,
      text: 'New Box',
      shadow: true,
      dashed: false
    };
  }
  
  function createText(x, y) {
    return {
      id: Date.now(),
      type: 'text',
      x: x - 50,
      y: y - 10,
      text: 'Text Label',
      fontSize: 14
    };
  }
  
  function createLine(x1, y1, x2, y2) {
    return {
      id: Date.now(),
      type: 'line',
      x1,
      y1,
      x2: x2 || x1 + 100,
      y2: y2 || y1,
      hasCircle: false,
      hasArrow: false,
      label: ''
    };
  }
  
  function handleCanvasClick(e) {
    if (tool === 'select') {
      selectedElement = null;
      editingText = null;
      editingLineLabel = false;
      return;
    }
    
    const rect = canvasElement.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    if (tool === 'box') {
      elements = [...elements, createBox(x, y)];
    } else if (tool === 'text') {
      elements = [...elements, createText(x, y)];
    } else if (tool === 'line') {
      elements = [...elements, createLine(x, y)];
    }
    
    tool = 'select';
  }
  
  function handleElementClick(e, element) {
    e.stopPropagation();
    selectedElement = element;
  }
  
  function handleDoubleClick(e, element) {
    e.stopPropagation();
    e.preventDefault();
    
    if (element.type === 'line') {
      // For lines, enable label editing
      if (element.label === undefined) {
        element.label = '';
      }
      selectedElement = element;
      editingText = element;
      editingLineLabel = true;
      
      // Force a re-render and then focus
      setTimeout(() => {
        const textInput = document.getElementById(`edit-line-label-${element.id}`);
        if (textInput) {
          textInput.focus();
          if (element.label) {
            textInput.select();
          }
        }
      }, 50);
    } else if (element.text !== undefined) {
      editingText = element;
      editingLineLabel = false;
      // Focus will be handled by the foreignObject
      setTimeout(() => {
        const textInput = document.getElementById(`edit-${element.id}`);
        if (textInput) {
          textInput.focus();
          textInput.select();
        }
      }, 50);
    }
  }
  
  // Helper function to find snap points near a coordinate
  function findSnapPoint(x, y, excludeElementId) {
    const snapDistance = 10; // pixels
    
    for (const el of elements) {
      if (el.id === excludeElementId) continue;
      
      if (el.type === 'box') {
        // Check box edges
        const edges = [
          { x: el.x, y: el.y }, // top-left
          { x: el.x + el.width, y: el.y }, // top-right
          { x: el.x, y: el.y + el.height }, // bottom-left
          { x: el.x + el.width, y: el.y + el.height }, // bottom-right
          { x: el.x + el.width / 2, y: el.y }, // top-center
          { x: el.x + el.width / 2, y: el.y + el.height }, // bottom-center
          { x: el.x, y: el.y + el.height / 2 }, // left-center
          { x: el.x + el.width, y: el.y + el.height / 2 }, // right-center
        ];
        
        for (const edge of edges) {
          const dist = Math.sqrt(Math.pow(x - edge.x, 2) + Math.pow(y - edge.y, 2));
          if (dist < snapDistance) {
            return edge;
          }
        }
      } else if (el.type === 'line') {
        // Check line endpoints
        const points = [
          { x: el.x1, y: el.y1 },
          { x: el.x2, y: el.y2 }
        ];
        
        for (const point of points) {
          const dist = Math.sqrt(Math.pow(x - point.x, 2) + Math.pow(y - point.y, 2));
          if (dist < snapDistance) {
            return point;
          }
        }
      }
    }
    
    return null;
  }
  
  function finishEditingText() {
    editingText = null;
    editingLineLabel = false;
  }
  
  function handleMouseDown(e, element) {
    // Don't start dragging if we're currently editing
    if (editingText || editingLineLabel) {
      return;
    }
    
    e.stopPropagation();
    selectedElement = element;
    isDragging = true;
    
    const rect = canvasElement.getBoundingClientRect();
    dragStart = {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
      elementX: element.x || element.x1,
      elementY: element.y || element.y1,
      elementWidth: element.width,
      elementHeight: element.height,
      elementX2: element.x2,
      elementY2: element.y2
    };
  }
  
  function handleResizeStart(e, element, handle) {
    e.stopPropagation();
    selectedElement = element;
    isResizing = true;
    resizeHandle = handle;
    
    const rect = canvasElement.getBoundingClientRect();
    dragStart = {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
      elementX: element.x || element.x1,
      elementY: element.y || element.y1,
      elementWidth: element.width,
      elementHeight: element.height,
      elementX2: element.x2,
      elementY2: element.y2
    };
  }
  
  function handleMouseMove(e) {
    if ((!isDragging && !isResizing) || !dragStart || !selectedElement) return;
    
    const rect = canvasElement.getBoundingClientRect();
    const currentX = e.clientX - rect.left;
    const currentY = e.clientY - rect.top;
    
    const dx = currentX - dragStart.x;
    const dy = currentY - dragStart.y;
    
    if (isResizing) {
      // Handle resizing
      if (selectedElement.type === 'box') {
        if (resizeHandle === 'se') {
          selectedElement.width = Math.max(50, dragStart.elementWidth + dx);
          selectedElement.height = Math.max(30, dragStart.elementHeight + dy);
        } else if (resizeHandle === 'sw') {
          const newWidth = Math.max(50, dragStart.elementWidth - dx);
          selectedElement.x = dragStart.elementX + (dragStart.elementWidth - newWidth);
          selectedElement.width = newWidth;
          selectedElement.height = Math.max(30, dragStart.elementHeight + dy);
        } else if (resizeHandle === 'ne') {
          selectedElement.width = Math.max(50, dragStart.elementWidth + dx);
          const newHeight = Math.max(30, dragStart.elementHeight - dy);
          selectedElement.y = dragStart.elementY + (dragStart.elementHeight - newHeight);
          selectedElement.height = newHeight;
        } else if (resizeHandle === 'nw') {
          const newWidth = Math.max(50, dragStart.elementWidth - dx);
          const newHeight = Math.max(30, dragStart.elementHeight - dy);
          selectedElement.x = dragStart.elementX + (dragStart.elementWidth - newWidth);
          selectedElement.y = dragStart.elementY + (dragStart.elementHeight - newHeight);
          selectedElement.width = newWidth;
          selectedElement.height = newHeight;
        }
      } else if (selectedElement.type === 'line') {
        if (resizeHandle === 'start') {
          // Check for snap points
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id);
          let finalX = snapPoint ? snapPoint.x : currentX;
          let finalY = snapPoint ? snapPoint.y : currentY;
          
          if (e.shiftKey) {
            // Snap to 15-degree intervals relative to the other end
            const angle = Math.atan2(finalY - selectedElement.y2, finalX - selectedElement.x2);
            const snapAngle = Math.round(angle / (Math.PI / 12)) * (Math.PI / 12); // 15 degrees = PI/12
            const distance = Math.sqrt(Math.pow(finalX - selectedElement.x2, 2) + Math.pow(finalY - selectedElement.y2, 2));
            selectedElement.x1 = selectedElement.x2 + distance * Math.cos(snapAngle);
            selectedElement.y1 = selectedElement.y2 + distance * Math.sin(snapAngle);
          } else {
            selectedElement.x1 = finalX;
            selectedElement.y1 = finalY;
          }
        } else if (resizeHandle === 'end') {
          // Check for snap points
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id);
          let finalX = snapPoint ? snapPoint.x : currentX;
          let finalY = snapPoint ? snapPoint.y : currentY;
          
          if (e.shiftKey) {
            // Snap to 15-degree intervals relative to the start
            const angle = Math.atan2(finalY - selectedElement.y1, finalX - selectedElement.x1);
            const snapAngle = Math.round(angle / (Math.PI / 12)) * (Math.PI / 12); // 15 degrees = PI/12
            const distance = Math.sqrt(Math.pow(finalX - selectedElement.x1, 2) + Math.pow(finalY - selectedElement.y1, 2));
            selectedElement.x2 = selectedElement.x1 + distance * Math.cos(snapAngle);
            selectedElement.y2 = selectedElement.y1 + distance * Math.sin(snapAngle);
          } else {
            selectedElement.x2 = finalX;
            selectedElement.y2 = finalY;
          }
        }
      }
    } else if (isDragging) {
      // Handle dragging
      if (selectedElement.type === 'line') {
        const width = dragStart.elementX2 - dragStart.elementX;
        const height = dragStart.elementY2 - dragStart.elementY;
        selectedElement.x1 = dragStart.elementX + dx;
        selectedElement.y1 = dragStart.elementY + dy;
        selectedElement.x2 = selectedElement.x1 + width;
        selectedElement.y2 = selectedElement.y1 + height;
      } else {
        selectedElement.x = dragStart.elementX + dx;
        selectedElement.y = dragStart.elementY + dy;
      }
    }
  }
  
  function handleMouseUp() {
    isDragging = false;
    isResizing = false;
    resizeHandle = null;
    dragStart = null;
  }
  
  function deleteSelected() {
    if (selectedElement) {
      elements = elements.filter(el => el.id !== selectedElement.id);
      selectedElement = null;
    }
  }
  
  function handleKeyDown(e) {
    // Don't process any keys if we're editing text
    if (editingText || editingLineLabel) {
      return;
    }
    
    // Delete/Backspace to remove element
    if ((e.key === 'Delete' || e.key === 'Backspace') && selectedElement) {
      e.preventDefault();
      deleteSelected();
      return;
    }
    
    // Arrow key nudging - works when element is selected and not editing
    if (selectedElement) {
      if (e.key === 'ArrowUp') {
        e.preventDefault();
        if (selectedElement.type === 'line') {
          selectedElement.y1 -= 1;
          selectedElement.y2 -= 1;
        } else {
          selectedElement.y -= 1;
        }
      } else if (e.key === 'ArrowDown') {
        e.preventDefault();
        if (selectedElement.type === 'line') {
          selectedElement.y1 += 1;
          selectedElement.y2 += 1;
        } else {
          selectedElement.y += 1;
        }
      } else if (e.key === 'ArrowLeft') {
        e.preventDefault();
        if (selectedElement.type === 'line') {
          selectedElement.x1 -= 1;
          selectedElement.x2 -= 1;
        } else {
          selectedElement.x -= 1;
        }
      } else if (e.key === 'ArrowRight') {
        e.preventDefault();
        if (selectedElement.type === 'line') {
          selectedElement.x1 += 1;
          selectedElement.x2 += 1;
        } else {
          selectedElement.x += 1;
        }
      }
    }
  }
  
  function updateText(element, newText) {
    element.text = newText;
  }
  
  function saveDiagram() {
    const name = prompt('Enter a name for this design:', currentDesignName);
    if (!name) return;
    
    currentDesignName = name;
    
    // Get existing designs
    const designs = JSON.parse(localStorage.getItem('savedDesigns') || '{}');
    designs[name] = {
      name,
      elements,
      savedAt: new Date().toISOString()
    };
    
    localStorage.setItem('savedDesigns', JSON.stringify(designs));
    loadSavedDesignsList();
    alert(`Design "${name}" saved!`);
  }
  
  function loadSavedDesignsList() {
    const designs = JSON.parse(localStorage.getItem('savedDesigns') || '{}');
    savedDesigns = Object.values(designs).sort((a, b) => 
      new Date(b.savedAt) - new Date(a.savedAt)
    );
  }
  
  function loadDesign(name) {
    const designs = JSON.parse(localStorage.getItem('savedDesigns') || '{}');
    if (designs[name]) {
      elements = designs[name].elements;
      currentDesignName = name;
      selectedElement = null;
      alert(`Design "${name}" loaded!`);
    }
  }
  
  function deleteDesign(name) {
    if (!confirm(`Delete design "${name}"?`)) return;
    
    const designs = JSON.parse(localStorage.getItem('savedDesigns') || '{}');
    delete designs[name];
    localStorage.setItem('savedDesigns', JSON.stringify(designs));
    loadSavedDesignsList();
    
    if (currentDesignName === name) {
      currentDesignName = 'Untitled';
      elements = [];
    }
  }
  
  function newDesign() {
    if (elements.length > 0 && !confirm('Start a new design? Unsaved changes will be lost.')) {
      return;
    }
    elements = [];
    currentDesignName = 'Untitled';
    selectedElement = null;
  }
  
  function exportDiagram() {
    const data = JSON.stringify(elements, null, 2);
    const blob = new Blob([data], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'diagram.json';
    a.click();
    URL.revokeObjectURL(url);
  }
  
  function importDiagram() {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = '.json';
    input.onchange = (e) => {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          try {
            elements = JSON.parse(event.target.result);
            selectedElement = null;
            alert('Diagram imported!');
          } catch (error) {
            alert('Error importing diagram: ' + error.message);
          }
        };
        reader.readAsText(file);
      }
    };
    input.click();
  }
  
  onMount(() => {
    window.addEventListener('keydown', handleKeyDown);
    loadSavedDesignsList();
    return () => window.removeEventListener('keydown', handleKeyDown);
  });
</script>

<svelte:window onmousemove={handleMouseMove} onmouseup={handleMouseUp} />

<div class="builder-container">
  <div class="toolbar">
    <div class="tool-section">
      <h3>Tools</h3>
      <button 
        class="tool-btn"
        class:active={tool === 'select'}
        onclick={() => tool = 'select'}
        title="Select (Esc)">
        <svg width="20" height="20" viewBox="0 0 20 20">
          <path d="M2 2 L2 18 L8 12 L12 18 L15 16 L11 10 L18 8 Z" fill="currentColor"/>
        </svg>
        Select
      </button>
      
      <button 
        class="tool-btn"
        class:active={tool === 'box'}
        onclick={() => tool = 'box'}
        title="Add Box">
        <svg width="20" height="20" viewBox="0 0 20 20">
          <rect x="2" y="4" width="16" height="12" fill="none" stroke="currentColor" stroke-width="2"/>
        </svg>
        Box
      </button>
      
      <button 
        class="tool-btn"
        class:active={tool === 'text'}
        onclick={() => tool = 'text'}
        title="Add Text">
        <svg width="20" height="20" viewBox="0 0 20 20">
          <text x="10" y="15" font-size="14" fill="currentColor" text-anchor="middle" font-weight="bold">T</text>
        </svg>
        Text
      </button>
      
      <button 
        class="tool-btn"
        class:active={tool === 'line'}
        onclick={() => tool = 'line'}
        title="Add Line">
        <svg width="20" height="20" viewBox="0 0 20 20">
          <line x1="2" y1="10" x2="18" y2="10" stroke="currentColor" stroke-width="2"/>
        </svg>
        Line
      </button>
    </div>
    
    <div class="tool-section">
      <h3>Actions</h3>
      <button 
        class="action-btn"
        onclick={deleteSelected}
        disabled={!selectedElement}
        title="Delete (Del)">
        Delete
      </button>
      
      <button 
        class="action-btn"
        onclick={() => elements = []}
        title="Clear Canvas">
        Clear All
      </button>
    </div>
    
    <div class="tool-section">
      <h3>File</h3>
      <label>
        Name:
        <input 
          type="text" 
          bind:value={currentDesignName}
          placeholder="Untitled"
          class="filename-input"
        />
      </label>
      
      <button 
        class="action-btn"
        onclick={newDesign}
        title="Start a new design">
        New
      </button>
      
      <button 
        class="action-btn"
        onclick={saveDiagram}
        title="Save current design">
        Save
      </button>
      
      {#if savedDesigns.length > 0}
        <label>
          Saved Designs:
          <select 
            class="design-select"
            onchange={(e) => loadDesign(e.target.value)}>
            <option value="">-- Select Design --</option>
            {#each savedDesigns as design}
              <option value={design.name}>{design.name}</option>
            {/each}
          </select>
        </label>
        
        <button 
          class="action-btn delete-btn"
          onclick={() => {
            const select = document.querySelector('.design-select');
            if (select.value) deleteDesign(select.value);
          }}
          title="Delete selected design">
          Delete Selected
        </button>
      {/if}
      
      <button 
        class="action-btn"
        onclick={exportDiagram}
        title="Export as JSON file">
        Export JSON
      </button>
      
      <button 
        class="action-btn"
        onclick={importDiagram}
        title="Import JSON file">
        Import JSON
      </button>
    </div>
    
    {#if selectedElement}
      <div class="tool-section properties">
        <h3>Properties</h3>
        
        {#if selectedElement.text !== undefined}
          <div class="inline-edit-hint">
            💡 Double-click element to edit text inline
          </div>
        {/if}
        
        {#if selectedElement.type === 'box'}
          <label>
            Width:
            <input 
              type="number" 
              bind:value={selectedElement.width}
              min="50"
              max="500"
            />
          </label>
          
          <label>
            Height:
            <input 
              type="number" 
              bind:value={selectedElement.height}
              min="30"
              max="300"
            />
          </label>
          
          <label class="checkbox">
            <input 
              type="checkbox" 
              bind:checked={selectedElement.shadow}
            />
            Drop Shadow
          </label>
          
          <label class="checkbox">
            <input 
              type="checkbox" 
              bind:checked={selectedElement.dashed}
            />
            Dashed Outline
          </label>
        {/if}
        
        {#if selectedElement.type === 'text'}
          <label>
            Font Size:
            <input 
              type="number" 
              bind:value={selectedElement.fontSize}
              min="8"
              max="72"
            />
          </label>
        {/if}
        
        {#if selectedElement.type === 'line'}
          <div class="inline-edit-hint">
            💡 Double-click line to add/edit label
          </div>
          
          <label class="checkbox">
            <input 
              type="checkbox" 
              bind:checked={selectedElement.hasCircle}
            />
            End Circle
          </label>
          
          <label class="checkbox">
            <input 
              type="checkbox" 
              bind:checked={selectedElement.hasArrow}
            />
            Arrow Head
          </label>
        {/if}
      </div>
    {/if}
  </div>
  
  <div 
    class="canvas"
    bind:this={canvasElement}
    onclick={handleCanvasClick}
    role="application"
    aria-label="Diagram canvas">
    
    <svg width="100%" height="100%" class="diagram-svg">
      <defs>
        <marker 
          id="arrowhead" 
          markerWidth="10" 
          markerHeight="10" 
          refX="9" 
          refY="3" 
          orient="auto">
          <polygon points="0 0, 10 3, 0 6" fill="#000000" />
        </marker>
      </defs>
      
      {#each elements as element (element.id)}
        {#if element.type === 'box'}
          <g 
            class="element box-element"
            class:selected={selectedElement?.id === element.id}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            {#if element.shadow}
              <!-- Shadow -->
              <rect 
                x={element.x + 3} 
                y={element.y + 3} 
                width={element.width} 
                height={element.height} 
                fill="#cccccc" 
                stroke="none" />
            {/if}
            
            <!-- Main box -->
            <rect 
              x={element.x} 
              y={element.y} 
              width={element.width} 
              height={element.height} 
              fill="#ffffff" 
              stroke="#000000" 
              stroke-width="2"
              stroke-dasharray={element.dashed ? "5,5" : "none"} />
            
            {#if editingText?.id === element.id}
              <!-- Inline editing with foreignObject -->
              <foreignObject 
                x={element.x} 
                y={element.y} 
                width={element.width} 
                height={element.height}>
                <input 
                  id={`edit-${element.id}`}
                  type="text" 
                  bind:value={element.text}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Enter') finishEditingText();
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; text-align: center; font-family: ui-monospace, 'Courier New', monospace; font-size: 12px; border: 2px solid #ff5722; background: white;"
                />
              </foreignObject>
            {:else}
              <text 
                x={element.x + element.width / 2} 
                y={element.y + element.height / 2} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size="12" 
                text-anchor="middle" 
                dominant-baseline="middle" 
                fill="#000000"
                pointer-events="none">
                {element.text}
              </text>
            {/if}
            
            <!-- Resize handles (only when selected) -->
            {#if selectedElement?.id === element.id}
              <!-- NW handle -->
              <circle 
                cx={element.x} 
                cy={element.y} 
                r="5" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'nw')} />
              
              <!-- NE handle -->
              <circle 
                cx={element.x + element.width} 
                cy={element.y} 
                r="5" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'ne')} />
              
              <!-- SW handle -->
              <circle 
                cx={element.x} 
                cy={element.y + element.height} 
                r="5" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'sw')} />
              
              <!-- SE handle -->
              <circle 
                cx={element.x + element.width} 
                cy={element.y + element.height} 
                r="5" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'se')} />
            {/if}
          </g>
          
        {:else if element.type === 'line'}
          {@const midX = (element.x1 + element.x2) / 2}
          {@const midY = (element.y1 + element.y2) / 2}
          
          <g 
            class="element line-element"
            class:selected={selectedElement?.id === element.id}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            <line 
              x1={element.x1} 
              y1={element.y1} 
              x2={element.x2} 
              y2={element.y2} 
              stroke="#000000" 
              stroke-width="2"
              marker-end={element.hasArrow ? "url(#arrowhead)" : ""} />
            
            {#if element.hasCircle}
              <circle 
                cx={element.x2} 
                cy={element.y2} 
                r="4" 
                fill="#000000" />
            {/if}
            
            <!-- Invisible thicker line for easier clicking -->
            <line 
              x1={element.x1} 
              y1={element.y1} 
              x2={element.x2} 
              y2={element.y2} 
              stroke="transparent" 
              stroke-width="10" />
            
            <!-- Line label centered above line -->
            {#if editingText?.id === element.id && editingLineLabel}
              <foreignObject 
                x={midX - 75} 
                y={midY - 20} 
                width="150" 
                height="30">
                <input 
                  id={`edit-line-label-${element.id}`}
                  type="text" 
                  bind:value={element.label}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Enter') finishEditingText();
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; text-align: center; font-family: ui-monospace, 'Courier New', monospace; font-size: 11px; border: 2px solid #ff5722; background: white; padding: 2px;"
                />
              </foreignObject>
            {:else if element.label && element.label !== ''}
              <text 
                x={midX} 
                y={midY - 1} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size="11" 
                text-anchor="middle" 
                dominant-baseline="baseline" 
                fill="#000000"
                pointer-events="none">
                {element.label}
              </text>
            {/if}
            
            <!-- Resize handles (only when selected) -->
            {#if selectedElement?.id === element.id}
              <!-- Start handle -->
              <circle 
                cx={element.x1} 
                cy={element.y1} 
                r="6" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'start')} />
              
              <!-- End handle -->
              <circle 
                cx={element.x2} 
                cy={element.y2} 
                r="6" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'end')} />
            {/if}
          </g>
          
        {:else if element.type === 'text'}
          <g 
            class="element text-element"
            class:selected={selectedElement?.id === element.id}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            {#if editingText?.id === element.id}
              <!-- Inline editing with foreignObject -->
              <foreignObject 
                x={element.x} 
                y={element.y - (element.fontSize || 14)} 
                width="300" 
                height={(element.fontSize || 14) * 2}>
                <input 
                  id={`edit-${element.id}`}
                  type="text" 
                  bind:value={element.text}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Enter') finishEditingText();
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; font-family: ui-monospace, 'Courier New', monospace; font-size: {element.fontSize || 14}px; border: 2px solid #ff5722; background: white; padding: 4px;"
                />
              </foreignObject>
            {:else}
              <text 
                x={element.x} 
                y={element.y} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size={element.fontSize || 14} 
                fill="#000000">
                {element.text}
              </text>
            {/if}
          </g>
        {/if}
      {/each}
    </svg>
    
    <div class="canvas-hint">
      {#if tool === 'select'}
        Click: select • Drag: move • Double-click: edit text/add line label • Arrow keys: nudge 1px • Line endpoints snap to edges • Hold Shift: snap angles
      {:else if tool === 'box'}
        Click anywhere to add a box
      {:else if tool === 'text'}
        Click anywhere to add text
      {:else if tool === 'line'}
        Click anywhere to add a line (use Properties panel to add circle/arrow)
      {/if}
    </div>
  </div>
</div>

<style>
  .builder-container {
    display: flex;
    gap: 0;
    height: 85vh;
    background: #ffffff;
    border: 2px solid #000000;
  }
  
  .toolbar {
    width: 220px;
    background: #ffffff;
    border-right: 2px solid #000000;
    padding: 16px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  
  .tool-section {
    border-bottom: 1px solid #cccccc;
    padding-bottom: 16px;
  }
  
  .tool-section:last-child {
    border-bottom: none;
  }
  
  .tool-section h3 {
    font-size: 12px;
    font-weight: 400;
    margin: 0 0 12px 0;
    color: #666666;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-family: ui-monospace, 'Courier New', monospace;
  }
  
  .tool-btn {
    display: flex;
    align-items: center;
    gap: 8px;
    width: 100%;
    padding: 10px 12px;
    background: #ffffff;
    border: 2px solid #000000;
    cursor: pointer;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 12px;
    font-weight: 400;
    color: #000000;
    margin-bottom: 8px;
    transition: all 0.2s ease;
  }
  
  .tool-btn:hover {
    background: #f5f5f5;
  }
  
  .tool-btn.active {
    background: #ff5722;
    color: #ffffff;
    border-color: #ff5722;
  }
  
  .tool-btn svg {
    flex-shrink: 0;
  }
  
  .action-btn {
    width: 100%;
    padding: 10px 12px;
    background: #ffffff;
    border: 2px solid #000000;
    cursor: pointer;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 12px;
    font-weight: 400;
    color: #000000;
    margin-bottom: 8px;
    transition: all 0.2s ease;
  }
  
  .action-btn:hover:not(:disabled) {
    background: #ff5722;
    color: #ffffff;
    border-color: #ff5722;
  }
  
  .action-btn:disabled {
    opacity: 0.3;
    cursor: not-allowed;
  }
  
  .properties label {
    display: block;
    margin-bottom: 12px;
    font-size: 11px;
    color: #333333;
    font-family: ui-monospace, 'Courier New', monospace;
  }
  
  .properties input[type="text"],
  .properties input[type="number"] {
    width: 100%;
    padding: 6px 8px;
    margin-top: 4px;
    border: 1px solid #000000;
    background: #ffffff;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
  }
  
  .properties input[type="text"]:focus,
  .properties input[type="number"]:focus {
    outline: 2px solid #ff5722;
    outline-offset: 0;
  }
  
  .properties .checkbox {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .properties .checkbox input {
    width: auto;
    margin: 0;
  }
  
  .filename-input {
    width: 100%;
    padding: 6px 8px;
    margin-top: 4px;
    margin-bottom: 12px;
    border: 1px solid #000000;
    background: #ffffff;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
    font-weight: 600;
  }
  
  .filename-input:focus {
    outline: 2px solid #ff5722;
    outline-offset: 0;
  }
  
  .design-select {
    width: 100%;
    padding: 6px 8px;
    margin-top: 4px;
    margin-bottom: 12px;
    border: 1px solid #000000;
    background: #ffffff;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
    cursor: pointer;
  }
  
  .design-select:focus {
    outline: 2px solid #ff5722;
    outline-offset: 0;
  }
  
  .delete-btn {
    background: #ffffff;
    border-color: #cc0000;
    color: #cc0000;
  }
  
  .delete-btn:hover:not(:disabled) {
    background: #cc0000;
    color: #ffffff;
  }
  
  .inline-edit-hint {
    padding: 8px;
    margin-bottom: 12px;
    background: #fff9e6;
    border: 1px solid #ffa500;
    font-size: 10px;
    line-height: 1.4;
    font-family: ui-monospace, 'Courier New', monospace;
  }
  
  .canvas {
    flex: 1;
    background: #ffffff;
    position: relative;
    overflow: hidden;
    cursor: crosshair;
  }
  
  .canvas[data-tool="select"] {
    cursor: default;
  }
  
  .diagram-svg {
    width: 100%;
    height: 100%;
  }
  
  .element {
    cursor: move;
  }
  
  .element.selected rect {
    stroke: #ff5722;
    stroke-width: 3;
  }
  
  .element.selected line {
    stroke: #ff5722;
    stroke-width: 3;
  }
  
  .element.selected text {
    fill: #ff5722;
  }
  
  .resize-handle {
    cursor: pointer;
  }
  
  .resize-handle:hover {
    r: 7;
    opacity: 0.8;
  }
  
  .canvas-hint {
    position: absolute;
    bottom: 16px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.8);
    color: #ffffff;
    padding: 8px 16px;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
    pointer-events: none;
  }
</style>

