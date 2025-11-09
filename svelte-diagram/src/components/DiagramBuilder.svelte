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
      textColor: '#000000'
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
      textColor: '#000000'
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
      selectedElements = [];
      editingText = null;
      editingLineLabel = false;
      return;
    }
    
    const rect = canvasElement.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    if (tool === 'box') {
      elements = [...elements, createBox(x, y)];
    } else if (tool === 'circle') {
      elements = [...elements, createCircle(x, y)];
    } else if (tool === 'text') {
      elements = [...elements, createText(x, y)];
    } else if (tool === 'line') {
      elements = [...elements, createLine(x, y)];
    }
    
    tool = 'select';
  }
  
  function handleElementClick(e, element) {
    e.stopPropagation();
    
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
    
    e.stopPropagation();
    
    // If clicked element is not in selection, select it (unless multi-selecting)
    if (!selectedElements.find(el => el.id === element.id) && !(e.ctrlKey || e.metaKey)) {
      selectedElements = [element];
    }
    
    selectedElement = element;
    isDragging = true;
    
    const rect = canvasElement.getBoundingClientRect();
    
    // Store initial positions for all selected elements
    const elementsData = selectedElements.map(el => ({
      element: el,
      startX: el.x || el.x1,
      startY: el.y || el.y1,
      startX2: el.x2,
      startY2: el.y2
    }));
    
    dragStart = {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
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
      elementRadius: element.radius,
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
          let cornerX = dragStart.elementX + dragStart.elementWidth + dx;
          let cornerY = dragStart.elementY + dragStart.elementHeight + dy;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(cornerX, cornerY);
            cornerX = snapped.x;
            cornerY = snapped.y;
          }
          
          const newWidth = Math.max(50, cornerX - dragStart.elementX);
          const newHeight = Math.max(30, cornerY - dragStart.elementY);
          
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
          
          const newWidth = Math.max(50, dragStart.elementX + dragStart.elementWidth - cornerX);
          const newHeight = Math.max(30, cornerY - dragStart.elementY);
          
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
          
          const newWidth = Math.max(50, cornerX - dragStart.elementX);
          const newHeight = Math.max(30, dragStart.elementY + dragStart.elementHeight - cornerY);
          
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
          
          const newWidth = Math.max(50, dragStart.elementX + dragStart.elementWidth - cornerX);
          const newHeight = Math.max(30, dragStart.elementY + dragStart.elementHeight - cornerY);
          
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
        let newRadius = Math.max(20, distance);
        
        // Apply grid snapping if 's' key is held
        if (isSnapKeyPressed) {
          // Snap the handle position to grid and recalculate radius
          const snapPoint = snapToGrid(currentX, currentY);
          const snappedDistanceX = snapPoint.x - centerX;
          const snappedDistanceY = snapPoint.y - centerY;
          const snappedDistance = Math.sqrt(snappedDistanceX * snappedDistanceX + snappedDistanceY * snappedDistanceY);
          newRadius = Math.max(20, snappedDistance);
        }
        
        // Update circle to maintain center position
        selectedElement.radius = newRadius;
        selectedElement.x = centerX - newRadius;
        selectedElement.y = centerY - newRadius;
      } else if (selectedElement.type === 'line') {
        if (resizeHandle === 'start') {
          // Check for snap points
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id);
          let finalX = snapPoint ? snapPoint.x : currentX;
          let finalY = snapPoint ? snapPoint.y : currentY;
          
          // Apply grid snapping if 's' key is held
          if (isSnapKeyPressed) {
            const snapped = snapToGrid(finalX, finalY);
            finalX = snapped.x;
            finalY = snapped.y;
          }
          
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
          // Check for snap points
          const snapPoint = findSnapPoint(currentX, currentY, selectedElement.id);
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
    isDragging = false;
    isResizing = false;
    resizeHandle = null;
    dragStart = null;
  }
  
  function deleteSelected() {
    if (selectedElements.length > 0) {
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
    
    // Don't process any keys if we're editing text
    if (editingText || editingLineLabel) {
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
    window.addEventListener('keyup', handleKeyUp);
    loadSavedDesignsList();
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
      
      {#each elements as element (element.id)}
        {#if element.type === 'box'}
          <g 
            class="element box-element"
            class:selected={selectedElements.some(el => el.id === element.id)}
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
              fill={element.bgColor || '#ffffff'} 
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
            
            {#if element.shadow}
              <!-- Shadow -->
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
              stroke-dasharray={element.dashed ? "5,5" : "none"} />
            
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
              stroke-width="10" />
            
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
    </svg>
    
    <div class="canvas-hint">
      {#if tool === 'select'}
        Click: select • Ctrl/Cmd+Click: multi-select • Drag: move • Arrow keys: nudge 1px • Ctrl/Cmd+C: copy • Ctrl/Cmd+V: paste • Hold S: snap to grid (move/resize) • Hold Shift: box→square / line→angle-snap • Double-click: edit text
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
          <div class="color-buttons">
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
        <label>
          Radius:
          <input 
            type="number" 
            bind:value={selectedElement.radius}
            min="20"
            max="200"
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
          <div class="color-buttons">
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

<style>
  .builder-container {
    display: flex;
    gap: 0;
    height: 85vh;
    background: #ffffff;
    border: 2px solid #000000;
  }
  
  .left-panel {
    display: flex;
    flex-direction: column;
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
  
  .properties-panel {
    width: 220px;
    background: #ffffff;
    border-left: 2px solid #000000;
    padding: 16px;
    overflow-y: auto;
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
  
  .color-btn {
    flex: 1;
    padding: 6px 4px;
    cursor: pointer;
    border: 2px solid transparent;
    font-family: ui-monospace, 'Courier New', monospace;
    font-size: 9px;
    font-weight: 400;
    transition: all 0.2s ease;
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

