<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>森のささやき島 3D - Ghibli Style</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Kiwi+Maru:wght@400;500&display=swap" rel="stylesheet">
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Kiwi Maru', serif; background: #e0f2f1; }
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
        .interactive { pointer-events: auto; }
        
        .ui-panel {
            background: rgba(255, 255, 255, 0.85);
            backdrop-filter: blur(8px);
            border-radius: 20px;
            padding: 15px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            border: 2px solid #d4e157;
        }

        .modal {
            display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: #fffef0; border: 4px solid #8b5a2b; border-radius: 24px;
            padding: 24px; z-index: 3000; width: 340px; box-shadow: 0 0 0 1000px rgba(0,0,0,0.4);
            pointer-events: auto;
        }

        #crosshair {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 20px; height: 20px; border: 2px solid rgba(255,255,255,0.5); border-radius: 50%;
            pointer-events: none;
        }

        .instruction {
            position: absolute; bottom: 100px; left: 50%; transform: translateX(-50%);
            background: rgba(0,0,0,0.5); color: white; padding: 5px 15px; border-radius: 20px; font-size: 12px;
        }

        #toast {
            position: fixed; top: 100px; left: 50%; transform: translateX(-50%);
            background: rgba(255, 255, 255, 0.9); padding: 8px 20px; border-radius: 20px;
            border: 2px solid #8b5a2b; font-size: 14px; opacity: 0; transition: opacity 0.3s;
            pointer-events: none; z-index: 4000;
        }
    </style>
</head>
<body>

<div id="ui-layer">
    <div class="ui-panel interactive" style="position:absolute; top:20px; left:20px;">
        <div class="text-green-800 font-bold">🌿 森のささやき島 3D</div>
        <div class="flex items-center gap-3 mt-1">
            <span class="text-yellow-600 font-bold">💰 <span id="coin-count">50</span> B</span>
            <span id="clock" class="text-xs text-gray-500">12:00</span>
        </div>
    </div>

    <div class="instruction">WASDで移動 / マウスドラッグで視点変更 / クリックでアクション</div>

    <div class="ui-panel interactive" style="position:absolute; bottom:20px; left:50%; transform:translateX(-50%); display:flex; gap:10px;">
        <div class="flex flex-col items-center">🍎<span id="count-apple" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🍊<span id="count-orange" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🐟<span id="count-fish" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🐚<span id="count-shell" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🦴<span id="count-fossil" class="text-xs">0</span></div>
    </div>

    <div class="interactive" style="position:absolute; top:20px; right:20px;">
        <button onclick="toggleModal('craft-modal')" class="ui-panel hover:bg-green-50 px-4 py-2 font-bold">⚒️ つくる</button>
    </div>

    <div id="crosshair"></div>
</div>

<div id="toast"></div>

<div id="shop-modal" class="modal">
    <h3 class="text-xl font-bold mb-4 text-center text-yellow-800">タヌキ商店</h3>
    <div class="space-y-2 overflow-y-auto max-h-64">
        <button onclick="sellAll('apple', 10)" class="w-full bg-white border-2 border-red-100 p-2 rounded-xl text-sm hover:bg-red-50">🍎 リンゴを売る (10B)</button>
        <button onclick="sellAll('orange', 15)" class="w-full bg-white border-2 border-orange-100 p-2 rounded-xl text-sm hover:bg-orange-50">🍊 オレンジを売る (15B)</button>
        <button onclick="sellAll('fish', 50)" class="w-full bg-white border-2 border-blue-100 p-2 rounded-xl text-sm hover:bg-blue-50">🐟 サカナを売る (50B)</button>
        <button onclick="sellAll('shell', 20)" class="w-full bg-white border-2 border-pink-100 p-2 rounded-xl text-sm hover:bg-pink-50">🐚 貝殻を売る (20B)</button>
        <button onclick="sellAll('fossil', 150)" class="w-full bg-white border-2 border-gray-200 p-2 rounded-xl text-sm hover:bg-gray-100">🦴 化石を売る (150B)</button>
    </div>
    <button onclick="toggleModal('shop-modal')" class="mt-6 w-full py-2 bg-gray-300 rounded-xl">とじる</button>
</div>

<div id="craft-modal" class="modal">
    <h3 class="text-xl font-bold mb-4 text-center text-green-800">建築メニュー</h3>
    <div class="space-y-3 overflow-y-auto max-h-60">
        <button onclick="craft('log_house')" class="w-full bg-green-50 p-3 rounded-xl text-left text-sm border border-green-200">🏡 ログハウス (100B)</button>
        <button onclick="craft('castle')" class="w-full bg-red-50 p-3 rounded-xl text-left text-sm border border-red-200">🏯 お城 (300B)</button>
        <button onclick="craft('fountain')" class="w-full bg-blue-50 p-3 rounded-xl text-left text-sm border border-blue-200">⛲ 噴水 (50B)</button>
    </div>
    <button onclick="toggleModal('craft-modal')" class="mt-6 w-full py-2 bg-gray-300 rounded-xl">とじる</button>
</div>

<script>
    let scene, camera, renderer, clock, player, sprout;
    const playerState = {
        coins: 50,
        inventory: { apple: 0, orange: 0, fish: 0, shell: 0, fossil: 0 },
        isMoving: false
    };
    const moveState = { forward: false, backward: false, left: false, right: false };
    const objects = [];
    let isMouseDown = false;
    let yaw = 0, pitch = -0.5;

    function init() {
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0xa1c4fd);
        scene.fog = new THREE.FogExp2(0xa1c4fd, 0.005);

        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        clock = new THREE.Clock();

        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);
        const sunLight = new THREE.DirectionalLight(0xfff0dd, 0.6);
        sunLight.position.set(50, 100, 50);
        scene.add(sunLight);

        // Ground
        const groundGeo = new THREE.CircleGeometry(500, 64);
        const groundMat = new THREE.MeshPhongMaterial({ color: 0x91cf60 });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        scene.add(ground);

        // Beach
        const beachGeo = new THREE.RingGeometry(80, 100, 64);
        const beachMat = new THREE.MeshPhongMaterial({ color: 0xffecb3 });
        const beach = new THREE.Mesh(beachGeo, beachMat);
        beach.rotation.x = -Math.PI / 2;
        beach.position.y = 0.02;
        scene.add(beach);

        // Player
        player = new THREE.Group();
        const whiteMat = new THREE.MeshPhongMaterial({ color: 0xffffff });
        const blackMat = new THREE.MeshPhongMaterial({ color: 0x333333 });
        const pinkMat = new THREE.MeshPhongMaterial({ color: 0xffb7c5 });
        const leafMat = new THREE.MeshPhongMaterial({ color: 0x4caf50 });

        const bodyBottom = new THREE.Mesh(new THREE.SphereGeometry(0.7, 16, 16), whiteMat);
        bodyBottom.position.y = 0.6;
        bodyBottom.scale.set(1.1, 0.9, 1.1);
        player.add(bodyBottom);

        const bodyTop = new THREE.Mesh(new THREE.SphereGeometry(0.55, 16, 16), whiteMat);
        bodyTop.position.y = 1.3;
        player.add(bodyTop);

        const eyeGeo = new THREE.SphereGeometry(0.06, 8, 8);
        const leftEye = new THREE.Mesh(eyeGeo, blackMat);
        leftEye.position.set(0.2, 1.4, 0.45);
        player.add(leftEye);
        const rightEye = new THREE.Mesh(eyeGeo, blackMat);
        rightEye.position.set(-0.2, 1.4, 0.45);
        player.add(rightEye);

        const cheekGeo = new THREE.SphereGeometry(0.1, 8, 8);
        const leftCheek = new THREE.Mesh(cheekGeo, pinkMat);
        leftCheek.position.set(0.35, 1.25, 0.4);
        leftCheek.scale.set(1, 0.5, 1);
        player.add(leftCheek);
        const rightCheek = new THREE.Mesh(cheekGeo, pinkMat);
        rightCheek.position.set(-0.35, 1.25, 0.4);
        rightCheek.scale.set(1, 0.5, 1);
        player.add(rightCheek);

        sprout = new THREE.Group();
        const leaf1 = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), leafMat);
        leaf1.scale.set(1, 0.2, 0.5);
        leaf1.position.set(0.1, 0, 0);
        leaf1.rotation.z = 0.3;
        sprout.add(leaf1);
        const leaf2 = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), leafMat);
        leaf2.scale.set(1, 0.2, 0.5);
        leaf2.position.set(-0.1, 0, 0);
        leaf2.rotation.z = -0.3;
        sprout.add(leaf2);
        sprout.position.y = 1.85;
        player.add(sprout);

        scene.add(player);

        // Spawning items
        for(let i=0; i<40; i++) spawnTree('apple');
        for(let i=0; i<30; i++) spawnTree('orange');
        for(let i=0; i<25; i++) spawnPond();
        for(let i=0; i<25; i++) spawnFossilMark();
        for(let i=0; i<40; i++) spawnShell();
        for(let i=0; i<150; i++) spawnFlower();

        createBuilding(-10, -10, 0xfbc02d, "商店");
        createBuilding(10, -15, 0x90caf9, "博物館");

        window.addEventListener('keydown', (e) => updateMovement(e.code, true));
        window.addEventListener('keyup', (e) => updateMovement(e.code, false));
        window.addEventListener('mousedown', (e) => { if(e.target.tagName === 'CANVAS') isMouseDown = true; });
        window.addEventListener('mouseup', () => isMouseDown = false);
        window.addEventListener('mousemove', onMouseMove);
        window.addEventListener('click', onMouseClick);
        window.addEventListener('resize', onWindowResize);

        animate();
    }

    // Updated Fruit Logic with Detail
    function spawnTree(fruitType) {
        const x = (Math.random() - 0.5) * 160;
        const z = (Math.random() - 0.5) * 160;
        if (Math.abs(x) < 8 && Math.abs(z) < 8) return;
        const group = new THREE.Group();
        const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.3, 2), new THREE.MeshPhongMaterial({ color: 0x795548 }));
        trunk.position.y = 1;
        group.add(trunk);
        const leaves = new THREE.Mesh(new THREE.SphereGeometry(1.2, 8, 8), new THREE.MeshPhongMaterial({ color: 0x4a7c44, flatShading: true }));
        leaves.position.y = 2.5;
        group.add(leaves);
        
        // Fruit Visual Improvement
        const fruitGroup = new THREE.Group();
        const fruitGeo = new THREE.SphereGeometry(0.18, 12, 12);
        const fruitMat = new THREE.MeshPhongMaterial({ 
            color: fruitType === 'apple' ? 0xff3d00 : 0xffa000,
            shininess: 100 
        });
        const fruit = new THREE.Mesh(fruitGeo, fruitMat);
        fruitGroup.add(fruit);

        // Add Stem (枝)
        const stemGeo = new THREE.CylinderGeometry(0.02, 0.02, 0.15);
        const stem = new THREE.Mesh(stemGeo, new THREE.MeshPhongMaterial({color: 0x4e342e}));
        stem.position.y = 0.18;
        fruitGroup.add(stem);

        // Add Leaf to fruit for Apple
        if(fruitType === 'apple') {
            const fLeafGeo = new THREE.SphereGeometry(0.08, 8, 8);
            const fLeaf = new THREE.Mesh(fLeafGeo, new THREE.MeshPhongMaterial({color: 0x64dd17}));
            fLeaf.scale.set(1, 0.1, 0.5);
            fLeaf.position.set(0.05, 0.22, 0);
            fLeaf.rotation.z = 0.5;
            fruitGroup.add(fLeaf);
        }

        fruitGroup.position.set(0.6, 2.2, 0.6);
        group.add(fruitGroup);

        group.position.set(x, 0, z);
        group.userData = { type: 'tree', fruit: fruitType };
        scene.add(group);
        objects.push(group);
    }

    function spawnFossilMark() {
        const x = (Math.random() - 0.5) * 150;
        const z = (Math.random() - 0.5) * 150;
        if (Math.abs(x) < 10 && Math.abs(z) < 10) return;
        const group = new THREE.Group();
        
        // Visual Improvement: Mound of dirt with crack
        const mound = new THREE.Mesh(new THREE.SphereGeometry(0.4, 8, 8), new THREE.MeshPhongMaterial({color: 0x8d6e63}));
        mound.scale.y = 0.1;
        group.add(mound);

        const markMat = new THREE.MeshPhongMaterial({ color: 0x4e342e });
        const m1 = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.02, 0.08), markMat);
        m1.rotation.y = Math.PI / 4;
        m1.position.y = 0.05;
        const m2 = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.02, 0.08), markMat);
        m2.rotation.y = -Math.PI / 4;
        m2.position.y = 0.05;
        group.add(m1, m2);

        group.position.set(x, 0, z);
        group.userData = { type: 'fossil' };
        scene.add(group);
        objects.push(group);
    }

    // Shell Design: Spiraled Snail Style
    function spawnShell() {
        const angle = Math.random() * Math.PI * 2;
        const radius = 85 + Math.random() * 10;
        const x = Math.cos(angle) * radius;
        const z = Math.sin(angle) * radius;
        
        const shellGroup = new THREE.Group();
        const mat = new THREE.MeshPhongMaterial({ color: 0xfff1f1 });
        
        // Create 3 segments for a spiral look
        for(let i=0; i<3; i++) {
            const seg = new THREE.Mesh(new THREE.SphereGeometry(0.2 - (i*0.05), 8, 8), mat);
            seg.position.x = i * 0.15;
            seg.scale.set(1.2, 1, 1);
            shellGroup.add(seg);
        }
        
        shellGroup.position.set(x, 0.1, z);
        shellGroup.rotation.y = Math.random() * Math.PI;
        shellGroup.userData = { type: 'shell' };
        scene.add(shellGroup);
        objects.push(shellGroup);
    }

    function spawnPond() {
        const x = (Math.random() - 0.5) * 180;
        const z = (Math.random() - 0.5) * 180;
        const group = new THREE.Group();

        const pond = new THREE.Mesh(new THREE.CircleGeometry(3, 24), new THREE.MeshPhongMaterial({ 
            color: 0x4fc3f7, transparent: true, opacity: 0.7 
        }));
        pond.rotation.x = -Math.PI / 2;
        group.add(pond);

        // Add Fish Shadow
        const shadow = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.01, 0.2), new THREE.MeshBasicMaterial({color: 0x01579b, transparent: true, opacity: 0.4}));
        shadow.position.set(1, 0.01, 0);
        shadow.userData = { isFish: true, offset: Math.random() * 10 };
        group.add(shadow);

        group.position.set(x, 0.01, z);
        group.userData = { type: 'pond' };
        scene.add(group);
        objects.push(group);
    }

    function spawnFlower() {
        const x = (Math.random() - 0.5) * 250;
        const z = (Math.random() - 0.5) * 250;
        const flower = new THREE.Mesh(new THREE.PlaneGeometry(0.4, 0.4), new THREE.MeshPhongMaterial({ 
            color: Math.random() > 0.5 ? 0xff4081 : 0xffeb3b, side: THREE.DoubleSide 
        }));
        flower.position.set(x, 0.15, z);
        flower.rotation.x = -Math.PI / 4;
        scene.add(flower);
    }

    function createBuilding(x, z, color, type) {
        const group = new THREE.Group();
        const body = new THREE.Mesh(new THREE.BoxGeometry(4, 4, 4), new THREE.MeshPhongMaterial({ color: color }));
        body.position.y = 2;
        group.add(body);
        const roof = new THREE.Mesh(new THREE.ConeGeometry(4, 3, 4), new THREE.MeshPhongMaterial({ color: 0x8b5a2b }));
        roof.position.y = 5.5;
        roof.rotation.y = Math.PI / 4;
        group.add(roof);
        group.position.set(x, 0, z);
        group.userData = { type: 'building', buildingType: type };
        scene.add(group);
        objects.push(group);
    }

    function updateMovement(code, isPressed) {
        switch(code) {
            case 'KeyW': moveState.forward = isPressed; break;
            case 'KeyS': moveState.backward = isPressed; break;
            case 'KeyA': moveState.left = isPressed; break;
            case 'KeyD': moveState.right = isPressed; break;
        }
    }

    function onMouseMove(e) {
        if (!isMouseDown) return;
        yaw -= e.movementX * 0.005;
        pitch -= e.movementY * 0.005;
        pitch = Math.max(-Math.PI/2.1, Math.min(Math.PI/4, pitch));
    }

    function onMouseClick(e) {
        if (e.target.tagName !== 'CANVAS') return;
        const raycaster = new THREE.Raycaster();
        raycaster.setFromCamera({x: 0, y: 0}, camera);
        const intersects = raycaster.intersectObjects(objects, true);
        if (intersects.length > 0) {
            let obj = intersects[0].object;
            while(obj.parent && obj.parent !== scene) obj = obj.parent;
            if (intersects[0].distance > 8) return;

            if (obj.userData.type === 'tree') {
                const f = obj.userData.fruit;
                playerState.inventory[f]++;
                showMessage(`${f === 'apple' ? '🍎 リンゴ' : '🍊 オレンジ'}を収穫した！`);
            } else if (obj.userData.type === 'pond') {
                if(Math.random() > 0.5) {
                    playerState.inventory.fish++;
                    showMessage("🐟 魚が釣れた！");
                } else showMessage("逃げられた...");
            } else if (obj.userData.type === 'fossil') {
                if(Math.random() > 0.3) {
                    playerState.inventory.fossil++;
                    showMessage("🦴 化石を掘り出した！");
                } else {
                    playerState.coins += 100;
                    showMessage("💰 100ベルが埋まっていた！");
                }
                scene.remove(obj);
                objects.splice(objects.indexOf(obj), 1);
            } else if (obj.userData.type === 'shell') {
                playerState.inventory.shell++;
                showMessage("🐚 貝殻を拾った！");
                scene.remove(obj);
                objects.splice(objects.indexOf(obj), 1);
            } else if (obj.userData.type === 'building' && obj.userData.buildingType === '商店') {
                toggleModal('shop-modal');
            }
            updateUI();
        }
    }

    function animate() {
        requestAnimationFrame(animate);
        const delta = clock.getDelta();
        const time = Date.now() * 0.001;

        const moveDir = new THREE.Vector3();
        if (moveState.forward) moveDir.z -= 1;
        if (moveState.backward) moveDir.z += 1;
        if (moveState.left) moveDir.x -= 1;
        if (moveState.right) moveDir.x += 1;

        if (moveDir.length() > 0) {
            moveDir.normalize();
            moveDir.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            player.position.addScaledVector(moveDir, 8 * delta);
            player.rotation.y = Math.atan2(moveDir.x, moveDir.z);
            player.position.y = Math.abs(Math.sin(time * 12)) * 0.3;
            sprout.rotation.y = Math.sin(time * 15) * 0.5;
        } else {
            player.position.y = 0;
            player.scale.y = 1 + Math.sin(time * 3) * 0.02;
            sprout.rotation.y = Math.sin(time * 2) * 0.1;
        }

        const camOffset = new THREE.Vector3(0, 0, 10);
        camOffset.applyAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
        camOffset.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
        camera.position.copy(player.position).add(camOffset);
        camera.lookAt(player.position.clone().add(new THREE.Vector3(0, 1.2, 0)));

        scene.traverse((child) => {
            if (child.userData.type === 'tree') child.rotation.z = Math.sin(time * 2) * 0.02;
            // Fish swimming logic
            if (child.userData.isFish) {
                const offset = child.userData.offset;
                child.position.x = Math.sin(time + offset) * 1.5;
                child.position.z = Math.cos(time + offset) * 1.5;
                child.rotation.y = -(time + offset);
            }
        });

        renderer.render(scene, camera);
    }

    function sellAll(type, price) {
        const count = playerState.inventory[type];
        if (count > 0) {
            playerState.coins += count * price;
            playerState.inventory[type] = 0;
            updateUI();
            showMessage(`💰 ${count * price}ベル手に入れた！`);
        }
    }

    function craft(type) {
        const costs = { log_house: 100, castle: 300, fountain: 50 };
        if (playerState.coins >= costs[type]) {
            playerState.coins -= costs[type];
            createBuilding(player.position.x + 3, player.position.z + 3, type === 'log_house' ? 0xd7ccc8 : (type === 'castle' ? 0xffffff : 0x80deea), "マイハウス");
            updateUI();
            showMessage("✨ 建築完了だも！");
            toggleModal('craft-modal');
        } else showMessage("ベルが足りないだも...");
    }

    function showMessage(text) {
        const toast = document.getElementById('toast');
        toast.innerText = text;
        toast.style.opacity = 1;
        setTimeout(() => toast.style.opacity = 0, 2000);
    }

    function updateUI() {
        document.getElementById('coin-count').innerText = playerState.coins;
        document.getElementById('count-apple').innerText = playerState.inventory.apple;
        document.getElementById('count-orange').innerText = playerState.inventory.orange;
        document.getElementById('count-fish').innerText = playerState.inventory.fish;
        document.getElementById('count-shell').innerText = playerState.inventory.shell;
        document.getElementById('count-fossil').innerText = playerState.inventory.fossil;
    }

    function toggleModal(id) {
        const m = document.getElementById(id);
        m.style.display = m.style.display === 'block' ? 'none' : 'block';
    }

    function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function updateClock() {
        document.getElementById('clock').innerText = new Date().toLocaleTimeString('ja-JP', {hour:'2-digit', minute:'2-digit'});
    }

    setInterval(updateClock, 1000);
    window.onload = init;
</script>
</body>
</html>
