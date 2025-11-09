<script>
  import { onMount } from 'svelte';
  
  let elements = $state([]);
  let selectedElements = $state([]); // Changed to array for multi-select
  let selectedElement = $state(null); // Keep for backward compatibility
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
  let showGrid = $state(false);
  let gridSize = $state(20);
  let isSnapKeyPressed = $state(false);
  let clipboard = $state([]);
  let zoomLevel = $state(1);
  let panX = $state(0);
  let panY = $state(0);
  let isPanning = $state(false);
  let panStartX = $state(0);
  let panStartY = $state(0);
  let isSpacePressed = $state(false);
  let showAboutModal = $state(false);
  let showHint = $state(true);
  let hintTimeout = null;
  let isMKeyPressed = $state(false);
  let selectionRect = $state(null); // { startX, startY, endX, endY } in canvas coordinates
  let isDrawingSelection = $state(false);
  let justFinishedSelection = $state(false);
  let history = $state([]);
  let historyIndex = $state(-1);
  let maxHistorySize = 50;
  
  // History management
  function saveHistory() {
    // Remove any history after current index (if user did undo, then made a change)
    history = history.slice(0, historyIndex + 1);
    
    // Deep clone the elements array
    const snapshot = JSON.parse(JSON.stringify(elements));
    history.push(snapshot);
    
    // Limit history size
    if (history.length > maxHistorySize) {
      history.shift();
    } else {
      historyIndex++;
    }
  }
  
  function undo() {
    if (historyIndex > 0) {
      historyIndex--;
      elements = JSON.parse(JSON.stringify(history[historyIndex]));
      selectedElements = [];
      selectedElement = null;
      editingText = null;
      editingLineLabel = false;
    }
  }
  
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
      dashed: false,
      fontSize: 12,
      bgColor: '#ffffff',
      textColor: '#000000',
      locked: false
    };
  }
  
  function createCircle(x, y) {
    return {
      id: Date.now(),
      type: 'circle',
      x: x - 50,
      y: y - 50,
      radius: 50,
      text: 'Circle',
      shadow: true,
      dashed: false,
      fontSize: 12,
      bgColor: '#ffffff',
      textColor: '#000000',
      locked: false
    };
  }
  
  function createText(x, y) {
    return {
      id: Date.now(),
      type: 'text',
      x: x - 50,
      y: y - 10,
      text: 'Text Label',
      fontSize: 14,
      locked: false
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
      label: '',
      locked: false
    };
  }
  
  // Transform screen coordinates to canvas coordinates (accounting for zoom and pan)
  function screenToCanvas(screenX, screenY) {
    const rect = canvasElement.getBoundingClientRect();
    const x = (screenX - rect.left - panX) / zoomLevel;
    const y = (screenY - rect.top - panY) / zoomLevel;
    return { x, y };
  }
  
  function zoomToPoint(newZoom, screenX, screenY) {
    const rect = canvasElement.getBoundingClientRect();
    const mouseX = screenX - rect.left;
    const mouseY = screenY - rect.top;
    
    // Calculate canvas coordinates under mouse before zoom
    const canvasX = (mouseX - panX) / zoomLevel;
    const canvasY = (mouseY - panY) / zoomLevel;
    
    // Apply new zoom level
    zoomLevel = Math.max(0.1, Math.min(newZoom, 5));
    
    // Adjust pan so the same canvas point is under the mouse
    panX = mouseX - canvasX * zoomLevel;
    panY = mouseY - canvasY * zoomLevel;
  }
  
  function zoomIn() {
    // Zoom to center of canvas
    const rect = canvasElement.getBoundingClientRect();
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;
    zoomToPoint(zoomLevel * 1.1, centerX, centerY);
  }
  
  function zoomOut() {
    // Zoom to center of canvas
    const rect = canvasElement.getBoundingClientRect();
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;
    zoomToPoint(zoomLevel / 1.1, centerX, centerY);
  }
  
  function resetZoom() {
    zoomLevel = 1;
    panX = 0;
    panY = 0;
  }
  
  function handleWheel(e) {
    if (e.ctrlKey || e.metaKey) {
      e.preventDefault();
      const delta = e.deltaY;
      const zoomFactor = delta < 0 ? 1.05 : 1 / 1.05;
      zoomToPoint(zoomLevel * zoomFactor, e.clientX, e.clientY);
    }
  }
  
  function handleCanvasMouseDown(e) {
    // Check if space key is pressed for panning
    if (isSpacePressed) {
      isPanning = true;
      panStartX = e.clientX - panX;
      panStartY = e.clientY - panY;
      e.preventDefault();
      return;
    }
    
    // Check if 'm' key is pressed for multi-select area
    if (isMKeyPressed && tool === 'select') {
      isDrawingSelection = true;
      const { x, y } = screenToCanvas(e.clientX, e.clientY);
      selectionRect = { startX: x, startY: y, endX: x, endY: y };
      e.preventDefault();
      return;
    }
  }
  
  function handleCanvasClick(e) {
    // Don't place elements if we're panning
    if (isPanning) {
      return;
    }
    
    // Don't clear selection if we just finished a selection rectangle
    if (justFinishedSelection) {
      justFinishedSelection = false;
      return;
    }
    
    if (tool === 'select') {
      selectedElement = null;
      selectedElements = [];
      editingText = null;
      editingLineLabel = false;
      return;
    }
    
    const { x, y } = screenToCanvas(e.clientX, e.clientY);
    
    if (tool === 'box') {
      saveHistory();
      elements = [...elements, createBox(x, y)];
    } else if (tool === 'circle') {
      saveHistory();
      elements = [...elements, createCircle(x, y)];
    } else if (tool === 'text') {
      saveHistory();
      elements = [...elements, createText(x, y)];
    } else if (tool === 'line') {
      saveHistory();
      elements = [...elements, createLine(x, y)];
    }
    
    tool = 'select';
  }
  
  function handleElementClick(e, element) {
    e.stopPropagation();
    
    // Don't change selection if 'M' key is pressed (multi-select mode)
    if (isMKeyPressed) {
      return;
    }
    
    // Multi-select with Ctrl/Cmd key
    if (e.ctrlKey || e.metaKey) {
      const index = selectedElements.findIndex(el => el.id === element.id);
      if (index >= 0) {
        // Deselect if already selected
        selectedElements = selectedElements.filter(el => el.id !== element.id);
      } else {
        // Add to selection
        selectedElements = [...selectedElements, element];
      }
      // Update selectedElement for backward compatibility
      selectedElement = selectedElements[0] || null;
    } else {
      // If element is already selected, do nothing (keep selection as is)
      const isAlreadySelected = selectedElements.find(el => el.id === element.id);
      if (!isAlreadySelected) {
        // Only change selection if clicking on an unselected element
        selectedElements = [element];
        selectedElement = element;
      }
    }
  }
  
  function handleDoubleClick(e, element) {
    e.stopPropagation();
    e.preventDefault();
    
    // Single-select the double-clicked element for editing
    selectedElements = [element];
    selectedElement = element;
    
    if (element.type === 'line') {
      // For lines, enable label editing
      if (element.label === undefined) {
        element.label = '';
      }
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
  
  // Helper function to snap to grid
  function snapToGrid(x, y) {
    return {
      x: Math.round(x / gridSize) * gridSize,
      y: Math.round(y / gridSize) * gridSize
    };
  }
  
  // Helper function to snap line to vertical/horizontal
  function snapLineToAxis(x1, y1, x2, y2) {
    const dx = Math.abs(x2 - x1);
    const dy = Math.abs(y2 - y1);
    const threshold = gridSize * 0.5; // Snap if within half a grid size
    
    // If close to horizontal, make it perfectly horizontal
    if (dy < threshold && dy < dx) {
      return { x1, y1, x2, y2: y1 };
    }
    
    // If close to vertical, make it perfectly vertical
    if (dx < threshold && dx < dy) {
      return { x1, y1, x2: x1, y2 };
    }
    
    // Otherwise return as is
    return { x1, y1, x2, y2 };
  }
  
  // Helper function to find the closest point on a line segment
  function closestPointOnLineSegment(x, y, x1, y1, x2, y2) {
    const dx = x2 - x1;
    const dy = y2 - y1;
    const lengthSquared = dx * dx + dy * dy;
    
    if (lengthSquared === 0) {
      return { x: x1, y: y1 };
    }
    
    // Calculate projection parameter t
    let t = ((x - x1) * dx + (y - y1) * dy) / lengthSquared;
    
    // Clamp t to [0, 1] to stay on the segment
    t = Math.max(0, Math.min(1, t));
    
    return {
      x: x1 + t * dx,
      y: y1 + t * dy
    };
  }
  
  // Helper function to find snap points near a coordinate
  function findSnapPoint(x, y, excludeElementId, otherEndX = null, otherEndY = null) {
    const snapDistance = 10; // pixels
    
    for (const el of elements) {
      if (el.id === excludeElementId) continue;
      
      if (el.type === 'box') {
        // Check box edges (corners and centers)
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
        
        // If other end of line is specified, also check for edge alignment
        if (otherEndX !== null && otherEndY !== null) {
          // Check for points on box edges that align with the other end (for orthogonal lines)
          const alignmentPoints = [];
          
          // Top edge - check if we can align Y with other end
          if (Math.abs(y - el.y) < snapDistance) {
            alignmentPoints.push({ x: otherEndX, y: el.y });
          }
          
          // Bottom edge - check if we can align Y with other end
          if (Math.abs(y - (el.y + el.height)) < snapDistance) {
            alignmentPoints.push({ x: otherEndX, y: el.y + el.height });
          }
          
          // Left edge - check if we can align X with other end
          if (Math.abs(x - el.x) < snapDistance) {
            alignmentPoints.push({ x: el.x, y: otherEndY });
          }
          
          // Right edge - check if we can align X with other end
          if (Math.abs(x - (el.x + el.width)) < snapDistance) {
            alignmentPoints.push({ x: el.x + el.width, y: otherEndY });
          }
          
          // Return the closest alignment point
          for (const point of alignmentPoints) {
            // Check if this point is within the box bounds
            if (point.x >= el.x && point.x <= el.x + el.width &&
                point.y >= el.y && point.y <= el.y + el.height) {
              const dist = Math.sqrt(Math.pow(x - point.x, 2) + Math.pow(y - point.y, 2));
              if (dist < snapDistance * 2) { // Slightly larger tolerance for edge alignment
                return point;
              }
            }
          }
        }
      } else if (el.type === 'circle') {
        // Check circle cardinal points
        const points = [
          { x: el.x + el.radius, y: el.y }, // top
          { x: el.x + el.radius * 2, y: el.y + el.radius }, // right
          { x: el.x + el.radius, y: el.y + el.radius * 2 }, // bottom
          { x: el.x, y: el.y + el.radius }, // left
          { x: el.x + el.radius, y: el.y + el.radius }, // center
        ];
        
        for (const point of points) {
          const dist = Math.sqrt(Math.pow(x - point.x, 2) + Math.pow(y - point.y, 2));
          if (dist < snapDistance) {
            return point;
          }
        }
      } else if (el.type === 'line') {
        // First check line endpoints
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
        
        // Then check if close to the line itself
        const closestPoint = closestPointOnLineSegment(x, y, el.x1, el.y1, el.x2, el.y2);
        const distToLine = Math.sqrt(Math.pow(x - closestPoint.x, 2) + Math.pow(y - closestPoint.y, 2));
        
        if (distToLine < snapDistance) {
          return closestPoint;
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
    
    // Don't start dragging if 'M' key is pressed (multi-select mode)
    if (isMKeyPressed) {
      return;
    }
    
    // Don't start dragging if element is locked
    if (element.locked) {
      return;
    }
    
    e.stopPropagation();
    
    // If clicked element is not in selection, select it (unless multi-selecting)
    if (!selectedElements.find(el => el.id === element.id) && !(e.ctrlKey || e.metaKey)) {
      selectedElements = [element];
    }
    
    selectedElement = element;
    isDragging = true;
    
    const { x, y } = screenToCanvas(e.clientX, e.clientY);
    
    // Store initial positions for all selected elements (excluding locked ones)
    const elementsData = selectedElements
      .filter(el => !el.locked)
      .map(el => ({
        element: el,
        startX: el.x || el.x1,
        startY: el.y || el.y1,
        startX2: el.x2,
        startY2: el.y2
      }));
    
    dragStart = {
      x: x,
      y: y,
      elementX: element.x || element.x1,
      elementY: element.y || element.y1,
      elementWidth: element.width,
      elementHeight: element.height,
      elementRadius: element.radius,
      elementX2: element.x2,
      elementY2: element.y2,
      elementsData: elementsData // Store for multi-element dragging
    };
  }
  
  function handleResizeStart(e, element, handle) {
    e.stopPropagation();
    
    // Don't allow resizing if element is locked
    if (element.locked) {
      return;
    }
    
    selectedElement = element;
    isResizing = true;
    resizeHandle = handle;
    
    const { x, y } = screenToCanvas(e.clientX, e.clientY);
    dragStart = {
      x: x,
      y: y,
      elementX: element.x || element.x1,
      elementY: element.y || element.y1,
      elementWidth: element.width,
      elementHeight: element.height,
      elementRadius: element.radius,
      elementX2: element.x2,
      elementY2: element.y2
    };
  }
  
  function handleMouseMove(e) {
    // Show hint and reset fade timer
    showHint = true;
    if (hintTimeout) clearTimeout(hintTimeout);
    hintTimeout = setTimeout(() => {
      showHint = false;
    }, 3000);
    
    // Handle panning
    if (isPanning) {
      panX = e.clientX - panStartX;
      panY = e.clientY - panStartY;
      return;
    }
    
    // Handle drawing selection rectangle
    if (isDrawingSelection && selectionRect) {
      const { x, y } = screenToCanvas(e.clientX, e.clientY);
      selectionRect.endX = x;
      selectionRect.endY = y;
      return;
    }
    
    if ((!isDragging && !isResizing) || !dragStart || !selectedElement) return;
    
    const { x: currentX, y: currentY } = screenToCanvas(e.clientX, e.clientY);
    
    const dx = currentX - dragStart.x;
    const dy = currentY - dragStart.y;
    
    if (isResizing) {
      // Handle resizing
      if (selectedElement.type === 'box') {
        if (resizeHandle === 'se') {
          let cornerX = dragStart.elementX + dragStart.elementWidth + dx;
          let cornerY = dragStart.elementY + dragStart.elementHeight + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(cornerX, cornerY);
            cornerX = snapped.x;
            cornerY = snapped.y;
          }
          
          const newWidth = Math.max(20, cornerX - dragStart.elementX);
          const newHeight = Math.max(20, cornerY - dragStart.elementY);
          
          if (e.shiftKey) {
            // Force square by using the larger dimension
            const size = Math.max(newWidth, newHeight);
            selectedElement.width = size;
            selectedElement.height = size;
          } else {
            selectedElement.width = newWidth;
            selectedElement.height = newHeight;
          }
        } else if (resizeHandle === 'sw') {
          let cornerX = dragStart.elementX + dx;
          let cornerY = dragStart.elementY + dragStart.elementHeight + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(cornerX, cornerY);
            cornerX = snapped.x;
            cornerY = snapped.y;
          }
          
          const newWidth = Math.max(20, dragStart.elementX + dragStart.elementWidth - cornerX);
          const newHeight = Math.max(20, cornerY - dragStart.elementY);
          
          if (e.shiftKey) {
            const size = Math.max(newWidth, newHeight);
            selectedElement.x = dragStart.elementX + dragStart.elementWidth - size;
            selectedElement.width = size;
            selectedElement.height = size;
          } else {
            selectedElement.x = dragStart.elementX + dragStart.elementWidth - newWidth;
            selectedElement.width = newWidth;
            selectedElement.height = newHeight;
          }
        } else if (resizeHandle === 'ne') {
          let cornerX = dragStart.elementX + dragStart.elementWidth + dx;
          let cornerY = dragStart.elementY + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(cornerX, cornerY);
            cornerX = snapped.x;
            cornerY = snapped.y;
          }
          
          const newWidth = Math.max(20, cornerX - dragStart.elementX);
          const newHeight = Math.max(20, dragStart.elementY + dragStart.elementHeight - cornerY);
          
          if (e.shiftKey) {
            const size = Math.max(newWidth, newHeight);
            selectedElement.width = size;
            selectedElement.y = dragStart.elementY + dragStart.elementHeight - size;
            selectedElement.height = size;
          } else {
            selectedElement.width = newWidth;
            selectedElement.y = dragStart.elementY + dragStart.elementHeight - newHeight;
            selectedElement.height = newHeight;
          }
        } else if (resizeHandle === 'nw') {
          let cornerX = dragStart.elementX + dx;
          let cornerY = dragStart.elementY + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(cornerX, cornerY);
            cornerX = snapped.x;
            cornerY = snapped.y;
          }
          
          const newWidth = Math.max(20, dragStart.elementX + dragStart.elementWidth - cornerX);
          const newHeight = Math.max(20, dragStart.elementY + dragStart.elementHeight - cornerY);
          
          if (e.shiftKey) {
            const size = Math.max(newWidth, newHeight);
            selectedElement.x = dragStart.elementX + dragStart.elementWidth - size;
            selectedElement.y = dragStart.elementY + dragStart.elementHeight - size;
            selectedElement.width = size;
            selectedElement.height = size;
          } else {
            selectedElement.x = dragStart.elementX + dragStart.elementWidth - newWidth;
            selectedElement.y = dragStart.elementY + dragStart.elementHeight - newHeight;
            selectedElement.width = newWidth;
            selectedElement.height = newHeight;
          }
        }
      } else if (selectedElement.type === 'circle') {
        // Handle circle resizing - always resize from center
        // Circle center stays fixed at (centerX, centerY)
        const centerX = dragStart.elementX + dragStart.elementRadius;
        const centerY = dragStart.elementY + dragStart.elementRadius;
        
        // Calculate new radius based on distance from center to mouse position
        const distanceX = currentX - centerX;
        const distanceY = currentY - centerY;
        const distance = Math.sqrt(distanceX * distanceX + distanceY * distanceY);
        let newRadius = Math.max(10, distance);
        
        // Apply grid snapping if 's' key is held
        if (isSnapKeyPressed) {
          // Snap the handle position to grid and recalculate radius
          const snapPoint = snapToGrid(currentX, currentY);
          const snappedDistanceX = snapPoint.x - centerX;
          const snappedDistanceY = snapPoint.y - centerY;
          const snappedDistance = Math.sqrt(snappedDistanceX * snappedDistanceX + snappedDistanceY * snappedDistanceY);
          newRadius = Math.max(10, snappedDistance);
        }
        
        // Update circle to maintain center position
        selectedElement.radius = newRadius;
        selectedElement.x = centerX - newRadius;
        selectedElement.y = centerY - newRadius;
      } else if (selectedElement.type === 'line') {
        if (resizeHandle === 'start') {
          // Check for snap points, passing the other end's coordinates for edge alignment
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id, selectedElement.x2, selectedElement.y2);
          let finalX = snapPoint ? snapPoint.x : currentX;
          let finalY = snapPoint ? snapPoint.y : currentY;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(finalX, finalY);
            finalX = snapped.x;
            finalY = snapped.y;
          }
          
          // Ensure minimum line length of 10 pixels
          const distance = Math.sqrt(Math.pow(finalX - selectedElement.x2, 2) + Math.pow(finalY - selectedElement.y2, 2));
          if (distance < 10) {
            const angle = Math.atan2(finalY - selectedElement.y2, finalX - selectedElement.x2);
            finalX = selectedElement.x2 + 10 * Math.cos(angle);
            finalY = selectedElement.y2 + 10 * Math.sin(angle);
          }
          
          if (e.shiftKey) {
            // Snap to 15-degree intervals relative to the other end
            const angle = Math.atan2(finalY - selectedElement.y2, finalX - selectedElement.x2);
            const snapAngle = Math.round(angle / (Math.PI / 12)) * (Math.PI / 12); // 15 degrees = PI/12
            const dist = Math.max(10, Math.sqrt(Math.pow(finalX - selectedElement.x2, 2) + Math.pow(finalY - selectedElement.y2, 2)));
            selectedElement.x1 = selectedElement.x2 + dist * Math.cos(snapAngle);
            selectedElement.y1 = selectedElement.y2 + dist * Math.sin(snapAngle);
          } else {
            selectedElement.x1 = finalX;
            selectedElement.y1 = finalY;
            
            // Apply axis snapping if 's' key is held
            if (isSnapKeyPressed) {
              const snapped = snapLineToAxis(selectedElement.x1, selectedElement.y1, selectedElement.x2, selectedElement.y2);
              selectedElement.x1 = snapped.x1;
              selectedElement.y1 = snapped.y1;
              selectedElement.x2 = snapped.x2;
              selectedElement.y2 = snapped.y2;
            }
          }
        } else if (resizeHandle === 'end') {
          // Check for snap points, passing the other end's coordinates for edge alignment
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id, selectedElement.x1, selectedElement.y1);
          let finalX = snapPoint ? snapPoint.x : currentX;
          let finalY = snapPoint ? snapPoint.y : currentY;
          
          // If snapping and line has a circle, offset by circle radius to keep it visible
          if (snapPoint && selectedElement.hasCircle) {
            const circleRadius = 4;
            const angle = Math.atan2(snapPoint.y - selectedElement.y1, snapPoint.x - selectedElement.x1);
            finalX = snapPoint.x - circleRadius * Math.cos(angle);
            finalY = snapPoint.y - circleRadius * Math.sin(angle);
          }
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(finalX, finalY);
            finalX = snapped.x;
            finalY = snapped.y;
          }
          
          // Ensure minimum line length of 10 pixels
          const distance = Math.sqrt(Math.pow(finalX - selectedElement.x1, 2) + Math.pow(finalY - selectedElement.y1, 2));
          if (distance < 10) {
            const angle = Math.atan2(finalY - selectedElement.y1, finalX - selectedElement.x1);
            finalX = selectedElement.x1 + 10 * Math.cos(angle);
            finalY = selectedElement.y1 + 10 * Math.sin(angle);
          }
          
          if (e.shiftKey) {
            // Snap to 15-degree intervals relative to the start
            const angle = Math.atan2(finalY - selectedElement.y1, finalX - selectedElement.x1);
            const snapAngle = Math.round(angle / (Math.PI / 12)) * (Math.PI / 12); // 15 degrees = PI/12
            const dist = Math.max(10, Math.sqrt(Math.pow(finalX - selectedElement.x1, 2) + Math.pow(finalY - selectedElement.y1, 2)));
            selectedElement.x2 = selectedElement.x1 + dist * Math.cos(snapAngle);
            selectedElement.y2 = selectedElement.y1 + dist * Math.sin(snapAngle);
          } else {
            selectedElement.x2 = finalX;
            selectedElement.y2 = finalY;
            
            // Apply axis snapping if 's' key is held
            if (isSnapKeyPressed) {
              const snapped = snapLineToAxis(selectedElement.x1, selectedElement.y1, selectedElement.x2, selectedElement.y2);
              selectedElement.x1 = snapped.x1;
              selectedElement.y1 = snapped.y1;
              selectedElement.x2 = snapped.x2;
              selectedElement.y2 = snapped.y2;
            }
          }
        }
      }
    } else if (isDragging) {
      // Handle dragging - move all selected elements together
      if (dragStart.elementsData && dragStart.elementsData.length >= 1) {
        // Multi-element drag (works for 1 or more elements)
        for (const data of dragStart.elementsData) {
          let newX = data.startX + dx;
          let newY = data.startY + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(newX, newY);
            newX = snapped.x;
            newY = snapped.y;
          }
          
          if (data.element.type === 'line') {
            const width = data.startX2 - data.startX;
            const height = data.startY2 - data.startY;
            
            if (isSnapKeyPressed) {
              const snappedEnd = snapToGrid(newX + width, newY + height);
              data.element.x1 = newX;
              data.element.y1 = newY;
              data.element.x2 = snappedEnd.x;
              data.element.y2 = snappedEnd.y;
              
              // Apply axis snapping
              const axisSnapped = snapLineToAxis(data.element.x1, data.element.y1, data.element.x2, data.element.y2);
              data.element.x1 = axisSnapped.x1;
              data.element.y1 = axisSnapped.y1;
              data.element.x2 = axisSnapped.x2;
              data.element.y2 = axisSnapped.y2;
            } else {
              data.element.x1 = newX;
              data.element.y1 = newY;
              data.element.x2 = newX + width;
              data.element.y2 = newY + height;
            }
          } else {
            data.element.x = newX;
            data.element.y = newY;
          }
        }
      } else {
        // Single element drag (existing behavior)
        let finalX = dragStart.elementX + dx;
        let finalY = dragStart.elementY + dy;
        
        // Apply grid snapping if 's' key is held
        if (isSnapKeyPressed) {
          const snapped = snapToGrid(finalX, finalY);
          finalX = snapped.x;
          finalY = snapped.y;
        }
        
        if (selectedElement.type === 'line') {
          const width = dragStart.elementX2 - dragStart.elementX;
          const height = dragStart.elementY2 - dragStart.elementY;
          
          if (isSnapKeyPressed) {
            // Snap both endpoints to grid
            const snappedStart = snapToGrid(finalX, finalY);
            const snappedEnd = snapToGrid(finalX + width, finalY + height);
            selectedElement.x1 = snappedStart.x;
            selectedElement.y1 = snappedStart.y;
            selectedElement.x2 = snappedEnd.x;
            selectedElement.y2 = snappedEnd.y;
            
            // Also apply axis snapping
            const axisSnapped = snapLineToAxis(selectedElement.x1, selectedElement.y1, selectedElement.x2, selectedElement.y2);
            selectedElement.x1 = axisSnapped.x1;
            selectedElement.y1 = axisSnapped.y1;
            selectedElement.x2 = axisSnapped.x2;
            selectedElement.y2 = axisSnapped.y2;
          } else {
            selectedElement.x1 = finalX;
            selectedElement.y1 = finalY;
            selectedElement.x2 = finalX + width;
            selectedElement.y2 = finalY + height;
          }
        } else {
          selectedElement.x = finalX;
          selectedElement.y = finalY;
        }
      }
    }
  }
  
  function handleMouseUp() {
    // Handle finishing selection rectangle
    if (isDrawingSelection && selectionRect) {
      const minX = Math.min(selectionRect.startX, selectionRect.endX);
      const maxX = Math.max(selectionRect.startX, selectionRect.endX);
      const minY = Math.min(selectionRect.startY, selectionRect.endY);
      const maxY = Math.max(selectionRect.startY, selectionRect.endY);
      
      // Select all elements that are completely within the selection rectangle
      const selectedInRect = elements.filter(element => {
        if (element.type === 'box') {
          // Check if box is completely inside the rectangle
          return element.x >= minX && 
                 element.x + element.width <= maxX && 
                 element.y >= minY && 
                 element.y + element.height <= maxY;
        } else if (element.type === 'circle') {
          // Check if entire circle is inside the rectangle
          const cx = element.x + element.radius;
          const cy = element.y + element.radius;
          return (cx - element.radius) >= minX && 
                 (cx + element.radius) <= maxX && 
                 (cy - element.radius) >= minY && 
                 (cy + element.radius) <= maxY;
        } else if (element.type === 'line') {
          // Check if both endpoints are within the rectangle
          return element.x1 >= minX && element.x1 <= maxX && 
                 element.y1 >= minY && element.y1 <= maxY &&
                 element.x2 >= minX && element.x2 <= maxX && 
                 element.y2 >= minY && element.y2 <= maxY;
        } else if (element.type === 'text') {
          // Check if text position is within rectangle
          return element.x >= minX && element.x <= maxX && 
                 element.y >= minY && element.y <= maxY;
        }
        return false;
      });
      
      if (selectedInRect.length > 0) {
        selectedElements = selectedInRect;
        selectedElement = selectedInRect[0];
      }
      
      isDrawingSelection = false;
      selectionRect = null;
      justFinishedSelection = true; // Prevent canvas click from clearing selection
      return;
    }
    
    // Save history after dragging or resizing
    if (isDragging || isResizing) {
      saveHistory();
    }
    
    isDragging = false;
    isResizing = false;
    resizeHandle = null;
    dragStart = null;
    isPanning = false;
  }
  
  function deleteSelected() {
    if (selectedElements.length > 0) {
      saveHistory();
      const selectedIds = selectedElements.map(el => el.id);
      elements = elements.filter(el => !selectedIds.includes(el.id));
      selectedElements = [];
      selectedElement = null;
    }
  }
  
  function copySelected() {
    // Deep clone selected elements
    clipboard = selectedElements.map(el => ({...el}));
  }
  
  function pasteFromClipboard() {
    saveHistory();
    const offset = 20; // Offset pasted elements by 20px
    const newIds = [];
    const baseId = Date.now();
    
    const newElements = clipboard.map((el, index) => {
      const newId = baseId + index + Math.random() * 1000;
      newIds.push(newId);
      
      const newEl = {...el, id: newId};
      
      if (newEl.type === 'line') {
        newEl.x1 += offset;
        newEl.y1 += offset;
        newEl.x2 += offset;
        newEl.y2 += offset;
      } else {
        newEl.x += offset;
        newEl.y += offset;
      }
      
      return newEl;
    });
    
    elements = [...elements, ...newElements];
    
    // Select the newly pasted elements by finding them in the elements array by ID
    selectedElements = elements.filter(el => newIds.includes(el.id));
    selectedElement = selectedElements[0] || null;
  }
  
  function handleKeyDown(e) {
    // Track 's' key for snap-to-grid
    if (e.key === 's' || e.key === 'S') {
      isSnapKeyPressed = true;
    }
    
    // Track 'm' key for multi-select mode
    if (e.key === 'm' || e.key === 'M') {
      isMKeyPressed = true;
    }
    
    // Track space key for panning (but don't pan if editing text)
    if (e.key === ' ' && !editingText && !editingLineLabel) {
      isSpacePressed = true;
      e.preventDefault();
    }
    
    // Don't process any keys if we're editing text
    if (editingText || editingLineLabel) {
      return;
    }
    
    // Undo: Ctrl/Cmd+Z
    if ((e.ctrlKey || e.metaKey) && e.key === 'z' && !e.shiftKey) {
      e.preventDefault();
      undo();
      return;
    }
    
    // Copy: Ctrl/Cmd+C
    if ((e.ctrlKey || e.metaKey) && e.key === 'c' && selectedElements.length > 0) {
      e.preventDefault();
      copySelected();
      return;
    }
    
    // Paste: Ctrl/Cmd+V
    if ((e.ctrlKey || e.metaKey) && e.key === 'v' && clipboard.length > 0) {
      e.preventDefault();
      pasteFromClipboard();
      return;
    }
    
    // Delete/Backspace to remove elements
    if ((e.key === 'Delete' || e.key === 'Backspace') && selectedElements.length > 0) {
      e.preventDefault();
      deleteSelected();
      return;
    }
    
    // Arrow key nudging - works when elements are selected and not editing
    if (selectedElements.length > 0) {
      e.preventDefault();
      
      for (const element of selectedElements) {
        // Skip locked elements
        if (element.locked) {
          continue;
        }
        
        if (e.key === 'ArrowUp') {
          if (element.type === 'line') {
            element.y1 -= 1;
            element.y2 -= 1;
          } else {
            element.y -= 1;
          }
        } else if (e.key === 'ArrowDown') {
          if (element.type === 'line') {
            element.y1 += 1;
            element.y2 += 1;
          } else {
            element.y += 1;
          }
        } else if (e.key === 'ArrowLeft') {
          if (element.type === 'line') {
            element.x1 -= 1;
            element.x2 -= 1;
          } else {
            element.x -= 1;
          }
        } else if (e.key === 'ArrowRight') {
          if (element.type === 'line') {
            element.x1 += 1;
            element.x2 += 1;
          } else {
            element.x += 1;
          }
        }
      }
    }
  }
  
  function handleKeyUp(e) {
    // Track 's' key release for snap-to-grid
    if (e.key === 's' || e.key === 'S') {
      isSnapKeyPressed = false;
    }
    
    // Track 'm' key release for multi-select mode
    if (e.key === 'm' || e.key === 'M') {
      isMKeyPressed = false;
      // Cancel any ongoing selection
      if (isDrawingSelection) {
        isDrawingSelection = false;
        selectionRect = null;
      }
    }
    
    // Track space key release for panning
    if (e.key === ' ') {
      isSpacePressed = false;
      isPanning = false;
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
      saveHistory();
      elements = designs[name].elements;
      currentDesignName = name;
      selectedElement = null;
      // Reset history after loading
      history = [];
      historyIndex = -1;
      saveHistory();
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
    saveHistory();
    elements = [];
    currentDesignName = 'Untitled';
    selectedElement = null;
    // Reset history for new design
    history = [];
    historyIndex = -1;
    saveHistory();
  }
  
  function exportDiagram() {
    const data = JSON.stringify(elements, null, 2);
    const blob = new Blob([data], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    const fileName = (currentDesignName.trim() || 'Untitled') + '.json';
    a.download = fileName;
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
            saveHistory();
            elements = JSON.parse(event.target.result);
            selectedElement = null;
            // Reset history after import
            history = [];
            historyIndex = -1;
            saveHistory();
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
    window.addEventListener('keyup', handleKeyUp);
    loadSavedDesignsList();
    // Initialize history with empty state
    saveHistory();
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  });
</script>

<svelte:window onmousemove={handleMouseMove} onmouseup={handleMouseUp} />

<div class="builder-container">
  <div class="left-panel">
  <div class="toolbar">
    <div class="tool-section">
      <h3>Tools</h3>
      <div class="tool-grid">
        <button 
          class="tool-btn"
          class:active={tool === 'select'}
          onclick={() => tool = 'select'}
          title="Select (Esc)">
          <svg width="20" height="20" viewBox="0 0 20 20">
            <path d="M2 2 L2 18 L8 12 L12 18 L15 16 L11 10 L18 8 Z" fill="currentColor"/>
          </svg>
          <span>Select</span>
        </button>
        
        <button 
          class="tool-btn"
          class:active={tool === 'box'}
          onclick={() => tool = 'box'}
          title="Add Box">
          <svg width="20" height="20" viewBox="0 0 20 20">
            <rect x="2" y="4" width="16" height="12" fill="none" stroke="currentColor" stroke-width="2"/>
          </svg>
          <span>Box</span>
        </button>
        
        <button 
          class="tool-btn"
          class:active={tool === 'circle'}
          onclick={() => tool = 'circle'}
          title="Add Circle">
          <svg width="20" height="20" viewBox="0 0 20 20">
            <circle cx="10" cy="10" r="8" fill="none" stroke="currentColor" stroke-width="2"/>
          </svg>
          <span>Circle</span>
        </button>
        
        <button 
          class="tool-btn"
          class:active={tool === 'text'}
          onclick={() => tool = 'text'}
          title="Add Text">
          <svg width="20" height="20" viewBox="0 0 20 20">
            <text x="10" y="15" font-size="14" fill="currentColor" text-anchor="middle" font-weight="bold">T</text>
          </svg>
          <span>Text</span>
        </button>
        
        <button 
          class="tool-btn"
          class:active={tool === 'line'}
          onclick={() => tool = 'line'}
          title="Add Line">
          <svg width="20" height="20" viewBox="0 0 20 20">
            <line x1="2" y1="10" x2="18" y2="10" stroke="currentColor" stroke-width="2"/>
          </svg>
          <span>Line</span>
        </button>
      </div>
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
        onclick={() => { saveHistory(); elements = []; }}
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
      
      <div class="button-row">
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
      </div>
      
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
      
      <div class="button-row">
        <button 
          class="action-btn"
          onclick={exportDiagram}
          title="Export design as file">
          Export
        </button>
        
        <button 
          class="action-btn"
          onclick={importDiagram}
          title="Import design from file">
          Import
        </button>
      </div>
    </div>
    
    <div class="tool-section">
      <h3>Zoom & View</h3>
      
      <div class="button-row">
        <button 
          class="action-btn"
          onclick={zoomIn}
          title="Zoom In (Ctrl/Cmd + Scroll)">
          Zoom In
        </button>
        
        <button 
          class="action-btn"
          onclick={zoomOut}
          title="Zoom Out (Ctrl/Cmd + Scroll)">
          Zoom Out
        </button>
      </div>
      
      <button 
        class="action-btn"
        onclick={resetZoom}
        title="Reset Zoom to 100%">
        Reset View
      </button>
      
      <p class="grid-hint">
        Zoom: {Math.round(zoomLevel * 100)}%<br/>
        Hold Ctrl/Cmd + Scroll to zoom<br/>
        Hold Space + Drag to pan
      </p>
    </div>
    
    <div class="tool-section">
      <h3>Grid</h3>
      
      <button 
        class="tool-btn"
        class:active={showGrid}
        onclick={() => showGrid = !showGrid}
        title="Toggle grid visibility">
        <svg width="20" height="20" viewBox="0 0 20 20">
          <line x1="5" y1="0" x2="5" y2="20" stroke="currentColor" stroke-width="1"/>
          <line x1="10" y1="0" x2="10" y2="20" stroke="currentColor" stroke-width="1"/>
          <line x1="15" y1="0" x2="15" y2="20" stroke="currentColor" stroke-width="1"/>
          <line x1="0" y1="5" x2="20" y2="5" stroke="currentColor" stroke-width="1"/>
          <line x1="0" y1="10" x2="20" y2="10" stroke="currentColor" stroke-width="1"/>
          <line x1="0" y1="15" x2="20" y2="15" stroke="currentColor" stroke-width="1"/>
        </svg>
        <span>Show Grid</span>
      </button>
      
      <label>
        Grid Size:
        <input 
          type="number" 
          bind:value={gridSize}
          min="5"
          max="100"
          step="5"
          class="grid-size-input"
        />
      </label>
      
      <p class="grid-hint">Hold 'S' while dragging to snap to grid. Lines also snap to vertical/horizontal when close.</p>
    </div>
  </div>
  </div>
  
  <div 
    class="canvas"
    class:space-pressed={isSpacePressed && !isPanning}
    class:panning={isPanning}
    class:m-key-pressed={isMKeyPressed && tool === 'select'}
    bind:this={canvasElement}
    onclick={handleCanvasClick}
    onmousedown={handleCanvasMouseDown}
    onwheel={handleWheel}
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
        
        <pattern 
          id="grid" 
          width={gridSize} 
          height={gridSize} 
          patternUnits="userSpaceOnUse">
          <path 
            d="M {gridSize} 0 L 0 0 0 {gridSize}" 
            fill="none" 
            stroke="#e0e0e0" 
            stroke-width="0.5"/>
        </pattern>
      </defs>
      
      {#if showGrid}
        <rect width="100%" height="100%" fill="url(#grid)" />
      {/if}
      
      <g transform="translate({panX}, {panY}) scale({zoomLevel})">
      {#each elements as element (element.id)}
        {#if element.type === 'box'}
          <g 
            class="element box-element"
            class:selected={selectedElements.some(el => el.id === element.id)}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            {#if element.shadow && element.bgColor !== 'none'}
              <!-- Shadow (only show when background is not transparent) -->
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
              fill={element.bgColor || '#ffffff'} 
              stroke="#000000" 
              stroke-width="2"
              stroke-dasharray={element.dashed ? "5,5" : "none"}
              pointer-events="all" />
            
            {#if editingText?.id === element.id}
              <!-- Inline editing with foreignObject -->
              <foreignObject 
                x={element.x} 
                y={element.y} 
                width={element.width} 
                height={element.height}>
                <textarea 
                  id={`edit-${element.id}`}
                  bind:value={element.text}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; text-align: center; font-family: ui-monospace, 'Courier New', monospace; font-size: 12px; border: 2px solid #ff5722; background: white; resize: none; overflow: auto;"
                />
              </foreignObject>
            {:else}
              {@const lines = (element.text || '').split('\n')}
              <text 
                x={element.x + element.width / 2} 
                y={element.y + element.height / 2 - (lines.length - 1) * (element.fontSize || 12) * 0.5} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size={element.fontSize || 12} 
                text-anchor="middle" 
                dominant-baseline="middle" 
                fill={element.textColor || '#000000'}
                pointer-events="none">
                {#each lines as line, i}
                  <tspan x={element.x + element.width / 2} dy={i === 0 ? 0 : (element.fontSize || 12)}>{line}</tspan>
                {/each}
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
          
        {:else if element.type === 'circle'}
          <g 
            class="element circle-element"
            class:selected={selectedElements.some(el => el.id === element.id)}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            {#if element.shadow && element.bgColor !== 'none'}
              <!-- Shadow (only show when background is not transparent) -->
              <circle 
                cx={element.x + element.radius + 3} 
                cy={element.y + element.radius + 3} 
                r={element.radius} 
                fill="#cccccc" 
                stroke="none" />
            {/if}
            
            <!-- Main circle -->
            <circle 
              cx={element.x + element.radius} 
              cy={element.y + element.radius} 
              r={element.radius} 
              fill={element.bgColor || '#ffffff'} 
              stroke="#000000" 
              stroke-width="2"
              stroke-dasharray={element.dashed ? "5,5" : "none"}
              pointer-events="all" />
            
            {#if editingText?.id === element.id}
              <!-- Inline editing with foreignObject -->
              <foreignObject 
                x={element.x} 
                y={element.y} 
                width={element.radius * 2} 
                height={element.radius * 2}>
                <textarea 
                  id={`edit-${element.id}`}
                  bind:value={element.text}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; text-align: center; font-family: ui-monospace, 'Courier New', monospace; font-size: 12px; border: 2px solid #ff5722; background: white; resize: none; overflow: auto;"
                />
              </foreignObject>
            {:else}
              {@const lines = (element.text || '').split('\n')}
              <text 
                x={element.x + element.radius} 
                y={element.y + element.radius - (lines.length - 1) * (element.fontSize || 12) * 0.5} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size={element.fontSize || 12} 
                text-anchor="middle" 
                dominant-baseline="middle" 
                fill={element.textColor || '#000000'}
                pointer-events="none">
                {#each lines as line, i}
                  <tspan x={element.x + element.radius} dy={i === 0 ? 0 : (element.fontSize || 12)}>{line}</tspan>
                {/each}
              </text>
            {/if}
            
            <!-- Resize handle (only when selected) -->
            {#if selectedElement?.id === element.id}
              <!-- Single resize handle at east (right) position -->
              <circle 
                cx={element.x + element.radius * 2} 
                cy={element.y + element.radius} 
                r="5" 
                fill="#ff5722" 
                stroke="#000000" 
                stroke-width="1"
                class="resize-handle"
                onmousedown={(e) => handleResizeStart(e, element, 'e')} />
            {/if}
          </g>
          
        {:else if element.type === 'line'}
          {@const midX = (element.x1 + element.x2) / 2}
          {@const midY = (element.y1 + element.y2) / 2}
          
          <g 
            class="element line-element"
            class:selected={selectedElements.some(el => el.id === element.id)}
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
              stroke-width="40" 
              pointer-events="stroke" />
            
            <!-- Line label centered above line -->
            {#if editingText?.id === element.id && editingLineLabel}
              <foreignObject 
                x={midX - 100} 
                y={midY - 60} 
                width="200" 
                height="60">
                <textarea 
                  id={`edit-line-label-${element.id}`}
                  bind:value={element.label}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; text-align: center; font-family: ui-monospace, 'Courier New', monospace; font-size: 11px; border: 2px solid #ff5722; background: white; padding: 2px; resize: none; overflow: auto;"
                />
              </foreignObject>
            {:else if element.label && element.label !== ''}
              {@const labelLines = (element.label || '').split('\n')}
              <text 
                x={midX} 
                y={midY - 4 - (labelLines.length - 1) * (element.labelSize || 11) * 0.5} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size={element.labelSize || 11} 
                text-anchor="middle" 
                dominant-baseline="baseline" 
                fill="#000000"
                pointer-events="none">
                {#each labelLines as labelLine, i}
                  <tspan x={midX} dy={i === 0 ? 0 : (element.labelSize || 11)}>{labelLine}</tspan>
                {/each}
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
            class:selected={selectedElements.some(el => el.id === element.id)}
            onmousedown={(e) => handleMouseDown(e, element)}
            onclick={(e) => handleElementClick(e, element)}
            ondblclick={(e) => handleDoubleClick(e, element)}>
            
            {#if editingText?.id === element.id}
              <!-- Inline editing with foreignObject -->
              <foreignObject 
                x={element.x} 
                y={element.y - (element.fontSize || 14)} 
                width="300" 
                height="150">
                <textarea 
                  id={`edit-${element.id}`}
                  bind:value={element.text}
                  onblur={finishEditingText}
                  onkeydown={(e) => {
                    if (e.key === 'Escape') finishEditingText();
                    e.stopPropagation();
                  }}
                  class="inline-text-edit"
                  style="width: 100%; height: 100%; font-family: ui-monospace, 'Courier New', monospace; font-size: {element.fontSize || 14}px; border: 2px solid #ff5722; background: white; padding: 4px; resize: none; overflow: auto;"
                />
              </foreignObject>
            {:else}
              {@const textLines = (element.text || '').split('\n')}
              <text 
                x={element.x} 
                y={element.y} 
                font-family="ui-monospace, 'Courier New', monospace" 
                font-size={element.fontSize || 14} 
                fill="#000000">
                {#each textLines as textLine, i}
                  <tspan x={element.x} dy={i === 0 ? 0 : (element.fontSize || 14)}>{textLine}</tspan>
                {/each}
              </text>
            {/if}
          </g>
        {/if}
      {/each}
      
      <!-- Selection rectangle while drawing - on top of everything -->
      {#if isDrawingSelection && selectionRect}
        {@const minX = Math.min(selectionRect.startX, selectionRect.endX)}
        {@const minY = Math.min(selectionRect.startY, selectionRect.endY)}
        {@const width = Math.abs(selectionRect.endX - selectionRect.startX)}
        {@const height = Math.abs(selectionRect.endY - selectionRect.startY)}
        <rect 
          x={minX} 
          y={minY} 
          width={width} 
          height={height} 
          fill="none" 
          stroke="#ff5722" 
          stroke-width="2"
          stroke-dasharray="5,5"
          pointer-events="none" />
      {/if}
      </g>
    </svg>
    
    <div class="canvas-hint" class:fade-out={!showHint}>
      {#if tool === 'select'}
        Click: select • Ctrl/Cmd+Click: multi-select • Hold M + Drag: area select • Drag: move • Arrow keys: nudge 1px • Ctrl/Cmd+C: copy • Ctrl/Cmd+V: paste • Ctrl/Cmd+Z: undo • Hold S: snap to grid (move/resize) • Hold Shift: box→square / line→angle-snap • Double-click: edit text
      {:else if tool === 'box'}
        Click anywhere to add a box • Hold Shift while resizing to force square • Hold S while dragging/resizing to snap to grid • When editing: Enter creates line break
      {:else if tool === 'circle'}
        Click anywhere to add a circle • Hold S while dragging/resizing to snap to grid • When editing: Enter creates line break
      {:else if tool === 'text'}
        Click anywhere to add text • Hold S while dragging to snap to grid • When editing: Enter creates line break
      {:else if tool === 'line'}
        Click anywhere to add a line (use Properties panel to add circle/arrow) • Hold S while dragging endpoints to snap to grid • When editing label: Enter creates line break
      {/if}
    </div>
  </div>
  
  {#if selectedElement}
    <div class="properties-panel">
      <h3>Properties</h3>
      
      {#if selectedElement.text !== undefined}
        <div class="inline-edit-hint">
          💡 Double-click to edit text inline. Press Enter for line breaks, Esc to finish.
        </div>
      {/if}
      
      {#if selectedElement.type === 'box'}
        <label class="checkbox">
          <input 
            type="checkbox" 
            bind:checked={selectedElement.locked}
          />
          Locked (prevents moving/resizing)
        </label>
        
        <label>
          Width:
          <input 
            type="number" 
            bind:value={selectedElement.width}
            min="20"
            max="500"
            onchange={() => selectedElement.width = Math.max(20, selectedElement.width)}
          />
        </label>
        
        <label>
          Height:
          <input 
            type="number" 
            bind:value={selectedElement.height}
            min="20"
            max="300"
            onchange={() => selectedElement.height = Math.max(20, selectedElement.height)}
          />
        </label>
        
        <label>
          Text Size:
          <input 
            type="number" 
            bind:value={selectedElement.fontSize}
            min="8"
            max="48"
          />
        </label>
        
        <label>
          Background:
          <div class="color-buttons-grid">
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#000000'}
              style="background: #000000; color: white;"
              onclick={() => selectedElement.bgColor = '#000000'}>
              Black
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#ffffff'}
              style="background: #ffffff; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.bgColor = '#ffffff'}>
              White
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#ff5722'}
              style="background: #ff5722; color: white;"
              onclick={() => selectedElement.bgColor = '#ff5722'}>
              Orange
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === 'none'}
              style="background: repeating-conic-gradient(#ccc 0% 25%, transparent 0% 50%) 50% / 10px 10px; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.bgColor = 'none'}>
              Clear
            </button>
          </div>
        </label>
        
        <label>
          Text Color:
          <div class="color-buttons">
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#000000'}
              style="background: #000000; color: white;"
              onclick={() => selectedElement.textColor = '#000000'}>
              Black
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#ffffff'}
              style="background: #ffffff; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.textColor = '#ffffff'}>
              White
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#ff5722'}
              style="background: #ff5722; color: white;"
              onclick={() => selectedElement.textColor = '#ff5722'}>
              Orange
            </button>
          </div>
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
      
      {#if selectedElement.type === 'circle'}
        <label class="checkbox">
          <input 
            type="checkbox" 
            bind:checked={selectedElement.locked}
          />
          Locked (prevents moving/resizing)
        </label>
        
        <label>
          Radius:
          <input 
            type="number" 
            bind:value={selectedElement.radius}
            min="10"
            max="200"
            onchange={() => selectedElement.radius = Math.max(10, selectedElement.radius)}
          />
        </label>
        
        <label>
          Text Size:
          <input 
            type="number" 
            bind:value={selectedElement.fontSize}
            min="8"
            max="48"
          />
        </label>
        
        <label>
          Background:
          <div class="color-buttons-grid">
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#000000'}
              style="background: #000000; color: white;"
              onclick={() => selectedElement.bgColor = '#000000'}>
              Black
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#ffffff'}
              style="background: #ffffff; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.bgColor = '#ffffff'}>
              White
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === '#ff5722'}
              style="background: #ff5722; color: white;"
              onclick={() => selectedElement.bgColor = '#ff5722'}>
              Orange
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.bgColor === 'none'}
              style="background: repeating-conic-gradient(#ccc 0% 25%, transparent 0% 50%) 50% / 10px 10px; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.bgColor = 'none'}>
              Clear
            </button>
          </div>
        </label>
        
        <label>
          Text Color:
          <div class="color-buttons">
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#000000'}
              style="background: #000000; color: white;"
              onclick={() => selectedElement.textColor = '#000000'}>
              Black
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#ffffff'}
              style="background: #ffffff; color: black; border: 1px solid #000;"
              onclick={() => selectedElement.textColor = '#ffffff'}>
              White
            </button>
            <button 
              class="color-btn"
              class:active={selectedElement.textColor === '#ff5722'}
              style="background: #ff5722; color: white;"
              onclick={() => selectedElement.textColor = '#ff5722'}>
              Orange
            </button>
          </div>
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
        <label class="checkbox">
          <input 
            type="checkbox" 
            bind:checked={selectedElement.locked}
          />
          Locked (prevents moving)
        </label>
        
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
        <label class="checkbox">
          <input 
            type="checkbox" 
            bind:checked={selectedElement.locked}
          />
          Locked (prevents moving/resizing)
        </label>
        
        <div class="inline-edit-hint">
          💡 Double-click line to add label. Press Enter for line breaks, Esc to finish.
        </div>
        
        <label>
          Label Size:
          <input 
            type="number" 
            bind:value={selectedElement.labelSize}
            min="8"
            max="24"
            placeholder="11"
          />
        </label>
        
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

<!-- About Button -->
<button 
  class="about-btn" 
  onclick={() => showAboutModal = !showAboutModal}
  title="About">
</button>

<!-- About Modal -->
{#if showAboutModal}
  <div class="modal-overlay" onclick={() => showAboutModal = false}>
    <div class="modal-content" onclick={(e) => e.stopPropagation()}>
      <button class="modal-close" onclick={() => showAboutModal = false}>×</button>
      <p>Made by <strong>Stephan Schulz</strong></p>
      <a href="https://github.com/stephanschulz/Planwerk/" target="_blank" rel="noopener noreferrer">
        View on GitHub →
      </a>
    </div>
  </div>
{/if}

<style>
  .builder-container {
    display: flex;
    gap: 0;
    height: 100%;
    width: 100%;
    background: #ffffff;
    border: 2px solid #000000;
    box-sizing: border-box;
  }
  
  .left-panel {
    display: flex;
    flex-direction: column;
    height: 100%;
  }
  
  .toolbar {
    width: 220px;
    height: 100%;
    background: #ffffff;
    border-right: 2px solid #000000;
    padding: 16px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 20px;
    box-sizing: border-box;
  }
  
  .properties-panel {
    width: 220px;
    height: 100%;
    background: #ffffff;
    border-left: 2px solid #000000;
    padding: 16px;
    overflow-y: auto;
    box-sizing: border-box;
  }
  
  .properties-panel h3 {
    font-size: 12px;
    font-weight: 400;
    margin: 0 0 16px 0;
    color: #666666;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-family: ui-monospace, 'Courier New', monospace;
  }
  
  .properties-panel label {
    display: block;
    margin-bottom: 12px;
    font-size: 11px;
    color: #333333;
    font-family: ui-monospace, 'Courier New', monospace;
  }
  
  .properties-panel input[type="text"],
  .properties-panel input[type="number"] {
    width: 100%;
    padding: 6px 8px;
    margin-top: 4px;
    border: 1px solid #000000;
    background: #ffffff;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
  }
  
  .properties-panel input[type="text"]:focus,
  .properties-panel input[type="number"]:focus {
    outline: 2px solid #ff5722;
    outline-offset: 0;
  }
  
  .properties-panel .checkbox {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .properties-panel .checkbox input {
    width: auto;
    margin: 0;
  }
  
  .color-buttons {
    display: flex;
    gap: 4px;
    margin-top: 4px;
  }
  
  .color-buttons-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 4px;
    margin-top: 4px;
  }
  
  .color-btn {
    flex: 1;
    padding: 6px 4px;
    cursor: pointer;
    border: 2px solid transparent;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 9px;
    font-weight: 400;
    transition: all 0.2s ease;
    min-height: 30px;
  }
  
  .color-buttons-grid .color-btn {
    flex: none;
  }
  
  .color-btn:hover {
    opacity: 0.8;
  }
  
  .color-btn.active {
    border-color: #ff5722;
    box-shadow: 0 0 0 2px #ff5722;
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
  
  .tool-grid {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 6px;
  }
  
  .tool-btn {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 4px;
    padding: 8px 4px;
    background: #ffffff;
    border: 2px solid #000000;
    cursor: pointer;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 10px;
    font-weight: 400;
    color: #000000;
    transition: all 0.2s ease;
  }
  
  .tool-btn span {
    display: block;
    white-space: nowrap;
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
  
  .button-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
    margin-bottom: 8px;
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
  
  .button-row .action-btn {
    margin-bottom: 0;
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
  
  .grid-size-input {
    width: 60px;
    padding: 4px 6px;
    margin-top: 4px;
    border: 1px solid #000000;
    background: #ffffff;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 11px;
  }
  
  .grid-size-input:focus {
    outline: 2px solid #ff5722;
    outline-offset: 0;
  }
  
  .grid-hint {
    font-size: 9px;
    color: #666666;
    margin-top: 8px;
    line-height: 1.3;
    font-style: italic;
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
  
  .canvas.space-pressed {
    cursor: grab !important;
  }
  
  .canvas.panning {
    cursor: grabbing !important;
  }
  
  .canvas.m-key-pressed {
    cursor: crosshair !important;
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
  
  .element.selected circle {
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
    opacity: 1;
    transition: opacity 0.5s ease-in-out;
  }
  
  .canvas-hint.fade-out {
    opacity: 0;
  }
  
  .about-btn {
    position: fixed;
    bottom: 20px;
    right: 20px;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background: #ff5722;
    border: 1px solid #000000;
    cursor: pointer;
    padding: 0;
    z-index: 1000;
    transition: all 0.2s;
  }
  
  .about-btn:hover {
    transform: scale(1.5);
  }
  
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;
  }
  
  .modal-content {
    background: white;
    padding: 32px;
    border: 2px solid #000000;
    border-radius: 8px;
    max-width: 400px;
    width: 90%;
    position: relative;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  }
  
  .modal-content p {
    margin: 0 0 16px 0;
    font-size: 16px;
    color: #333333;
  }
  
  .modal-content a {
    display: inline-block;
    color: #000000;
    text-decoration: none;
    font-weight: bold;
    padding: 8px 16px;
    border: 2px solid #000000;
    border-radius: 4px;
    transition: all 0.2s;
  }
  
  .modal-content a:hover {
    background: #000000;
    color: white;
  }
  
  .modal-close {
    position: absolute;
    top: 8px;
    right: 8px;
    background: none;
    border: none;
    font-size: 32px;
    cursor: pointer;
    color: #666666;
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    line-height: 1;
    transition: color 0.2s;
  }
  
  .modal-close:hover {
    color: #000000;
  }
</style>

