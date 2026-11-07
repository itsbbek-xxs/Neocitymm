# Neocitymm
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Noctora City 3D</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Roboto, sans-serif; }
        body { overflow: hidden; background-color: #05050a; color: #fff; }
        #canvas-container { width: 100vw; height: 100vh; }
        
        /* HUD UI */
        .ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
        .interactive { pointer-events: auto; }
        
        .header { position: absolute; top: 20px; left: 20px; background: rgba(10, 10, 20, 0.8); backdrop-filter: blur(10px); padding: 15px 25px; border-radius: 12px; border: 1px solid rgba(0, 240, 255, 0.3); }
        .header h1 { font-size: 24px; color: #00f0ff; text-shadow: 0 0 10px #00f0ff; margin-bottom: 4px; }
        .header p { font-size: 12px; color: #aaa; }
        
        .btn-main { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); background: linear-gradient(45deg, #ff007f, #00f0ff); border: none; padding: 15px 35px; border-radius: 30px; font-weight: bold; color: #white; font-size: 16px; cursor: pointer; box-shadow: 0 0 20px rgba(0, 240, 255, 0.5); transition: 0.3s; }
        .btn-main:hover { transform: translateX(-50%) scale(1.05); box-shadow: 0 0 30px rgba(255, 0, 127, 0.8); }

        /* Modal Form */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); backdrop-filter: blur(5px); justify-content: center; align-items: center; z-index: 100; }
        .modal-content { background: #10101a; border: 1px solid #00f0ff; padding: 30px; border-radius: 16px; width: 350px; text-align: center; box-shadow: 0 0 30px rgba(0,240,255,0.2); }
        .modal-content h2 { color: #00f0ff; margin-bottom: 20px; }
        .modal-content input { width: 100%; padding: 12px; margin: 10px 0; background: #1a1a2e; border: 1px solid #333; color: #fff; border-radius: 8px; outline: none; }
        .modal-content input:focus { border-color: #00f0ff; }
        .modal-content button { width: 100%; padding: 12px; margin-top: 15px; background: #00f0ff; border: none; font-weight: bold; color: #000; border-radius: 8px; cursor: pointer; }
    </style>
    <!-- Three.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>
    <div id="canvas-container"></div>

    <div class="ui-layer">
        <div class="header">
            <h1>NOCTORA CITY</h1>
            <p>Интерактивная 3D-сеть каналов</p>
        </div>
        <button class="btn-main interactive" onclick="openModal()">+ Создать свой канал</button>
    </div>

    <!-- Modal Window -->
    <div class="modal interactive" id="channelModal">
        <div class="modal-content">
            <h2>Регистрация канала</h2>
            <input type="text" id="channelName" placeholder="Название канала">
            <input type="text" id="instaNick" placeholder="Ваш ник в Instagram (@nick)">
            <button onclick="createChannel()">Зажечь здание</button>
        </div>
    </div>

    <script>
        // Init Three.js Scene
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x05050a, 0.015);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(30, 20, 30);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.shadowMap.enabled = true;
        container.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.maxPolarAngle = Math.PI / 2.1;

        // Lights
        const ambientLight = new THREE.AmbientLight(0x222244, 1.5);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0x00f0ff, 1);
        dirLight.position.set(20, 40, 20);
        scene.add(dirLight);

        // Ground Grid
        const gridHelper = new THREE.GridHelper(100, 50, 0x00f0ff, 0x111122);
        gridHelper.position.y = -0.01;
        scene.add(gridHelper);

        // Generate City Blocks
        const buildings = [];
        const buildingGeometry = new THREE.BoxGeometry(2, 1, 2);

        function createCity() {
            for(let x = -20; x <= 20; x += 4) {
                for(let z = -20; z <= 20; z += 4) {
                    if(Math.random() > 0.3) {
                        const height = Math.random() * 8 + 2;
                        const material = new THREE.MeshStandardMaterial({
                            color: 0x111122,
                            roughness: 0.2,
                            metalness: 0.8
                        });
                        
                        const building = new THREE.Mesh(buildingGeometry, material);
                        building.scale.set(1.5, height, 1.5);
                        building.position.set(x, height / 2, z);
                        scene.add(building);
                        buildings.push(building);
                    }
                }
            }
        }
        createCity();

        // UI Interactions
        function openModal() {
            document.getElementById('channelModal').style.display = 'flex';
        }

        function createChannel() {
            const name = document.getElementById('channelName').value;
            const nick = document.getElementById('instaNick').value;
            
            if(name && nick) {
                // Select a random building to light up with neon
                const randomBuilding = buildings[Math.floor(Math.random() * buildings.length)];
                randomBuilding.material.color.setHex(0x00f0ff);
                randomBuilding.material.emissive.setHex(0x00f0ff);
                randomBuilding.material.emissiveIntensity = 0.8;
                
                alert(`Отлично! Канал "${name}" зарегистрирован на карте.`);
                document.getElementById('channelModal').style.display = 'none';
            } else {
                alert('Пожалуйста, заполните все поля!');
            }
        }

        // Animation Loop
        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }
        animate();

        // Resize Event
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
