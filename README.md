<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, shrink-to-fit=no">
    <title>Adit Futsal 3D</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            touch-action: pan-y pinch-zoom;
        }
        #info {
            position: absolute;
            top: 15px;
            left: 0;
            width: 100%;
            text-align: center;
            color: white;
            text-shadow: 0 2px 10px rgba(0,0,0,0.8);
            pointer-events: none;
            z-index: 10;
            font-size: 1.2rem;
            letter-spacing: 2px;
        }
        #info h1 {
            margin: 0;
            font-weight: 800;
            background: linear-gradient(45deg, #ffcc00, #ff8800);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 0 20px rgba(255, 200, 0, 0.3);
            font-size: 2.2rem;
        }
        #info small {
            display: block;
            color: #eee;
            -webkit-text-fill-color: #eee;
            background: none;
            font-weight: 300;
            font-size: 0.8rem;
            margin-top: -5px;
        }
        #controls {
            position: absolute;
            bottom: 25px;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            box-sizing: border-box;
            z-index: 20;
            pointer-events: none;
        }
        .ctrl-btn {
            background: rgba(30, 30, 40, 0.7);
            backdrop-filter: blur(4px);
            border: 2px solid rgba(255, 255, 255, 0.3);
            color: white;
            font-size: 1.8rem;
            font-weight: bold;
            width: 70px;
            height: 70px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            pointer-events: auto;
            box-shadow: 0 8px 20px rgba(0,0,0,0.5);
            transition: 0.1s ease;
            user-select: none;
            touch-action: manipulation;
        }
        .ctrl-btn:active {
            transform: scale(0.85);
            background: rgba(255, 200, 0, 0.3);
            border-color: #ffcc00;
        }
        #left-btn {
            margin-right: auto;
        }
        #right-btn {
            margin-left: auto;
        }
        #shoot-btn {
            background: rgba(255, 80, 80, 0.7);
            border-color: #ff4444;
            width: 80px;
            height: 80px;
            font-size: 1.2rem;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        @media (max-width: 480px) {
            .ctrl-btn { width: 60px; height: 60px; font-size: 1.5rem; }
            #shoot-btn { width: 70px; height: 70px; font-size: 1rem; }
            #info h1 { font-size: 1.8rem; }
        }
        #scoreboard {
            position: absolute;
            top: 80px;
            right: 15px;
            background: rgba(0,0,0,0.6);
            backdrop-filter: blur(4px);
            padding: 8px 16px;
            border-radius: 30px;
            color: white;
            font-weight: bold;
            font-size: 1.2rem;
            border: 1px solid rgba(255,255,255,0.2);
            z-index: 15;
            letter-spacing: 1px;
            pointer-events: none;
        }
        #scoreboard span {
            color: #ffcc00;
        }
    </style>
</head>
<body>
    <div id="info">
        <h1>⚽ ADIT FUTSAL</h1>
        <small>geser kiri/kanan • tembak untuk gol!</small>
    </div>
    <div id="scoreboard">⚽ <span id="goal-count">0</span></div>

    <div id="controls">
        <div id="left-btn" class="ctrl-btn">◀</div>
        <div id="shoot-btn" class="ctrl-btn">🔥</div>
        <div id="right-btn" class="ctrl-btn">▶</div>
    </div>

    <!-- Three.js -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'https://unpkg.com/three@0.128.0/examples/jsm/controls/OrbitControls.js';

        // --- Setup Scene ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x1a2a3a);
        
        // --- Camera (lebih rendah untuk tampilan 3D) ---
        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 5, 12);
        camera.lookAt(0, 0, 0);

        // --- Renderer ---
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        // --- Lighting ---
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);

        const mainLight = new THREE.DirectionalLight(0xffeedd, 1);
        mainLight.position.set(5, 12, 8);
        mainLight.castShadow = true;
        mainLight.shadow.mapSize.width = 1024;
        mainLight.shadow.mapSize.height = 1024;
        scene.add(mainLight);

        const fillLight = new THREE.DirectionalLight(0x4466ff, 0.3);
        fillLight.position.set(-5, 3, 5);
        scene.add(fillLight);
        
        const backLight = new THREE.PointLight(0x3355aa, 0.2);
        backLight.position.set(0, 2, -8);
        scene.add(backLight);

        // --- Lapangan Futsal (dengan tekstur sederhana) ---
        const fieldMat = new THREE.MeshStandardMaterial({ color: 0x2a8c4a, roughness: 0.7 });
        const field = new THREE.Mesh(new THREE.PlaneGeometry(10, 6), fieldMat);
        field.rotation.x = -Math.PI / 2;
        field.position.y = -0.01;
        field.receiveShadow = true;
        scene.add(field);

        // Garis lapangan (menggunakan box tipis)
        const lineMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
        function addLine(w, h, x, z) {
            const line = new THREE.Mesh(new THREE.BoxGeometry(w, 0.02, h), lineMat);
            line.position.set(x, 0, z);
            scene.add(line);
        }
        // Garis tengah
        addLine(0.1, 4.5, 0, 0);
        // Lingkaran tengah (hanya garis luar, pakai torus)
        const circleLine = new THREE.Mesh(new THREE.TorusGeometry(1.2, 0.05, 8, 20), lineMat);
        circleLine.rotation.x = Math.PI / 2;
        circleLine.position.set(0, 0.01, 0);
        scene.add(circleLine);
        // Garis gawang (kotak kecil)
        addLine(2.5, 0.1, 0, -2.7);
        addLine(2.5, 0.1, 0, 2.7);
        addLine(0.1, 0.6, -1.2, -2.7);
        addLine(0.1, 0.6, 1.2, -2.7);
        addLine(0.1, 0.6, -1.2, 2.7);
        addLine(0.1, 0.6, 1.2, 2.7);

        // --- Gawang (3D) ---
        function createGoal(x, z, rotY) {
            const group = new THREE.Group();
            const postMat = new THREE.MeshStandardMaterial({ color: 0xffaa33, emissive: 0x442200 });
            const netMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, wireframe: true, transparent: true, opacity: 0.25 });
            
            // Tiang
            const tiang = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.8, 0.12), postMat);
            tiang.position.set(0, 0.4, 0);
            group.add(tiang);
            
            const tiang2 = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.8, 0.12), postMat);
            tiang2.position.set(0, 0.4, 1.2);
            group.add(tiang2);
            
            // Mistar
            const mistar = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.12, 1.2), postMat);
            mistar.position.set(0, 0.8, 0.6);
            group.add(mistar);
            
            // Jaring (kotak transparan)
            const net = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.7, 1.1), netMat);
            net.position.set(0, 0.4, 0.6);
            group.add(net);
            
            // Jaring samping
            const netSide1 = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.7, 0.02), netMat);
            netSide1.position.set(0, 0.4, 0);
            group.add(netSide1);
            const netSide2 = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.7, 0.02), netMat);
            netSide2.position.set(0, 0.4, 1.2);
            group.add(netSide2);
            
            group.position.set(x, 0, z);
            group.rotation.y = rotY;
            return group;
        }

        const goalKiri = createGoal(-4.8, 0, 0);
        scene.add(goalKiri);
        const goalKanan = createGoal(4.8, 0, Math.PI);
        scene.add(goalKanan);

        // --- Bola ---
        const ballMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3, emissive: 0x222222 });
        const ball = new THREE.Mesh(new THREE.SphereGeometry(0.35, 24, 24), ballMat);
        ball.castShadow = true;
        ball.receiveShadow = true;
        ball.position.set(0, 0.35, 0);
        scene.add(ball);

        // Pola bola (garis hitam)
        const stripeMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
        for (let i = 0; i < 5; i++) {
            const stripe = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.02, 0.45), stripeMat);
            stripe.position.set(0, 0, 0);
            ball.add(stripe);
        }

        // --- Pemain (Adit) ---
        function createPlayer(color, x, z, name = '') {
            const group = new THREE.Group();
            
            // Badan
            const bodyMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.4 });
            const body = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.4, 0.6, 8), bodyMat);
            body.position.y = 0.5;
            body.castShadow = true;
            group.add(body);
            
            // Kepala
            const headMat = new THREE.MeshStandardMaterial({ color: 0xffccaa });
            const head = new THREE.Mesh(new THREE.SphereGeometry(0.22, 16, 16), headMat);
            head.position.y = 0.9;
            head.castShadow = true;
            group.add(head);
            
            // Kaki
            const legMat = new THREE.MeshStandardMaterial({ color: 0x2255aa });
            const leg1 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 0.3, 6), legMat);
            leg1.position.set(-0.15, 0.15, 0);
            group.add(leg1);
            const leg2 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 0.3, 6), legMat);
            leg2.position.set(0.15, 0.15, 0);
            group.add(leg2);
            
            // Tangan (opsional)
            const armMat = new THREE.MeshStandardMaterial({ color: 0xffccaa });
            const arm1 = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.3, 6), armMat);
            arm1.position.set(-0.4, 0.5, 0);
            arm1.rotation.z = 0.3;
            group.add(arm1);
            const arm2 = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.3, 6), armMat);
            arm2.position.set(0.4, 0.5, 0);
            arm2.rotation.z = -0.3;
            group.add(arm2);
            
            // Nama (menggunakan sprite atau label sederhana tidak mudah, kita gunakan box kecil)
            if (name) {
                const labelMat = new THREE.MeshStandardMaterial({ color: 0xffdd44 });
                const label = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.05, 0.1), labelMat);
                label.position.set(0, 1.1, 0);
                group.add(label);
            }
            
            group.position.set(x, 0, z);
            return group;
        }

        // Pemain utama (Adit) - warna kuning
        const adit = createPlayer(0xffaa00, 0, 1.2, 'Adit');
        scene.add(adit);

        // Pemain lawan (merah)
        const lawan1 = createPlayer(0xcc3333, -0.8, -1.0);
        scene.add(lawan1);
        const lawan2 = createPlayer(0xcc3333, 0.8, -0.8);
        scene.add(lawan2);
        const lawan3 = createPlayer(0xcc3333, -1.5, -0.2);
        scene.add(lawan3);
        const lawan4 = createPlayer(0xcc3333, 1.5, -0.3);
        scene.add(lawan4);

        // Pemain teman (biru) - statis
        const teman1 = createPlayer(0x3366ff, -1.2, 0.8);
        scene.add(teman1);
        const teman2 = createPlayer(0x3366ff, 1.2, 0.7);
        scene.add(teman2);

        // --- Variabel Game ---
        let goals = 0;
        document.getElementById('goal-count').textContent = goals;
        const playerSpeed = 0.12;
        const ballSpeed = 0.15;
        let moveLeft = false;
        let moveRight = false;
        let shootCooldown = false;

        // --- Kontrol Touch/Keyboard ---
        // Keyboard
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a') moveLeft = true;
            if (e.key === 'ArrowRight' || e.key === 'd') moveRight = true;
            if (e.key === ' ' || e.key === 'f') { e.preventDefault(); shoot(); }
        });
        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a') moveLeft = false;
            if (e.key === 'ArrowRight' || e.key === 'd') moveRight = false;
        });

        // Touch (HP)
        const leftBtn = document.getElementById('left-btn');
        const rightBtn = document.getElementById('right-btn');
        const shootBtn = document.getElementById('shoot-btn');

        leftBtn.addEventListener('touchstart', (e) => { e.preventDefault(); moveLeft = true; });
        leftBtn.addEventListener('touchend', (e) => { e.preventDefault(); moveLeft = false; });
        leftBtn.addEventListener('touchcancel', (e) => { moveLeft = false; });
        
        rightBtn.addEventListener('touchstart', (e) => { e.preventDefault(); moveRight = true; });
        rightBtn.addEventListener('touchend', (e) => { e.preventDefault(); moveRight = false; });
        rightBtn.addEventListener('touchcancel', (e) => { moveRight = false; });

        shootBtn.addEventListener('touchstart', (e) => { e.preventDefault(); shoot(); });
        
        // Mouse fallback
        leftBtn.addEventListener('mousedown', () => moveLeft = true);
        leftBtn.addEventListener('mouseup', () => moveLeft = false);
        rightBtn.addEventListener('mousedown', () => moveRight = true);
        rightBtn.addEventListener('mouseup', () => moveRight = false);
        shootBtn.addEventListener('click', shoot);

        // --- Fungsi Tembak ---
        function shoot() {
            if (shootCooldown) return;
            shootCooldown = true;
            setTimeout(() => { shootCooldown = false; }, 400);

            // Arah tembak berdasarkan posisi adit
            const dir = new THREE.Vector3(0, 0, -1); // ke arah gawang lawan (z negatif)
            // Tambahkan sedikit acak agar menarik
            const randomOffset = (Math.random() - 0.5) * 0.15;
            const targetX = adit.position.x + randomOffset;
            
            // Animasi bola
            const startPos = ball.position.clone();
            const endPos = new THREE.Vector3(targetX, 0.35, -3.8); // mendekati gawang
            
            // Sederhana: langsung set posisi dengan gerak cepat
            ball.position.copy(endPos);
            
            // Cek gol (jika bola masuk gawang)
            if (Math.abs(ball.position.x) < 1.0 && ball.position.z < -2.5) {
                goals++;
                document.getElementById('goal-count').textContent = goals;
                // Reset bola
                setTimeout(() => {
                    ball.position.set(0, 0.35, 0);
                }, 300);
            } else {
                // Kembalikan bola ke posisi awal setelah 0.5 detik
                setTimeout(() => {
                    ball.position.set(0, 0.35, 0);
                }, 500);
            }

            // Efek visual: getar kamera ringan (opsional)
        }

        // --- Game Loop ---
        function animate() {
            requestAnimationFrame(animate);

            // Gerakan Adit (kiri/kanan)
            if (moveLeft && adit.position.x > -4.2) {
                adit.position.x -= playerSpeed;
                adit.rotation.y = 0.2;
            }
            if (moveRight && adit.position.x < 4.2) {
                adit.position.x += playerSpeed;
                adit.rotation.y = -0.2;
            }
            // Animasi idle
            if (!moveLeft && !moveRight) {
                adit.rotation.y = 0;
            }

            // Bola mengikuti Adit (sedikit di depan)
            if (!shootCooldown) {
                ball.position.x = adit.position.x * 0.9;
                ball.position.z = adit.position.z + 0.25;
            }

            // Rotasi bola
            ball.rotation.x += 0.03;
            ball.rotation.z += 0.02;

            // Gerakan lawan sederhana (maju mundur)
            const time = Date.now() * 0.001;
            lawan1.position.z = -1.0 + Math.sin(time * 0.5) * 0.4;
            lawan2.position.z = -0.8 + Math.sin(time * 0.7 + 1) * 0.4;
            lawan3.position.x = -1.5 + Math.sin(time * 0.3) * 0.5;
            lawan4.position.x = 1.5 + Math.sin(time * 0.4 + 2) * 0.5;

            renderer.render(scene, camera);
        }

        animate();

        // --- Responsif ---
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // Mencegah scroll di HP
        document.addEventListener('touchmove', (e) => {
            if (e.target.closest('#controls')) e.preventDefault();
        }, { passive: false });

        console.log('⚽ Adit Futsal 3D siap!');
    </script>
</body>
</html>
