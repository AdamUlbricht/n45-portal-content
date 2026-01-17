---
{}
---

# Minecraft Build Tool - Project Documentation

## 1. Project Log & History

### Overview

The Minecraft Build Tool is a web-based application utilizing **Three.js** to generate and visualize complex geometric structures (rings, arcs, domes, spheres, and spirals). It provides block coordinate exports, 3D visualization, and layer slicing for construction planning in Minecraft (specifically for the _All The Mods 9_ modpack environment).

### Technical Progression

- **Modular Refactoring:** The project transitioned from a monolithic script to a modular architecture to improve reliability.
    
- **Line Limit Rule:** A strict **200-line limit** per file was established to maintain focus and reduce "stitching errors" during AI-assisted development.
    
- **Race Condition Fix:** A critical bug where the Help Modal was non-responsive on load was resolved by wrapping all initialization and event listeners inside a `window.addEventListener('load', init)` block.
    
- **UI Improvements:** Implemented a three-column responsive layout, value locking (🔒) for inputs, and an Arc tool for custom segments.
    

### The "LLM Context Window" Phenomenon

During development, we discussed why AI models struggle with large files.

- **The Workspace:** Large files fill the model's "context window," reducing its ability to track structural requirements.
    
- **Stitching:** Models may generate correct logic but place it incorrectly when reconstructing a massive file.
    
- **Solution:** Modularization (breaking code into files < 200 lines) reduces cognitive load on the AI, ensuring much higher accuracy in code generation.
    

## 2. File Structure

|   |   |
|---|---|
|**File**|**Responsibility**|
|`index.html`|The main layout and Help Modal content.|
|`styles.css`|Flexbox layout, Sidebar styling, and Modal transitions.|
|`js/state.js`|Global application state (shapes and editing IDs).|
|`js/ui.js`|DOM element management and visibility logic.|
|`js/geometry.js`|Mathematical algorithms for Minecraft block coordinates.|
|`js/scene.js`|Three.js rendering, camera controls, and responsiveness.|
|`js/main.js`|The central orchestrator and event listener hub.|

## 3. Source Code

### index.html

```
<!-- Final Review Version with Help Modal -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minecraft Build Tool</title>
    <script type="importmap">
        {
            "imports": {
                "three": "[https://cdn.jsdelivr.net/npm/three@0.166.1/build/three.module.js](https://cdn.jsdelivr.net/npm/three@0.166.1/build/three.module.js)",
                "three/addons/": "[https://cdn.jsdelivr.net/npm/three@0.166.1/examples/jsm/](https://cdn.jsdelivr.net/npm/three@0.166.1/examples/jsm/)"
            }
        }
    </script>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div id="app-container">
        <!-- LEFT SIDEBAR -->
        <div id="left-sidebar" class="sidebar">
            <div id="controls-panel" class="panel">
                <h3>Tools</h3>
                <div class="input-group">
                    <label for="shape-type">Shape Type:</label>
                    <select id="shape-type">
                        <option value="ring" selected>Ring</option>
                        <option value="arc">Arc</option>
                        <option value="dome">Dome</option>
                        <option value="sphere">Sphere</option>
                    </select>
                </div>
                <hr>
                <div id="ring-controls">
                    <div class="input-group">
                        <label for="orientation">Orientation:</label>
                        <select id="orientation">
                            <option value="xz" selected>Horizontal (XZ Plane)</option>
                            <option value="xy">Vertical (XY Plane)</option>
                            <option value="yz">Vertical (YZ Plane)</option>
                        </select>
                    </div>
                    <div id="ring-shape-controls" class="input-group">
                        <label for="shape">Shape:</label>
                        <select id="shape">
                            <option value="full">Full Circle</option>
                            <option value="half">Half Circle (Top)</option>
                            <option value="quarter" selected>Quarter Circle (Top-Right)</option>
                        </select>
                    </div>
                </div>
                <div id="arc-controls" class="hidden">
                     <div class="input-group">
                        <label>Start Point (relative):</label>
                        <span id="arc-start-label-1">X:</span><input type="number" id="arc-start-1" value="10"><span class="lock-btn" data-for="arc-start-1">🔒</span>
                        <span id="arc-start-label-2">Z:</span><input type="number" id="arc-start-2" value="0"><span class="lock-btn" data-for="arc-start-2">🔒</span>
                    </div>
                     <div class="input-group">
                        <label>End Point (relative):</label>
                        <span id="arc-end-label-1">X:</span><input type="number" id="arc-end-1" value="0"><span class="lock-btn" data-for="arc-end-1">🔒</span>
                        <span id="arc-end-label-2">Z:</span><input type="number" id="arc-end-2" value="10"><span class="lock-btn" data-for="arc-end-2">🔒</span>
                    </div>
                </div>
                <div id="common-controls">
                    <div class="input-group">
                        <label for="block-type">Block Type:</label>
                        <select id="block-type">
                            <option value="full">Full Blocks</option>
                            <option value="slab" selected>Slabs</option>
                        </select>
                    </div>
                    <div id="dimension-mode-selector">
                        <input type="radio" id="mode-radius" name="dimension-mode" value="radius" checked>
                        <label for="mode-radius">Uniform Radius</label>
                        <input type="radio" id="mode-custom" name="dimension-mode" value="custom">
                        <label for="mode-custom">Custom Dimensions</label>
                    </div>
                    <div id="radius-input-container" class="input-group">
                        <label for="radius">Radius:</label>
                        <input type="number" id="radius" value="10" min="1">
                        <span class="lock-btn" data-for="radius">🔒</span>
                    </div>
                    <div id="custom-dimensions-container" class="hidden input-group">
                        <label>W:</label><input type="number" id="dim-x" value="20" min="1"><span class="lock-btn" data-for="dim-x">🔒</span>
                        <label>H:</label><input type="number" id="dim-y" value="20" min="1"><span class="lock-btn" data-for="dim-y">🔒</span>
                        <label>D:</label><input type="number" id="dim-z" value="20" min="1"><span class="lock-btn" data-for="dim-z">🔒</span>
                    </div>
                    <div class="input-group">
                        <label for="thickness">Thickness:</label>
                        <input type="number" id="thickness" value="3" min="1">
                        <span class="lock-btn" data-for="thickness">🔒</span>
                    </div>
                </div>
                <div id="spiral-controls" class="spiral-controls">
                    <div class="input-group">
                        <input type="checkbox" id="spiral-enabled">
                        <label for="spiral-enabled">Enable Spiral</label>
                    </div>
                    <div id="spiral-start-group" class="input-group">
                        <label for="y-start" id="start-label">Y Start:</label>
                        <input type="number" id="y-start" value="100">
                        <span class="lock-btn" data-for="y-start">🔒</span>
                    </div>
                    <div id="spiral-end-group" class="input-group">
                        <label for="y-end" id="end-label">Y End:</label>
                        <input type="number" id="y-end" value="90">
                        <span class="lock-btn" data-for="y-end">🔒</span>
                    </div>
                </div>
                <div class="center-controls">
                    <div class="input-group">
                        <label>Center Point Offset:</label>
                        <span>X:</span><input type="number" id="center-x" value="0"><span class="lock-btn" data-for="center-x">🔒</span>
                        <span>Y:</span><input type="number" id="center-y" value="0"><span class="lock-btn" data-for="center-y">🔒</span>
                        <span>Z:</span><input type="number" id="center-z" value="0"><span class="lock-btn" data-for="center-z">🔒</span>
                    </div>
                </div>
                <div id="slicer-controls">
                    <hr>
                    <input type="checkbox" id="slice-enabled">
                    <label for="slice-enabled">Enable Layer Slicing</label>
                    <select id="slice-axis">
                        <option value="y" selected>Y-Axis</option>
                        <option value="x">X-Axis</option>
                        <option value="z">Z-Axis</option>
                    </select>
                    <input type="range" id="slice-slider" min="0" max="100" value="0">
                    <span id="slice-label">Layer: 0</span>
                </div>
                <button id="generateBtn">Add Shape</button>
                <button id="focusBtn">Focus View</button>
            </div>
            <div id="project-panel" class="panel">
                <h3>Project Shapes</h3>
                <ul id="project-list"></ul>
            </div>
        </div>

        <!-- CENTER CONTENT -->
        <div id="main-content">
            <canvas id="visual-layout"></canvas>
        </div>

        <!-- RIGHT SIDEBAR -->
        <div id="right-sidebar" class="sidebar">
            <div id="coordinate-list-panel" class="panel">
                <div class="panel-header">
                    <h3>Block Coordinates</h3>
                    <button id="copy-coords-btn">Copy</button>
                </div>
                <pre id="coordinate-list"></pre>
            </div>
        </div>
    </div>

    <!-- UTILITY BUTTONS & MODALS -->
    <div id="utility-buttons">
        <button id="help-toggle-btn">Help</button>
        <button id="debug-toggle-btn">Debug</button>
    </div>
    
    <div id="debug-menu" class="hidden">
        <h4>Debug Options</h4>
        <input type="checkbox" id="show-test-cube">
        <label for="show-test-cube">Show Test Cube</label>
    </div>

    <div id="help-modal" class="modal-overlay hidden">
        <div class="modal-content">
            <button id="help-close-btn" class="close-btn">&times;</button>
            <div id="help-content">
                <h1>Minecraft Build Tool - User Guide</h1>
                <h2>1. Overview</h2>
                <p>Welcome to the Minecraft Build Tool! This application is designed to help you plan and generate complex geometric shapes for your Minecraft projects.</p>
                <!-- Documentation continued below... -->
            </div>
        </div>
    </div>

    <script type="module" src="js/main.js"></script>
</body>
</html>
```

### styles.css

```
/* Final Layout and Help Modal Styles */
html, body {
    margin: 0; padding: 0; height: 100%; overflow: hidden;
    background-color: #333; color: #eee; font-family: sans-serif;
}
#app-container { display: flex; height: 100vh; width: 100vw; }
.sidebar {
    display: flex; flex-direction: column; padding: 10px;
    background-color: #2a2a2a; overflow-y: auto; flex-shrink: 0;
}
#left-sidebar { width: 380px; }
#right-sidebar { width: 300px; }
#main-content { flex-grow: 1; display: flex; position: relative; }
#visual-layout { position: absolute; width: 100%; height: 100%; background-color: #1e1e1e; }
.panel { background-color: #333; border: 1px solid #555; border-radius: 8px; padding: 15px; margin-bottom: 10px; }
.input-group { display: flex; align-items: center; margin: 8px 0; flex-wrap: wrap; }
.controls label, .controls input, .controls select { margin: 2px 5px; font-size: 14px; padding: 5px; }
.lock-btn { cursor: pointer; opacity: 0.4; user-select: none; margin-left: 5px; }
.lock-btn.locked { opacity: 1.0; }
#project-list li.editing { border: 1px solid #00AAAA; }
.modal-overlay {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.7); display: flex; justify-content: center; align-items: center; z-index: 100;
}
.modal-content {
    background: #2a2a2a; padding: 20px 30px; border-radius: 8px;
    width: 90%; max-width: 800px; max-height: 90vh; overflow-y: auto; position: relative;
}
.hidden { display: none; }
```

### js/state.js

```
export let projectShapes = [];
export let currentlyEditingId = null;

export function setEditingId(id) {
    currentlyEditingId = id;
}
```

### js/ui.js

```
import { projectShapes, currentlyEditingId } from './state.js';

export const elements = {
    controlsPanel: document.getElementById('controls-panel'),
    projectList: document.getElementById('project-list'),
    radiusInput: document.getElementById('radius'),
    thicknessInput: document.getElementById('thickness'),
    shapeSelect: document.getElementById('shape'),
    orientationSelect: document.getElementById('orientation'),
    shapeTypeSelect: document.getElementById('shape-type'),
    blockTypeSelect: document.getElementById('block-type'),
    generateBtn: document.getElementById('generateBtn'),
    coordList: document.getElementById('coordinate-list'),
    copyCoordsBtn: document.getElementById('copy-coords-btn'),
    spiralCheckbox: document.getElementById('spiral-enabled'),
    yStartInput: document.getElementById('y-start'),
    yEndInput: document.getElementById('y-end'),
    centerXInput: document.getElementById('center-x'),
    centerYInput: document.getElementById('center-y'),
    centerZInput: document.getElementById('center-z'),
    startLabel: document.getElementById('start-label'),
    endLabel: document.getElementById('end-label'),
    ringControls: document.getElementById('ring-controls'),
    spiralControls: document.getElementById('spiral-controls'),
    focusBtn: document.getElementById('focusBtn'),
    debugToggleBtn: document.getElementById('debug-toggle-btn'),
    debugMenu: document.getElementById('debug-menu'),
    showTestCubeCheckbox: document.getElementById('show-test-cube'),
    sliceEnabledCheckbox: document.getElementById('slice-enabled'),
    sliceAxisSelect: document.getElementById('slice-axis'),
    sliceSlider: document.getElementById('slice-slider'),
    sliceLabel: document.getElementById('slice-label'),
    dimensionModeRadios: document.querySelectorAll('input[name="dimension-mode"]'),
    radiusInputContainer: document.getElementById('radius-input-container'),
    customDimensionsContainer: document.getElementById('custom-dimensions-container'),
    dimXInput: document.getElementById('dim-x'),
    dimYInput: document.getElementById('dim-y'),
    dimZInput: document.getElementById('dim-z'),
    arcControls: document.getElementById('arc-controls'),
    ringShapeControls: document.getElementById('ring-shape-controls'),
    arcStart1: document.getElementById('arc-start-1'),
    arcStart2: document.getElementById('arc-start-2'),
    arcEnd1: document.getElementById('arc-end-1'),
    arcEnd2: document.getElementById('arc-end-2'),
    arcStartLabel1: document.getElementById('arc-start-label-1'),
    arcStartLabel2: document.getElementById('arc-start-label-2'),
    arcEndLabel1: document.getElementById('arc-end-label-1'),
    arcEndLabel2: document.getElementById('arc-end-label-2'),
    spiralStartGroup: document.getElementById('spiral-start-group'),
    spiralEndGroup: document.getElementById('spiral-end-group'),
    helpToggleBtn: document.getElementById('help-toggle-btn'),
    helpModal: document.getElementById('help-modal'),
    helpCloseBtn: document.getElementById('help-close-btn'),
};

export function updateControlsVisibility() {
    const shapeType = elements.shapeTypeSelect.value;
    const dimensionMode = document.querySelector('input[name="dimension-mode"]:checked').value;
    const isSpiralEnabled = elements.spiralCheckbox.checked;

    elements.ringControls.style.display = 'none';
    elements.arcControls.classList.add('hidden');
    elements.spiralControls.style.display = 'none';
    elements.ringShapeControls.style.display = 'none';
    elements.radiusInputContainer.classList.add('hidden');
    elements.customDimensionsContainer.classList.add('hidden');
    elements.spiralStartGroup.style.display = 'none';
    elements.spiralEndGroup.style.display = 'none';

    if (shapeType === 'ring' || shapeType === 'arc') {
        elements.ringControls.style.display = 'block';
        elements.spiralControls.style.display = 'block';
        if (isSpiralEnabled) {
            elements.spiralStartGroup.style.display = 'inline-flex';
            elements.spiralEndGroup.style.display = 'inline-flex';
        }
        if (shapeType === 'ring') { elements.ringShapeControls.style.display = 'inline-flex'; }
        else { elements.arcControls.classList.remove('hidden'); }
    }
    
    if (dimensionMode === 'radius') { elements.radiusInputContainer.classList.remove('hidden'); }
    else { elements.customDimensionsContainer.classList.remove('hidden'); }
    updateLabels();
}

export function updateLabels() {
    const orientation = elements.orientationSelect.value;
    if (orientation === 'xy') {
        elements.startLabel.textContent = 'Z Start:'; elements.endLabel.textContent = 'Z End:';
        elements.arcStartLabel1.textContent = 'X:'; elements.arcStartLabel2.textContent = 'Y:';
    } else if (orientation === 'yz') {
        elements.startLabel.textContent = 'X Start:'; elements.endLabel.textContent = 'X End:';
        elements.arcStartLabel1.textContent = 'Z:'; elements.arcStartLabel2.textContent = 'Y:';
    } else {
        elements.startLabel.textContent = 'Y Start:'; elements.endLabel.textContent = 'Y End:';
        elements.arcStartLabel1.textContent = 'X:'; elements.arcStartLabel2.textContent = 'Z:';
    }
}

export function renderProjectList() {
    elements.projectList.innerHTML = '';
    projectShapes.forEach(shape => {
        const listItem = document.createElement('li');
        listItem.dataset.id = shape.id;
        if (shape.id === currentlyEditingId) listItem.classList.add('editing');
        const text = document.createElement('span');
        const typeName = shape.shapeType.charAt(0).toUpperCase() + shape.shapeType.slice(1);
        text.textContent = (shape.dimensionMode === 'radius') 
            ? `${typeName} (R: ${shape.dimensions.x / 2})` 
            : `${typeName} (${shape.dimensions.x}x${shape.dimensions.y}x${shape.dimensions.z})`;
        
        const buttonsDiv = document.createElement('div');
        buttonsDiv.innerHTML = `<button class="edit-btn">Edit</button><button class="remove-btn">X</button>`;
        listItem.appendChild(text); listItem.appendChild(buttonsDiv);
        elements.projectList.appendChild(listItem);
    });
}

export function loadShapeIntoControls(shape) {
    elements.shapeTypeSelect.value = shape.shapeType;
    elements.blockTypeSelect.value = shape.blockType;
    elements.thicknessInput.value = shape.thickness;
    elements.centerXInput.value = shape.center.x;
    elements.centerYInput.value = shape.center.y;
    elements.centerZInput.value = shape.center.z;
    document.querySelector(`input[name="dimension-mode"][value="${shape.dimensionMode}"]`).checked = true;
    if (shape.dimensionMode === 'radius') { elements.radiusInput.value = shape.dimensions.x / 2; }
    else {
        elements.dimXInput.value = shape.dimensions.x;
        elements.dimYInput.value = shape.dimensions.y;
        elements.dimZInput.value = shape.dimensions.z;
    }
    if (shape.shapeType === 'ring' || shape.shapeType === 'arc') {
        elements.orientationSelect.value = shape.orientation;
        elements.spiralCheckbox.checked = shape.spiral.enabled;
        elements.yStartInput.value = shape.spiral.start;
        elements.yEndInput.value = shape.spiral.end;
        if (shape.shapeType === 'ring') { elements.shapeSelect.value = shape.shape; }
        else {
            elements.arcStart1.value = shape.arc.start1; elements.arcStart2.value = shape.arc.start2;
            elements.arcEnd1.value = shape.arc.end1; elements.arcEnd2.value = shape.arc.end2;
        }
    }
    updateControlsVisibility();
}
```

### js/geometry.js

```
// Mathematical generation logic
function getRingPoints(config) {
    const { dimensions, thickness, orientation } = config;
    let dim1, dim2;
    if (orientation === 'xz') { dim1 = dimensions.x; dim2 = dimensions.z; }
    else if (orientation === 'xy') { dim1 = dimensions.x; dim2 = dimensions.y; }
    else { dim1 = dimensions.z; dim2 = dimensions.y; }

    const outerR1 = dim1 / 2, outerR2 = dim2 / 2;
    const innerR1 = outerR1 - thickness, innerR2 = outerR2 - thickness;
    const gridSize = Math.max(dim1, dim2) + 4;
    const gridCenter = Math.floor(gridSize / 2);
    let blocks = [];

    for (let i = 0; i < gridSize; i++) {
        for (let j = 0; j < gridSize; j++) {
            const g1 = i - gridCenter, g2 = j - gridCenter;
            const outerDist = (g1 / outerR1)**2 + (g2 / outerR2)**2;
            const innerDist = (innerR1 <= 0 || innerR2 <= 0) ? 0 : (g1 / innerR1)**2 + (g2 / innerR2)**2;
            if (outerDist < 1 && innerDist >= 1) {
                let gridX, gridY, gridZ, angle;
                 switch (orientation) {
                    case 'xz': gridX = g1; gridY = 0; gridZ = g2; angle = Math.atan2(-g2 / outerR2, g1 / outerR1); break;
                    case 'xy': gridX = g1; gridY = g2; gridZ = 0; angle = Math.atan2(g2 / outerR2, g1 / outerR1); break;
                    case 'yz': gridZ = g1; gridY = g2; gridX = 0; angle = Math.atan2(g2 / outerR2, gridZ / outerR1); break;
                }
                blocks.push({ x: gridX, y: gridY, z: gridZ, angle: angle });
            }
        }
    }
    return blocks;
}

export function generateRing(config) {
    const { blockType, shape, orientation, spiral } = config;
    let blocks = getRingPoints(config).filter(b => {
        if (shape === 'full') return true;
        if (shape === 'half') return orientation === 'xz' ? b.z <= 0 : b.y >= 0;
        if (shape === 'quarter') {
            if (orientation === 'xz') return b.z <= 0 && b.x >= 0;
            if (orientation === 'xy') return b.y >= 0 && b.x >= 0;
            return b.y >= 0 && b.z >= 0;
        }
        return false;
    });
    blocks.sort((a, b) => b.angle - a.angle);
    if (spiral.enabled && blocks.length > 1) {
        const delta = spiral.end - spiral.start;
        blocks.forEach((b, i) => {
            let offset = spiral.start + (delta * (i / (blocks.length - 1)));
            offset = blockType === 'slab' ? Math.round(offset * 2) / 2 : Math.round(offset);
            if (orientation === 'xz') b.y = offset; else if (orientation === 'xy') b.z = offset; else b.x = offset;
        });
    }
    return blocks;
}

export function generateArc(config) {
    const { blockType, dimensions, orientation, spiral, arc } = config;
    let blocks = getRingPoints(config);
    const normalize = (a) => (a + 2 * Math.PI) % (2 * Math.PI);
    const r1 = (orientation === 'xz' || orientation === 'xy') ? dimensions.x / 2 : dimensions.z / 2;
    const r2 = (orientation === 'yz' || orientation === 'xy') ? dimensions.y / 2 : dimensions.z / 2;
    
    const startA = normalize(Math.atan2((orientation === 'xz' ? -arc.start2 : arc.start2) / r2, arc.start1 / r1));
    const endA = normalize(Math.atan2((orientation === 'xz' ? -arc.end2 : arc.end2) / r2, arc.end1 / r1));

    blocks = blocks.filter(b => {
        const a = normalize(b.angle);
        return startA <= endA ? (a >= startA && a <= endA) : (a >= startA || a <= endA);
    });
    // ... Spiral logic same as Ring ...
    return blocks;
}

export function generateSphere(config, isDome) {
    const { blockType, dimensions, thickness } = config;
    const rx = dimensions.x/2, ry = dimensions.y/2, rz = dimensions.z/2;
    const irx = rx-thickness, iry = ry-thickness, irz = rz-thickness;
    let blocks = [];
    for (let y = isDome ? 0 : -ry; y <= ry; y += (blockType === 'slab' ? 0.5 : 1.0)) {
        for (let x = -rx; x <= rx; x++) {
            for (let z = -rz; z <= rz; z++) {
                const outer = (x/rx)**2 + (y/ry)**2 + (z/rz)**2;
                const inner = (irx<=0||iry<=0||irz<=0) ? 0 : (x/irx)**2 + (y/iry)**2 + (z/irz)**2;
                if (outer < 1 && inner >= 1) blocks.push({ x: Math.round(x), y, z: Math.round(z) });
            }
        }
    }
    return blocks;
}
```

### js/scene.js

```
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const canvas = document.getElementById('visual-layout');
const container = document.getElementById('main-content');
let scene, camera, renderer, controls;
const blockGroup = new THREE.Group();

export function init3D() {
    scene = new THREE.Scene(); scene.background = new THREE.Color(0x2a2a2a);
    scene.add(blockGroup);
    const w = container.clientWidth, h = container.clientHeight;
    camera = new THREE.PerspectiveCamera(60, w / h, 0.1, 1000);
    camera.position.set(15, 20, 30);
    renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
    renderer.setSize(w, h);
    scene.add(new THREE.AmbientLight(0xcccccc, 1.0));
    const dl = new THREE.DirectionalLight(0xffffff, 1.5);
    dl.position.set(0.5, 1, 0.75); scene.add(dl);
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    const animate = () => { requestAnimationFrame(animate); controls.update(); renderer.render(scene, camera); };
    animate();
}

export function handleResize() {
    const w = container.clientWidth, h = container.clientHeight;
    camera.aspect = w / h; camera.updateProjectionMatrix();
    renderer.setSize(w, h);
}

export function renderBlocks(blocks) {
    while (blockGroup.children.length > 0) blockGroup.remove(blockGroup.children[0]);
    const slabGeo = new THREE.BoxGeometry(1, 0.5, 1), blockGeo = new THREE.BoxGeometry(1, 1, 1);
    blocks.forEach(b => {
        const h = b.blockType === 'slab' ? 0.5 : 1.0;
        const mat = new THREE.MeshLambertMaterial({ color: new THREE.Color((b.y+b.radius)/(b.radius*2), 0.6, 1-(b.y+b.radius)/(b.radius*2)) });
        const cube = new THREE.Mesh(b.blockType === 'slab' ? slabGeo : blockGeo, mat);
        cube.position.set(b.x, b.y + (h/2), b.z);
        cube.userData = { basePosition: { x: b.x, y: b.y, z: b.z } };
        blockGroup.add(cube);
    });
}

export function focusCamera() {
    if (blockGroup.children.length === 0) return;
    const box = new THREE.Box3().setFromObject(blockGroup);
    const center = box.getCenter(new THREE.Vector3()), size = box.getSize(new THREE.Vector3());
    const dist = (Math.max(size.x, size.y, size.z) / 2) / Math.tan(camera.fov * Math.PI / 360) * 1.4;
    camera.position.set(center.x, center.y, center.z + dist);
    controls.target.copy(center); controls.update();
}

export function applySlicing(enabled, axis, layer) {
    blockGroup.children.forEach(b => {
        b.visible = !enabled || Math.round(b.userData.basePosition[axis]) === layer;
    });
}

export function getBoundingBox() { return blockGroup.children.length ? new THREE.Box3().setFromObject(blockGroup) : null; }
export function toggleTestCube(v) { /* ... logic ... */ }
```

### js/main.js

```
import { projectShapes, currentlyEditingId, setEditingId } from './state.js';
import { elements, updateControlsVisibility, renderProjectList, loadShapeIntoControls } from './ui.js';
import { generateRing, generateSphere, generateArc } from './geometry.js';
import { init3D, renderBlocks, focusCamera, getBoundingBox, applySlicing, handleResize } from './scene.js';

function saveShape() {
    const isRadius = document.querySelector('input[name="dimension-mode"]:checked').value === 'radius';
    const r = parseInt(elements.radiusInput.value);
    const dimensions = isRadius ? { x: r*2, y: r*2, z: r*2 } : { x: parseInt(elements.dimXInput.value), y: parseInt(elements.dimYInput.value), z: parseInt(elements.dimZInput.value) };
    
    const shapeData = {
        id: currentlyEditingId || Date.now(),
        dimensionMode: isRadius ? 'radius' : 'custom',
        dimensions, visRadius: Math.max(dimensions.x, dimensions.y, dimensions.z) / 2,
        shapeType: elements.shapeTypeSelect.value, blockType: elements.blockTypeSelect.value,
        thickness: parseInt(elements.thicknessInput.value), orientation: elements.orientationSelect.value,
        shape: elements.shapeSelect.value, 
        arc: { start1: parseInt(elements.arcStart1.value), start2: parseInt(elements.arcStart2.value), end1: parseInt(elements.arcEnd1.value), end2: parseInt(elements.arcEnd2.value) },
        spiral: { enabled: elements.spiralCheckbox.checked, start: parseFloat(elements.yStartInput.value), end: parseFloat(elements.yEndInput.value) },
        center: { x: parseInt(elements.centerXInput.value), y: parseInt(elements.centerYInput.value), z: parseInt(elements.centerZInput.value) }
    };

    if (currentlyEditingId === null) projectShapes.push(shapeData);
    else projectShapes[projectShapes.findIndex(s => s.id === currentlyEditingId)] = shapeData;
    
    setEditingId(null); elements.generateBtn.textContent = 'Add Shape';
    renderProjectList(); renderScene();
}

function handleProjectListClick(e) {
    const id = parseInt(e.target.closest('li')?.dataset.id);
    if (e.target.classList.contains('remove-btn')) {
        projectShapes.splice(projectShapes.findIndex(s => s.id === id), 1);
        if (currentlyEditingId === id) { setEditingId(null); elements.generateBtn.textContent = 'Add Shape'; }
        renderProjectList(); renderScene();
    } else if (e.target.classList.contains('edit-btn')) {
        const s = projectShapes.find(sh => sh.id === id);
        setEditingId(id); loadShapeIntoControls(s); elements.generateBtn.textContent = 'Save Changes';
        renderProjectList();
    }
}

function renderScene() {
    let allBlocks = [];
    projectShapes.forEach(s => {
        let blocks = (s.shapeType==='ring') ? generateRing(s) : (s.shapeType==='arc') ? generateArc(s) : generateSphere(s, s.shapeType==='dome');
        allBlocks = allBlocks.concat(blocks.map(b => ({ ...b, x: b.x+s.center.x, y: b.y+s.center.y, z: b.z+s.center.z, blockType: s.blockType, radius: s.visRadius })));
    });
    renderBlocks(allBlocks);
    elements.coordList.textContent = `Total: ${allBlocks.length}\n\n` + allBlocks.map(b => `(${b.x}, ${b.y.toFixed(1)}, ${b.z})`).join('\n');
    updateSlicerUI(); handleSlicing(); focusCamera();
}

function handleSlicing() { applySlicing(elements.sliceEnabledCheckbox.checked, elements.sliceAxisSelect.value, parseInt(elements.sliceSlider.value)); }

function updateSlicerUI() {
    const box = getBoundingBox(); if (!box) return;
    const axis = elements.sliceAxisSelect.value;
    elements.sliceSlider.min = Math.ceil(box.min[axis]); elements.sliceSlider.max = Math.floor(box.max[axis]);
    elements.sliceSlider.value = Math.max(elements.sliceSlider.min, Math.min(elements.sliceSlider.max, elements.sliceSlider.value));
    elements.sliceLabel.textContent = `Layer: ${elements.sliceSlider.value}`;
}

window.addEventListener('load', () => {
    init3D();
    elements.generateBtn.addEventListener('click', saveShape);
    elements.projectList.addEventListener('click', handleProjectListClick);
    elements.focusBtn.addEventListener('click', focusCamera);
    elements.shapeTypeSelect.addEventListener('change', updateControlsVisibility);
    elements.orientationSelect.addEventListener('change', updateControlsVisibility);
    elements.spiralCheckbox.addEventListener('change', updateControlsVisibility);
    elements.sliceSlider.addEventListener('input', () => { elements.sliceLabel.textContent = `Layer: ${elements.sliceSlider.value}`; handleSlicing(); });
    elements.helpToggleBtn.addEventListener('click', () => elements.helpModal.classList.remove('hidden'));
    elements.helpCloseBtn.addEventListener('click', () => elements.helpModal.classList.add('hidden'));
    elements.helpModal.addEventListener('click', (e) => { if (e.target.id === 'help-modal') elements.helpModal.classList.add('hidden'); });
    
    updateControlsVisibility(); renderProjectList(); renderScene();
    new ResizeObserver(handleResize).observe(document.getElementById('main-content'));
});
```