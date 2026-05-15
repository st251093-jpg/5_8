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
    <!-- ステータス -->
    <div class="ui-panel interactive" style="position:absolute; top:20px; left:20px;">
        <div class="text-green-800 font-bold">🌿 森のささやき島 3D</div>
        <div class="flex items-center gap-3 mt-1">
            <span class="text-yellow-600 font-bold">💰 <span id="coin-count">50</span> B</span>
            <span id="clock" class="text-xs text-gray-500">12:00</span>
        </div>
    </div>

    <!-- 操作ガイド -->
    <div class="instruction">WASDで移動 / マウスドラッグで視点変更 / クリックで収穫・釣り</div>

    <!-- インベントリ -->
    <div class="ui-panel interactive" style="position:absolute; bottom:20px; left:50%; transform:translateX(-50%); display:flex; gap:10px;">
        <div class="flex flex-col items-center">🍎<span id="count-apple" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🍊<span id="count-orange" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🐟<span id="count-fish" class="text-xs">0</span></div>
        <div class="flex flex-col items-center">🥢<span id="count-stick" class="text-xs">0</span></div>
    </div>

    <div class="interactive" style="position:absolute; top:20px; right:20px;">
        <button onclick="toggleModal('craft-modal')" class="ui-panel hover:bg-green-50 px-4 py-2 font-bold">⚒️ つくる</button>
    </div>

    <div id="crosshair"></div>
</div>

<div id="toast"></div>

<!-- 商店モーダル -->
<div id="shop-modal" class="modal">
    <h3 class="text-xl font-bold mb-4 text-center text-yellow-800">タヌキ商店</h3>
    <div class="space-y-3">
        <button onclick="sellAll('apple', 10)" class="w-full bg-white border-2 border-yellow-200 p-2 rounded-xl text-sm hover:bg-yellow-50">🍎 リンゴをすべて売る</button>
        <button onclick="sellAll('fish', 50)" class="w-full bg-white border-2 border-blue-200 p-2 rounded-xl text-sm hover:bg-blue-50">🐟 サカナをすべて売る</button>
    </div>
    <button onclick="toggleModal('shop-modal')" class="mt-6 w-full py-2 bg-gray-300 rounded-xl">とじる</button>
</div>

<!-- クラフトモーダル -->
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
    let scene, camera, renderer, clock, player;
    const playerState = {
        x: 0, z: 0, rotation: 0,
        coins: 50,
        inventory: { apple: 0, orange: 0, fish: 0, stick: 0 }
    };
    const moveState = { forward: false, backward: false, left: false, right: false };
    const objects = [];
    let isMouseDown = false;
    let yaw = 0, pitch = -0.5;

    function init() {
        scene = new THREE.Scene();
        // ジブリのような鮮やかな空の色
        scene.background = new THREE.Color(0xa1c4fd);
        scene.fog = new THREE.FogExp2(0xa1c4fd, 0.005);

        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        clock = new THREE.Clock();

        // ライティング
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);
        const sunLight = new THREE.DirectionalLight(0xfff0dd, 0.6);
        sunLight.position.set(50, 100, 50);
        scene.add(sunLight);

        // 地面 (水彩画のような緑)
        const groundGeo = new THREE.CircleGeometry(500, 64);
        const groundMat = new THREE.MeshPhongMaterial({ color: 0x91cf60 });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        scene.add(ground);

        // プレイヤー (CapsuleGeometryエラー回避のため、球体と円柱で作成)
        player = new THREE.Group();
        
        const bodyGeo = new THREE.CylinderGeometry(0.5, 0.5, 1, 16);
        const bodyMat = new THREE.MeshPhongMaterial({ color: 0xfdf5e6 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.position.y = 1;
        player.add(body);

        const headGeo = new THREE.SphereGeometry(0.5, 16, 16);
        const head = new THREE.Mesh(headGeo, bodyMat);
        head.position.y = 1.5;
        player.add(head);

        const bottomGeo = new THREE.SphereGeometry(0.5, 16, 16);
        const bottom = new THREE.Mesh(bottomGeo, bodyMat);
        bottom.position.y = 0.5;
        player.add(bottom);

        player.position.y = 0;
        scene.add(player);

        // 自然生成
        for(let i=0; i<80; i++) spawnTree();
        for(let i=0; i<30; i++) spawnPond();
        for(let i=0; i<150; i++) spawnFlower();

        // 商店と博物館
        createBuilding(-10, -10, 0xfbc02d, "商店");
        createBuilding(10, -15, 0x90caf9, "博物館");

        // イベント
        window.addEventListener('keydown', (e) => updateMovement(e.code, true));
        window.addEventListener('keyup', (e) => updateMovement(e.code, false));
        window.addEventListener('mousedown', (e) => { if(e.target.tagName === 'CANVAS') isMouseDown = true; });
        window.addEventListener('mouseup', () => isMouseDown = false);
        window.addEventListener('mousemove', onMouseMove);
        window.addEventListener('click', onMouseClick);
        window.addEventListener('resize', onWindowResize);

        animate();
    }

    function spawnTree() {
        const x = (Math.random() - 0.5) * 200;
        const z = (Math.random() - 0.5) * 200;
        if (Math.abs(x) < 5 && Math.abs(z) < 5) return;

        const group = new THREE.Group();
        
        const trunkGeo = new THREE.CylinderGeometry(0.2, 0.3, 2);
        const trunkMat = new THREE.MeshPhongMaterial({ color: 0x795548 });
        const trunk = new THREE.Mesh(trunkGeo, trunkMat);
        trunk.position.y = 1;
        group.add(trunk);

        const leavesGeo = new THREE.SphereGeometry(1.2, 8, 8);
        const leavesMat = new THREE.MeshPhongMaterial({ color: 0x4a7c44, flatShading: true });
        const leaves = new THREE.Mesh(leavesGeo, leavesMat);
        leaves.position.y = 2.5;
        group.add(leaves);

        group.position.set(x, 0, z);
        group.userData = { type: 'tree' };
        scene.add(group);
        objects.push(group);
    }

    function spawnPond() {
        const x = (Math.random() - 0.5) * 180;
        const z = (Math.random() - 0.5) * 180;
        const pondGeo = new THREE.CircleGeometry(2 + Math.random() * 3, 16);
        const pondMat = new THREE.MeshPhongMaterial({ color: 0x4fc3f7, transparent: true, opacity: 0.8 });
        const pond = new THREE.Mesh(pondGeo, pondMat);
        pond.rotation.x = -Math.PI / 2;
        pond.position.set(x, 0.01, z);
        pond.userData = { type: 'pond' };
        scene.add(pond);
        objects.push(pond);
    }

    function spawnFlower() {
        const x = (Math.random() - 0.5) * 250;
        const z = (Math.random() - 0.5) * 250;
        const geo = new THREE.PlaneGeometry(0.4, 0.4);
        const mat = new THREE.MeshPhongMaterial({ color: Math.random() > 0.5 ? 0xff4081 : 0xffeb3b, side: THREE.DoubleSide });
        const flower = new THREE.Mesh(geo, mat);
        flower.position.set(x, 0.2, z);
        flower.rotation.x = -Math.PI / 4;
        scene.add(flower);
    }

    function createBuilding(x, z, color, type) {
        const group = new THREE.Group();
        const bodyGeo = new THREE.BoxGeometry(4, 4, 4);
        const bodyMat = new THREE.MeshPhongMaterial({ color: color });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.position.y = 2;
        group.add(body);

        const roofGeo = new THREE.ConeGeometry(4, 3, 4);
        const roofMat = new THREE.MeshPhongMaterial({ color: 0x8b5a2b });
        const roof = new THREE.Mesh(roofGeo, roofMat);
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
            // 親グループがある場合はそれを取得
            while(obj.parent && obj.parent !== scene) {
                obj = obj.parent;
            }
            
            const dist = intersects[0].distance;
            if (dist > 8) return;

            if (obj.userData.type === 'tree') {
                playerState.inventory.apple++;
                showMessage("🍎 リンゴを収穫した！");
            } else if (obj.userData.type === 'pond') {
                if(Math.random() > 0.5) {
                    playerState.inventory.fish++;
                    showMessage("🐟 魚が釣れた！");
                } else {
                    showMessage("逃げられた...");
                }
            } else if (obj.userData.type === 'building' && obj.userData.buildingType === '商店') {
                toggleModal('shop-modal');
            }
            updateUI();
        }
    }

    function animate() {
        requestAnimationFrame(animate);
        const delta = clock.getDelta();

        // プレイヤー移動
        const moveDir = new THREE.Vector3();
        if (moveState.forward) moveDir.z -= 1;
        if (moveState.backward) moveDir.z += 1;
        if (moveState.left) moveDir.x -= 1;
        if (moveState.right) moveDir.x += 1;

        if (moveDir.length() > 0) {
            moveDir.normalize();
            moveDir.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            player.position.addScaledVector(moveDir, 8 * delta);
            // 進行方向を向く
            const targetRotation = Math.atan2(moveDir.x, moveDir.z);
            player.rotation.y = targetRotation;
        }

        // カメラ更新 (三人称視点)
        const camOffset = new THREE.Vector3(0, 0, 10);
        camOffset.applyAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
        camOffset.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
        
        camera.position.copy(player.position).add(camOffset);
        camera.lookAt(player.position.clone().add(new THREE.Vector3(0, 1, 0)));

        // 植物の揺れアニメーション
        const time = Date.now() * 0.002;
        scene.traverse((child) => {
            if (child.userData.type === 'tree') {
                child.rotation.z = Math.sin(time) * 0.02;
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
            const color = type === 'log_house' ? 0xd7ccc8 : (type === 'castle' ? 0xffffff : 0x80deea);
            createBuilding(player.position.x + 3, player.position.z + 3, color, "マイハウス");
            updateUI();
            showMessage("✨ 建築が完了しただも！");
            toggleModal('craft-modal');
        } else {
            showMessage("ベルが足りないだも...");
        }
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
        document.getElementById('count-fish').innerText = playerState.inventory.fish;
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
</html><!DOCTYPE html>
