<script>
  let boxes = [];
  let nextId = 1;
  let resizingBox = null;
  let startX, startY, startWidth, startHeight;

  // Connector logic
  let connectors = [];
  let nextConnectorId = 1;
  let connecting = null; // { connectorId, stage: 0|1 }

  function addBox() {
    boxes = [
      ...boxes,
      {
        id: nextId++,
        x: 40,
        y: 40,
        width: 160,
        height: 160
      }
    ];
  }

  function addConnector() {
    connectors = [
      ...connectors,
      {
        id: nextConnectorId++,
        from: null,
        to: null
      }
    ];
    connecting = { connectorId: nextConnectorId - 1, stage: 0 };
  }

  function handleBoxClick(box) {
    if (connecting) {
      const connector = connectors.find(c => c.id === connecting.connectorId);
      if (connector) {
        if (connecting.stage === 0 && connector.from === null) {
          connector.from = box.id;
          connecting.stage = 1;
        } else if (connecting.stage === 1 && connector.to === null && box.id !== connector.from) {
          connector.to = box.id;
          connecting = null;
        }
        connectors = [...connectors];
      }
    }
  }

  function onDragStart(event, box) {
    event.dataTransfer.setData("text/plain", box.id);
    event.dataTransfer.effectAllowed = "move";
  }

  function onDragOver(event) {
    event.preventDefault();
  }

  function onDrop(event) {
    event.preventDefault();
    const id = Number(event.dataTransfer.getData("text/plain"));
    const rect = event.currentTarget.getBoundingClientRect();
    const box = boxes.find(b => b.id === id);
    if (box) {
      const x = event.clientX - rect.left - box.width / 2;
      const y = event.clientY - rect.top - box.height / 2;
      boxes = boxes.map(b =>
        b.id === id ? { ...b, x, y } : b
      );
    }
  }

  function onResizeMouseDown(event, box) {
    event.stopPropagation();
    resizingBox = box;
    startX = event.clientX;
    startY = event.clientY;
    startWidth = box.width;
    startHeight = box.height;
    window.addEventListener('mousemove', onResizeMouseMove);
    window.addEventListener('mouseup', onResizeMouseUp);
  }

  function onResizeMouseMove(event) {
    if (!resizingBox) return;
    const dx = event.clientX - startX;
    const dy = event.clientY - startY;
    resizingBox.width = Math.max(60, startWidth + dx);
    resizingBox.height = Math.max(60, startHeight + dy);
    boxes = [...boxes];
  }

  function onResizeMouseUp() {
    resizingBox = null;
    window.removeEventListener('mousemove', onResizeMouseMove);
    window.removeEventListener('mouseup', onResizeMouseUp);
  }

  function getBoxCenter(box) {
    return {
      x: box.x + box.width / 2,
      y: box.y + box.height / 2
    };
  }
</script>

<div class="container">
  <aside class="side-panel">
    <h2>Menu</h2>
    <button on:click={addBox}>Add</button>
    <button on:click={addConnector} style="margin-top:0.5rem;">Connector</button>
  </aside>
  <section
    class="main-panel"
    on:dragover={onDragOver}
    on:drop={onDrop}
  >
    <!-- SVG for connectors -->
    <svg class="connectors-svg" width="100%" height="100%">
      {#each connectors as connector (connector.id)}
        {#if connector.from !== null && connector.to !== null}
          {#key connector.id}
            {#if boxes.find(b => b.id === connector.from) && boxes.find(b => b.id === connector.to)}
              <line
                x1={getBoxCenter(boxes.find(b => b.id === connector.from)).x}
                y1={getBoxCenter(boxes.find(b => b.id === connector.from)).y}
                x2={getBoxCenter(boxes.find(b => b.id === connector.to)).x}
                y2={getBoxCenter(boxes.find(b => b.id === connector.to)).y}
                stroke="#1976d2"
                stroke-width="4"
                marker-end="url(#arrowhead)"
              />
            {/if}
          {/key}
        {/if}
      {/each}
      <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" fill="#1976d2" />
        </marker>
      </defs>
    </svg>
    {#each boxes as box (box.id)}
      <div
        class="draggable-box"
        draggable="true"
        on:dragstart={(e) => onDragStart(e, box)}
        style="left: {box.x}px; top: {box.y}px; width: {box.width}px; height: {box.height}px;"
        on:click={() => handleBoxClick(box)}
      >
        Box {box.id}
        <div
          class="resize-handle"
          on:mousedown={(e) => onResizeMouseDown(e, box)}
        ></div>
      </div>
    {/each}
    {#if connecting}
      <div class="connecting-indicator">
        {connecting.stage === 0
          ? "Select first box"
          : "Select second box"}
      </div>
    {/if}
  </section>
</div>

<style>
  .container {
    display: flex;
    height: 100vh;
  }
  .side-panel {
    width: 200px;
    background: #f4f4f4;
    padding: 1rem;
    border-right: 1px solid #ddd;
    box-sizing: border-box;
  }
  .main-panel {
    flex: 1;
    position: relative;
    background: #fff;
    overflow: hidden;
  }
  .connectors-svg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 1;
  }
  .draggable-box {
    position: absolute;
    background: #4f8cff;
    color: #fff;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 6px;
    cursor: grab;
    user-select: none;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    font-size: 1.2rem;
    transition: box-shadow 0.1s;
    z-index: 2;
    overflow: visible;
  }
  .draggable-box:active {
    box-shadow: 0 4px 16px rgba(0,0,0,0.16);
  }
  .resize-handle {
    position: absolute;
    right: 0;
    bottom: 0;
    width: 18px;
    height: 18px;
    background: #fff;
    border: 2px solid #1976d2;
    border-radius: 0 0 6px 0;
    cursor: se-resize;
    z-index: 3;
  }
  button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    width: 100%;
  }
  .connecting-indicator {
    position: absolute;
    left: 50%;
    top: 10px;
    transform: translateX(-50%);
    background: #1976d2;
    color: #fff;
    padding: 0.3rem 1rem;
    border-radius: 6px;
    z-index: 10;
    font-size: 1rem;
    pointer-events: none;
    opacity: 0.95;
  }
</style>
