<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mashina Online - Cobalt Simulator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Orbitron', sans-serif; background: #000; user-select: none; }
        
        #left-ui {
            position: absolute;
            bottom: 15px;
            left: 15px;
            z-index: 100;
            display: flex;
            flex-direction: column;
            gap: 12px;
            align-items: center;
        }

        /* Compact Pedals */
        .pedals-box {
            display: flex;
            gap: 8px;
            background: rgba(10, 10, 15, 0.95);
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #222;
            box-shadow: 0 0 15px rgba(0,0,0,0.5);
        }

        .pedal-wrap { text-align: center; color: #fff; font-size: 8px; font-weight: bold; opacity: 0.7; }
        .pedal-v {
            width: 14px;
            height: 55px;
            background: #050505;
            border: 1px solid #333;
            border-radius: 3px;
            position: relative;
            overflow: hidden;
            margin-bottom: 3px;
        }

        .fill {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 0%;
            transition: height 0.05s linear;
        }

        /* Compact H-Shifter (R on Right-Bottom) */
        .shifter-container {
            background: radial-gradient(circle, #1a1a1a 0%, #000 100%);
            border: 2px solid #333;
            border-radius: 12px;
            padding: 8px;
            display: grid;
            grid-template-columns: repeat(3, 32px);
            grid-template-rows: repeat(2, 32px);
            gap: 6px;
            box-shadow: 0 5px 20px rgba(0,0,0,0.9);
            position: relative;
        }

        .shifter-container::before {
            content: '';
            position: absolute;
            top: 50%; left: 10%; right: 10%; height: 1px; background: #222; transform: translateY(-50%);
            z-index: 0;
        }

        .gear-node {
            width: 32px;
            height: 32px;
            background: #111;
            border: 1px solid #444;
            color: #555;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            font-size: 11px;
            font-weight: 900;
            cursor: pointer;
            z-index: 1;
            transition: 0.1s;
        }

        .gear-node.active {
            background: #00d4ff;
            color: #000;
            border-color: #fff;
            box-shadow: 0 0 10px #00d4ff;
        }

        .neutral-indicator {
            position: absolute;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            color: #00d4ff; font-size: 10px; font-weight: bold;
            pointer-events: none; opacity: 0.4;
        }

        .neutral-active { opacity: 1; text-shadow: 0 0 5px #00d4ff; }

        /* Speedometer */
        #right-ui {
            position: absolute;
            bottom: 15px;
            right: 15px;
            z-index: 100;
        }

        .gauge-compact {
            width: 90px;
            height: 90px;
            background: #000;
            border: 2px solid #00d4ff;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: #fff;
            box-shadow: 0 0 15px rgba(0,212,255,0.2);
        }

        .speed-val { font-size: 22px; font-weight: 900; }
        .speed-unit { font-size: 7px; color: #00d4ff; letter-spacing: 1px; }

        #message-overlay {
            position: absolute;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.95);
            padding: 30px; border-radius: 15px; text-align: center;
            display: none; color: white; border: 2px solid #ff4444; z-index: 200;
        }

        .domain-badge {
            position: absolute;
            top: 15px;
            right: 15px;
            color: #00d4ff;
            font-size: 14px;
            font-weight: 900;
            text-shadow: 0 0 10px #00d4ff;
            letter-spacing: 1px;
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;900&display=swap" rel="stylesheet">
</head>
<body>

    <div class="domain-badge">MASHINA.ONLINE</div>

    <div id="left-ui">
        <div class="pedals-box">
            <div class="pedal-wrap">
                <div class="pedal-v"><div id="f-clutch" class="fill" style="background: #aaa;"></div></div>
                SHIFT
            </div>
            <div class="pedal-wrap">
                <div class="pedal-v"><div id="f-brake" class="fill" style="background: #ff4444;"></div></div>
                S
            </div>
            <div class="pedal-wrap">
                <div class="pedal-v"><div id="f-gas" class="fill" style="background: #00ff88;"></div></div>
                W
            </div>
        </div>

        <div class="shifter-container">
            <div id="neutral-label" class="neutral-indicator neutral-active">N</div>
            <!-- Top: 1-3-5 -->
            <div class="gear-node" id="g1">1</div>
            <div class="gear-node" id="g3">3</div>
            <div class="gear-node" id="g5">5</div>
            <!-- Bottom: 2-4-R -->
            <div class="gear-node" id="g2">2</div>
            <div class="gear-node" id="g4">4</div>
            <div class="gear-node" id="gr" style="color:#ff4444">R</div>
        </div>
    </div>

    <div id="right-ui">
        <div class="gauge-compact">
            <div class="speed-val" id="speed-num">0</div>
            <div class="speed-unit">KM/H</div>
        </div>
    </div>

    <div id="message-overlay">
        <h1>AVARIYA!</h1>
        <button onclick="location.reload()" style="background:#00d4ff; border:none; padding:10px 20px; cursor:pointer; font-family:'Orbitron'; font-weight:900; border-radius:5px;">QAYTADAN</button>
    </div>

    <script>
        let scene, camera, renderer, car;
        let speed = 0, angle = 0, gear = 0;
        let isClutch = false, isGas = false, isBrake = false, isLeft = false, isRight = false;
        let trafficCars = [];
        let gameActive = true;

        const GEAR_DATA = {
            0: { max: 0, torque: 0 },
            1: { max: 0.20, torque: 0.016 },
            2: { max: 0.38, torque: 0.012 },
            3: { max: 0.65, torque: 0.010 },
            4: { max: 0.90, torque: 0.008 },
            5: { max: 1.30, torque: 0.007 },
            "-1": { max: -0.25, torque: 0.013 }
        };

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x020205);
            scene.fog = new THREE.FogExp2(0x020205, 0.015);

            camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 3000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);

            const sun = new THREE.DirectionalLight(0xffffff, 1.2);
            sun.position.set(50, 100, 50);
            scene.add(sun);
            scene.add(new THREE.AmbientLight(0xffffff, 0.3));

            const ground = new THREE.Mesh(new THREE.PlaneGeometry(10000, 10000), new THREE.MeshStandardMaterial({ color: 0x080808 }));
            ground.rotation.x = -Math.PI / 2;
            scene.add(ground);
            scene.add(new THREE.GridHelper(5000, 300, 0x00d4ff, 0x111111));

            createCobalt();
            createTraffic();

            window.addEventListener('keydown', e => handleKey(e.code, true));
            window.addEventListener('keyup', e => handleKey(e.code, false));
            window.addEventListener('resize', onResize);

            animate();
        }

        function createCobalt() {
            car = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(2.2, 0.9, 4.6), new THREE.MeshStandardMaterial({ color: 0xffffff, metalness: 0.5 }));
            body.position.y = 0.6;
            car.add(body);
            
            const roof = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.6, 2.5), new THREE.MeshStandardMaterial({ color: 0xffffff }));
            roof.position.set(0, 1.3, -0.2);
            car.add(roof);

            car.position.set(0, 0, 10);
            scene.add(car);

            // Front Hood View
            camera.position.set(0, 2.0, 4.0); 
            camera.lookAt(0, 0.5, -10);
            car.add(camera);
        }

        function createTraffic() {
            for(let i = 0; i < 45; i++) {
                const tCar = new THREE.Mesh(new THREE.BoxGeometry(2.3, 1.2, 4.8), new THREE.MeshStandardMaterial({ color: 0x333333 }));
                tCar.position.set(Math.random() * 120 - 60, 0.6, Math.random() * -2000 - 100);
                tCar.userData.speed = Math.random() * 0.18 + 0.1;
                scene.add(tCar);
                trafficCars.push(tCar);
            }
        }

        function setGear(g) {
            if (!isClutch && speed !== 0) return;
            gear = g;
            document.querySelectorAll('.gear-node').forEach(n => n.classList.remove('active'));
            document.getElementById('neutral-label').classList.toggle('neutral-active', g === 0);
            if (g !== 0) {
                const id = g === -1 ? 'gr' : 'g' + g;
                const el = document.getElementById(id);
                if (el) el.classList.add('active');
            }
        }

        function handleKey(code, val) {
            if (code === 'KeyW') isGas = val;
            if (code === 'KeyS') isBrake = val;
            if (code === 'KeyA') isLeft = val;
            if (code === 'KeyD') isRight = val;
            if (code === 'ShiftLeft' || code === 'ShiftRight') isClutch = val;

            if (val && isClutch) {
                if (code === 'Digit1') setGear(1);
                if (code === 'Digit2') setGear(2);
                if (code === 'Digit3') setGear(3);
                if (code === 'Digit4') setGear(4);
                if (code === 'Digit5') setGear(5);
                if (code === 'KeyR') setGear(-1);
                if (code === 'Digit0') setGear(0);
            }
        }

        function animate() {
            requestAnimationFrame(animate);
            if (!gameActive) return;

            document.getElementById('f-clutch').style.height = isClutch ? '100%' : '0%';
            document.getElementById('f-gas').style.height = isGas ? '100%' : '0%';
            document.getElementById('f-brake').style.height = isBrake ? '100%' : '0%';

            if (!isClutch && gear !== 0) {
                const data = GEAR_DATA[gear];
                if (isGas) {
                    if (gear > 0) speed = Math.min(speed + data.torque, data.max);
                    else speed = Math.max(speed - data.torque, data.max);
                }
            }

            speed *= 0.996;
            if (isBrake) speed *= 0.85;

            if (Math.abs(speed) > 0.001) {
                const turn = (0.04 - Math.abs(speed)*0.015) * (speed > 0 ? 1 : -1);
                if (isLeft) angle += turn;
                if (isRight) angle -= turn;
            }

            car.rotation.y = angle;
            car.position.x += Math.sin(angle) * speed;
            car.position.z += Math.cos(angle) * speed;

            document.getElementById('speed-num').innerText = Math.round(Math.abs(speed) * 450);

            const carBox = new THREE.Box3().setFromObject(car);
            trafficCars.forEach(t => {
                t.position.z += t.userData.speed;
                if (t.position.z > car.position.z + 150) {
                    t.position.z = car.position.z - 1500;
                    t.position.x = car.position.x + (Math.random() * 140 - 70);
                }
                if (carBox.intersectsBox(new THREE.Box3().setFromObject(t))) {
                    gameActive = false;
                    document.getElementById('message-overlay').style.display = 'block';
                }
            });

            renderer.render(scene, camera);
        }

        function onResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        window.onload = init;
    </script>
</body>
</html>
