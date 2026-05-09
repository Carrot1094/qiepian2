<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>3D蓝色积木 - 触屏适配 | 可调长宽高 | 六向穿透消除</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Segoe UI', 'PingFang SC', Roboto, 'Microsoft YaHei', sans-serif; touch-action: none; }
        .controls-panel {
            position: absolute;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: rgba(20, 25, 40, 0.85);
            backdrop-filter: blur(12px);
            border-radius: 20px;
            padding: 12px 20px;
            color: white;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 12px;
            align-items: center;
            z-index: 20;
            font-size: 14px;
            font-weight: 500;
            border: 1px solid rgba(255,255,255,0.2);
            box-shadow: 0 4px 20px rgba(0,0,0,0.4);
            pointer-events: auto;
        }
        .panel-group { display: flex; gap: 8px; background: rgba(0,0,0,0.3); padding: 5px 12px; border-radius: 40px; align-items: center; }
        .size-btn, .action-btn {
            background: #2c3e66; border: none; color: white; padding: 8px 16px; border-radius: 40px;
            cursor: pointer; font-weight: bold; transition: 0.2s; font-size: 14px; min-width: 56px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.3); touch-action: manipulation;
        }
        /* 增大触摸区域 */
        .size-btn, .action-btn, .dim-input, #apply-size {
            min-height: 44px;
        }
        .size-btn.active { background: #e67e22; box-shadow: 0 0 8px rgba(230,126,34,0.6); }
        .action-btn.reset-btn { background: #e67e22; }
        .action-btn.separate-btn { background: #27ae60; }
        .action-btn:hover, .size-btn:hover { filter: brightness(1.1); transform: translateY(-1px); }
        .divider { width: 1px; height: 32px; background: rgba(255,255,255,0.3); margin: 0 4px; }
        .info-badge { font-size: 11px; background: rgba(0,0,0,0.5); padding: 6px 12px; border-radius: 25px; letter-spacing: 0.5px; }
        .rule-badge { font-size: 11px; background: #1e3a5f; padding: 6px 14px; border-radius: 30px; font-weight: bold; }
        .logo {
            position: fixed; top: 20px; right: 20px; font-size: 28px; font-weight: bold; color: white;
            font-family: 'Arial', 'Microsoft YaHei', sans-serif; text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            letter-spacing: 2px; z-index: 100; background: rgba(0,0,0,0.3); padding: 8px 16px;
            border-radius: 12px; backdrop-filter: blur(4px); pointer-events: none;
        }
        /* 尺寸输入框样式 - 增大触摸区域 */
        .dimension-control {
            display: flex;
            gap: 8px;
            background: rgba(0,0,0,0.4);
            padding: 5px 12px;
            border-radius: 40px;
            align-items: center;
            flex-wrap: wrap;
        }
        .dim-input {
            background: #1e2a3a;
            border: 1px solid #3a5a7a;
            color: white;
            padding: 8px 10px;
            border-radius: 30px;
            width: 55px;
            text-align: center;
            font-weight: bold;
            font-size: 14px;
            touch-action: manipulation;
        }
        .dim-label {
            font-size: 12px;
            font-weight: bold;
        }
        /* 触摸提示 */
        .touch-tip {
            position: absolute;
            top: 80px;
            left: 20px;
            background: rgba(0,0,0,0.5);
            padding: 6px 12px;
            border-radius: 25px;
            font-size: 11px;
            color: #ccc;
            pointer-events: none;
            z-index: 20;
            backdrop-filter: blur(4px);
        }
        @media (max-width: 800px) {
            .logo { font-size: 22px; top: 12px; right: 12px; padding: 5px 12px; }
            .controls-panel { flex-wrap: wrap; gap: 8px; padding: 10px 15px; bottom: 10px; left: 10px; right: 10px; }
            .action-btn, .size-btn { padding: 6px 12px; font-size: 12px; min-width: 48px; }
            .dim-input { width: 48px; font-size: 12px; padding: 6px 8px; }
            .rule-badge { font-size: 9px; padding: 4px 8px; }
            .info-badge { font-size: 9px; padding: 4px 8px; }
            .touch-tip { top: 60px; font-size: 9px; }
        }
    </style>
</head>
<body>
    <div class="logo">YHX</div>
    <div class="touch-tip">👆 手指拖拽旋转 | 点击方块消除</div>
    <div class="controls-panel">
        <div class="dimension-control">
            <span class="dim-label">📏 长(X)</span>
            <input type="number" id="size-x" class="dim-input" min="1" max="9" value="5">
            <span class="dim-label">宽(Z)</span>
            <input type="number" id="size-z" class="dim-input" min="1" max="9" value="5">
            <span class="dim-label">高(Y)</span>
            <input type="number" id="size-y" class="dim-input" min="1" max="9" value="5">
            <button id="apply-size" class="action-btn" style="background:#1e5a7a;">✅ 应用</button>
        </div>
        <div class="divider"></div>
        <div class="panel-group">
            <button id="separate-all" class="action-btn separate-btn">🧅 一键逐层分离</button>
            <button id="reset-all" class="action-btn reset-btn">🔄 一键复原</button>
        </div>
        <div class="rule-badge">🎯 点击小块：自身消失 + 对面方向整排消失</div>
        <div class="info-badge">👆 单指旋转 | ✌️ 双指缩放 | 👉 右键平移</div>
    </div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.128.0/build/three.module.js", "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/" } }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0a0a2a);
        scene.fog = new THREE.FogExp2(0x0a0a2a, 0.006);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(12, 9, 16);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // OrbitControls 默认支持触摸
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.rotateSpeed = 1.0;
        controls.zoomSpeed = 1.0;
        controls.panSpeed = 0.8;
        controls.enableZoom = true;
        controls.enablePan = true;
        controls.enableTouchRotate = true;
        controls.touchRotate = true;
        controls.zoomSpeed = 0.8;
        controls.target.set(0, 0, 0);

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        // 光照
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        const mainLight = new THREE.DirectionalLight(0xfff5e0, 1.2);
        mainLight.position.set(4, 6, 3);
        mainLight.castShadow = true;
        scene.add(mainLight);
        const fillLight = new THREE.PointLight(0x88aaff, 0.5);
        fillLight.position.set(-3, 2, 4);
        scene.add(fillLight);
        const warmFill = new THREE.PointLight(0xffaa66, 0.4);
        warmFill.position.set(2, 3, -3);
        scene.add(warmFill);
        const backRim = new THREE.PointLight(0xff88aa, 0.35);
        backRim.position.set(-1, 1, -5);
        scene.add(backRim);

        const gridHelper = new THREE.GridHelper(20, 20, 0x88aaff, 0x446688);
        gridHelper.position.y = -3.5;
        gridHelper.material.transparent = true;
        gridHelper.material.opacity = 0.25;
        scene.add(gridHelper);
        
        const axesHelper = new THREE.AxesHelper(8);
        axesHelper.material.transparent = true;
        axesHelper.material.opacity = 0.1;
        scene.add(axesHelper);
        
        // 粒子
        const particleCount = 1500;
        const particlesGeo = new THREE.BufferGeometry();
        const particlePositions = new Float32Array(particleCount * 3);
        for (let i = 0; i < particleCount; i++) {
            particlePositions[i*3] = (Math.random() - 0.5) * 40;
            particlePositions[i*3+1] = (Math.random() - 0.5) * 25;
            particlePositions[i*3+2] = (Math.random() - 0.5) * 35 - 8;
        }
        particlesGeo.setAttribute('position', new THREE.BufferAttribute(particlePositions, 3));
        const particlesMat = new THREE.PointsMaterial({ color: 0x77aaff, size: 0.05, transparent: true, opacity: 0.4, blending: THREE.AdditiveBlending });
        const particles = new THREE.Points(particlesGeo, particlesMat);
        scene.add(particles);

        // 当前尺寸
        let currentSizeX = 5;
        let currentSizeY = 5;
        let currentSizeZ = 5;
        
        let cubesGroup = new THREE.Group();
        scene.add(cubesGroup);
        let cubesMap = new Map();
        let cubesArray = [];
        let separatedLayers = [];
        let isSeparating = false;
        let isDragging = false;  // 防止拖拽时误触发点击

        const BLUE_COLOR = new THREE.Color(0x3a86ff);
        const HIGHLIGHT_COLOR = new THREE.Color(0xffaa44);
        const SPACING = 5;
        
        function getLayerOffset(z, maxZ) {
            return (maxZ - z) * SPACING;
        }
        
        function getSortedLayerZValues() {
            const zValues = new Set();
            for (let cube of cubesArray) {
                zValues.add(cube.userData.z);
            }
            return Array.from(zValues).sort((a, b) => b - a);
        }

        function buildCubes(sizeX, sizeY, sizeZ) {
            separatedLayers.forEach(layer => {
                scene.remove(layer.group);
            });
            separatedLayers = [];
            isSeparating = false;
            
            while(cubesGroup.children.length) cubesGroup.remove(cubesGroup.children[0]);
            cubesMap.clear();
            cubesArray = [];
            
            const offsetX = (sizeX - 1) / 2;
            const offsetY = (sizeY - 1) / 2;
            const offsetZ = (sizeZ - 1) / 2;
            
            const cubeBaseSize = 0.92;
            const geometry = new THREE.BoxGeometry(cubeBaseSize, cubeBaseSize, cubeBaseSize);
            
            for (let i = 0; i < sizeX; i++) {
                for (let j = 0; j < sizeY; j++) {
                    for (let k = 0; k < sizeZ; k++) {
                        const x = i - offsetX;
                        const y = j - offsetY;
                        const z = k - offsetZ;
                        
                        const material = new THREE.MeshStandardMaterial({ 
                            color: BLUE_COLOR, 
                            roughness: 0.25, 
                            metalness: 0.15, 
                            emissive: 0x001133, 
                            emissiveIntensity: 0.05 
                        });
                        
                        const cube = new THREE.Mesh(geometry, material);
                        cube.position.set(x, y, z);
                        cube.castShadow = true;
                        
                        const edgesGeo = new THREE.EdgesGeometry(geometry);
                        const wireframe = new THREE.LineSegments(edgesGeo, new THREE.LineBasicMaterial({ color: 0x000000 }));
                        cube.add(wireframe);
                        
                        cube.userData = { 
                            x, y, z, 
                            posKey: `${x},${y},${z}`, 
                            layerZ: z, 
                            isSeparated: false 
                        };
                        
                        cubesGroup.add(cube);
                        cubesMap.set(cube.userData.posKey, cube);
                        cubesArray.push(cube);
                    }
                }
            }
            
            if (window.boundsWire) scene.remove(window.boundsWire);
            const wireGeo = new THREE.BoxGeometry(sizeX, sizeY, sizeZ);
            const wireEdges = new THREE.EdgesGeometry(wireGeo);
            const wireMat = new THREE.LineBasicMaterial({ color: 0x7a9dce, transparent: true, opacity: 0.3 });
            window.boundsWire = new THREE.LineSegments(wireEdges, wireMat);
            window.boundsWire.position.set(0, 0, 0);
            scene.add(window.boundsWire);
            
            console.log(`重建积木: ${sizeX} x ${sizeY} x ${sizeZ}，共 ${sizeX*sizeY*sizeZ} 块`);
        }
        
        function separateLayers() {
            if (isSeparating || cubesArray.length === 0) return;
            
            let maxZ = -Infinity;
            for (let cube of cubesArray) {
                if (cube.userData.z > maxZ) maxZ = cube.userData.z;
            }
            
            isSeparating = true;
            const zValues = getSortedLayerZValues();
            let idx = 0;
            
            function next() {
                if (idx >= zValues.length) { 
                    isSeparating = false; 
                    console.log("逐层分离完成");
                    return; 
                }
                const z = zValues[idx++];
                const toSeparate = cubesArray.filter(c => c.userData.z === z);
                const offsetZ = getLayerOffset(z, maxZ);
                const layerGroup = new THREE.Group();
                
                toSeparate.forEach(cube => {
                    cubesGroup.remove(cube);
                    cubesMap.delete(cube.userData.posKey);
                    const i = cubesArray.indexOf(cube);
                    if (i !== -1) cubesArray.splice(i, 1);
                    cube.userData.isSeparated = true;
                    layerGroup.add(cube);
                });
                
                layerGroup.position.set(0, 0, offsetZ);
                scene.add(layerGroup);
                separatedLayers.push({ group: layerGroup, originalZ: z, offset: offsetZ });
                
                toSeparate.forEach(cube => cube.material.color.setHex(HIGHLIGHT_COLOR.getHex()));
                setTimeout(() => {
                    toSeparate.forEach(cube => cube.material.color.setHex(BLUE_COLOR.getHex()));
                    next();
                }, 100);
            }
            next();
        }

        function removeCube(cube) {
            if (!cube.parent) return;
            cube.material.color.setHex(HIGHLIGHT_COLOR.getHex());
            setTimeout(() => {
                if (cube.parent) {
                    if (cube.parent !== cubesGroup && cube.parent.parent === scene) {
                        cube.parent.remove(cube);
                        cubesMap.delete(cube.userData.posKey);
                        if (cube.parent.children.length === 0) {
                            const li = separatedLayers.findIndex(l => l.group === cube.parent);
                            if (li !== -1) { 
                                scene.remove(cube.parent); 
                                separatedLayers.splice(li, 1); 
                            }
                        }
                    } else if (cube.parent === cubesGroup) {
                        cubesGroup.remove(cube);
                        cubesMap.delete(cube.userData.posKey);
                        const i = cubesArray.indexOf(cube);
                        if (i !== -1) cubesArray.splice(i, 1);
                    }
                }
            }, 50);
        }

        // 通用的点击处理（支持鼠标和触摸）
        function handlePick(clientX, clientY) {
            if (isSeparating) return;
            
            const rect = renderer.domElement.getBoundingClientRect();
            mouse.x = ((clientX - rect.left) / rect.width) * 2 - 1;
            mouse.y = -((clientY - rect.top) / rect.height) * 2 + 1;
            
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(cubesArray);
            if (intersects.length === 0) return;
            
            const clicked = intersects[0].object;
            const normal = intersects[0].face.normal;
            const cx = clicked.userData.x, cy = clicked.userData.y, cz = clicked.userData.z;
            const toRemove = [clicked];
            
            if (Math.abs(normal.x) > 0.8) {
                if (normal.x > 0) {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.x < cx && cube.userData.y === cy && cube.userData.z === cz) toRemove.push(cube);
                    }
                } else {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.x > cx && cube.userData.y === cy && cube.userData.z === cz) toRemove.push(cube);
                    }
                }
            } else if (Math.abs(normal.y) > 0.8) {
                if (normal.y > 0) {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.y < cy && cube.userData.x === cx && cube.userData.z === cz) toRemove.push(cube);
                    }
                } else {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.y > cy && cube.userData.x === cx && cube.userData.z === cz) toRemove.push(cube);
                    }
                }
            } else if (Math.abs(normal.z) > 0.8) {
                if (normal.z > 0) {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.z < cz && cube.userData.x === cx && cube.userData.y === cy) toRemove.push(cube);
                    }
                } else {
                    for (let [k, cube] of cubesMap) {
                        if (cube !== clicked && cube.userData.z > cz && cube.userData.x === cx && cube.userData.y === cy) toRemove.push(cube);
                    }
                }
            }
            toRemove.forEach(c => removeCube(c));
        }

        // 鼠标事件
        let mouseDownTime = 0;
        let mouseMoveFlag = false;
        
        renderer.domElement.addEventListener('mousedown', (e) => {
            mouseDownTime = Date.now();
            mouseMoveFlag = false;
        });
        
        renderer.domElement.addEventListener('mousemove', () => {
            mouseMoveFlag = true;
        });
        
        renderer.domElement.addEventListener('mouseup', (e) => {
            const duration = Date.now() - mouseDownTime;
            if (!mouseMoveFlag && duration < 200) {
                handlePick(e.clientX, e.clientY);
            }
        });
        
        // 触摸事件（触屏专用）
        let touchStartTime = 0;
        let touchMoved = false;
        let touchStartPos = { x: 0, y: 0 };
        
        renderer.domElement.addEventListener('touchstart', (e) => {
            touchStartTime = Date.now();
            touchMoved = false;
            if (e.touches.length === 1) {
                touchStartPos = { x: e.touches[0].clientX, y: e.touches[0].clientY };
            }
        });
        
        renderer.domElement.addEventListener('touchmove', (e) => {
            if (e.touches.length === 1) {
                const dx = Math.abs(e.touches[0].clientX - touchStartPos.x);
                const dy = Math.abs(e.touches[0].clientY - touchStartPos.y);
                if (dx > 10 || dy > 10) {
                    touchMoved = true;
                }
            } else {
                touchMoved = true;
            }
        });
        
        renderer.domElement.addEventListener('touchend', (e) => {
            const duration = Date.now() - touchStartTime;
            // 如果没有移动且点击时间短，视为点击
            if (!touchMoved && duration < 200 && e.changedTouches.length > 0) {
                const touch = e.changedTouches[0];
                handlePick(touch.clientX, touch.clientY);
            }
        });

        function resetAll() { 
            buildCubes(currentSizeX, currentSizeY, currentSizeZ); 
        }
        
        function applySize() {
            const newX = parseInt(document.getElementById('size-x').value) || 5;
            const newY = parseInt(document.getElementById('size-y').value) || 5;
            const newZ = parseInt(document.getElementById('size-z').value) || 5;
            
            currentSizeX = Math.min(Math.max(newX, 1), 9);
            currentSizeY = Math.min(Math.max(newY, 1), 9);
            currentSizeZ = Math.min(Math.max(newZ, 1), 9);
            
            document.getElementById('size-x').value = currentSizeX;
            document.getElementById('size-y').value = currentSizeY;
            document.getElementById('size-z').value = currentSizeZ;
            
            buildCubes(currentSizeX, currentSizeY, currentSizeZ);
            
            const maxDim = Math.max(currentSizeX, currentSizeY, currentSizeZ);
            const dist = Math.max(8, maxDim * 1.5);
            camera.position.set(dist, dist * 0.7, dist);
            controls.target.set(0, 0, 0);
            controls.update();
        }

        // 事件绑定
        document.getElementById('reset-all').addEventListener('click', resetAll);
        document.getElementById('separate-all').addEventListener('click', separateLayers);
        document.getElementById('apply-size').addEventListener('click', applySize);
        
        ['size-x', 'size-y', 'size-z'].forEach(id => {
            document.getElementById(id).addEventListener('keypress', (e) => {
                if (e.key === 'Enter') applySize();
            });
        });

        buildCubes(5, 5, 5);
        
        let time = 0;
        function animate() {
            requestAnimationFrame(animate);
            time += 0.006;
            particles.rotation.y = time * 0.08;
            particles.rotation.x = Math.sin(time * 0.15) * 0.1;
            controls.update();
            renderer.render(scene, camera);
        }
        animate();
        
        window.addEventListener('resize', () => { 
            camera.aspect = window.innerWidth / window.innerHeight; 
            camera.updateProjectionMatrix(); 
            renderer.setSize(window.innerWidth, window.innerHeight); 
        });
        
        console.log("✅ 3D积木已启动 | 支持触摸屏 | 可分别调整长(X)、宽(Z)、高(Y)");
    </script>
</body>
</html>
