# Fnaf-yapay-zeka-oyunu-deneme
Bu denemedir ve özeldir
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <title>FNAF 3D – Final HTML Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Three.js -->
  <script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script>

  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: black;
      font-family: monospace;
    }
    #ui {
      position: fixed;
      top: 10px;
      left: 10px;
      color: white;
      z-index: 10;
      font-size: 14px;
    }
    #buttons {
      position: fixed;
      bottom: 15px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10px;
      z-index: 10;
    }
    button {
      padding: 10px 14px;
      font-size: 13px;
      font-weight: bold;
    }
    #cameraBtn {
      position: fixed;
      top: 10px;
      right: 10px;
      z-index: 10;
    }
    #gameover {
      position: fixed;
      inset: 0;
      background: black;
      color: red;
      display: none;
      align-items: center;
      justify-content: center;
      font-size: 36px;
      z-index: 20;
    }
  </style>
</head>

<body>
  <div id="ui">
    ⚡ Güç: <span id="power">100</span>%<br>
    📷 Kamera: <span id="camState">KAPALI</span>
  </div>

  <button id="cameraBtn">KAMERA</button>

  <div id="buttons">
    <button id="leftDoor">SOL KAPI</button>
    <button id="rightDoor">SAĞ KAPI</button>
  </div>

  <div id="gameover">OYUN BİTTİ</div>

  <script>
    /* ================== SAHNE ================== */
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x050505);
    scene.fog = new THREE.Fog(0x050505, 10, 90);

    const camera = new THREE.PerspectiveCamera(
      70,
      window.innerWidth / window.innerHeight,
      0.1,
      200
    );
    camera.position.set(0, 1.6, -6);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    /* ================== IŞIK ================== */
    scene.add(new THREE.AmbientLight(0x404040, 0.4));

    const officeLight = new THREE.PointLight(0xffaa55, 1.2, 10);
    officeLight.position.set(0, 3, -6);
    scene.add(officeLight);

    /* ================== ZEMİN ================== */
    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(100, 100),
      new THREE.MeshStandardMaterial({ color: 0x222222 })
    );
    floor.rotation.x = -Math.PI / 2;
    scene.add(floor);

    /* ================== OFİS ================== */
    const wallMat = new THREE.MeshStandardMaterial({ color: 0x3a3a3a });

    const backWall = new THREE.Mesh(
      new THREE.BoxGeometry(10, 4, 0.4),
      wallMat
    );
    backWall.position.set(0, 2, -8);
    scene.add(backWall);

    const leftWall = new THREE.Mesh(
      new THREE.BoxGeometry(0.4, 4, 6),
      wallMat
    );
    leftWall.position.set(-5, 2, -5);
    scene.add(leftWall);

    const rightWall = new THREE.Mesh(
      new THREE.BoxGeometry(0.4, 4, 6),
      wallMat
    );
    rightWall.position.set(5, 2, -5);
    scene.add(rightWall);

    /* ================== KORİDOR ================== */
    const corL = new THREE.Mesh(
      new THREE.BoxGeometry(0.4, 4, 50),
      wallMat
    );
    corL.position.set(-3, 2, 20);
    scene.add(corL);

    const corR = new THREE.Mesh(
      new THREE.BoxGeometry(0.4, 4, 50),
      wallMat
    );
    corR.position.set(3, 2, 20);
    scene.add(corR);

    /* ================== KAPILAR ================== */
    const doorMat = new THREE.MeshStandardMaterial({ color: 0x6b4b2a });

    const leftDoor = new THREE.Mesh(
      new THREE.BoxGeometry(2, 3, 0.2),
      doorMat
    );
    leftDoor.position.set(-4, 4.5, -2);
    scene.add(leftDoor);

    const rightDoor = new THREE.Mesh(
      new THREE.BoxGeometry(2, 3, 0.2),
      doorMat
    );
    rightDoor.position.set(4, 4.5, -2);
    scene.add(rightDoor);

    const DOOR_OPEN_Y = 4.5;
    const DOOR_CLOSED_Y = 1.5;

    let leftClosed = false;
    let rightClosed = false;

    /* ================== ANIMATRONIC ================== */
    const animatronic = new THREE.Mesh(
      new THREE.BoxGeometry(0.8, 2, 0.6),
      new THREE.MeshStandardMaterial({ color: 0x8b4513 })
    );
    animatronic.position.set(0, 1, 45);
    scene.add(animatronic);

    let targetZ = 45;

    /* ================== OYUN DURUMU ================== */
    let power = 100;
    let cameraOpen = false;
    let gameOver = false;

    /* ================== UI ================== */
    const powerEl = document.getElementById("power");
    const camStateEl = document.getElementById("camState");

    document.getElementById("leftDoor").onclick = () => leftClosed = !leftClosed;
    document.getElementById("rightDoor").onclick = () => rightClosed = !rightClosed;
    document.getElementById("cameraBtn").onclick = () => {
      cameraOpen = !cameraOpen;
      camStateEl.textContent = cameraOpen ? "AÇIK" : "KAPALI";
    };

    /* ================== ANA DÖNGÜ ================== */
    function animate() {
      if (gameOver) return;
      requestAnimationFrame(animate);

      leftDoor.position.y += ((leftClosed ? DOOR_CLOSED_Y : DOOR_OPEN_Y) - leftDoor.position.y) * 0.15;
      rightDoor.position.y += ((rightClosed ? DOOR_CLOSED_Y : DOOR_OPEN_Y) - rightDoor.position.y) * 0.15;

      if (!cameraOpen) {
        targetZ -= 0.012;
        animatronic.position.z += (targetZ - animatronic.position.z) * 0.01;
      }

      if (animatronic.position.z < -1) {
        document.getElementById("gameover").style.display = "flex";
        gameOver = true;
      }

      renderer.render(scene, camera);
    }
    animate();

    /* ================== GÜÇ ================== */
    setInterval(() => {
      if (gameOver) return;
      let drain = 0.15;
      if (leftClosed) drain += 0.4;
      if (rightClosed) drain += 0.4;
      if (cameraOpen) drain += 0.3;
      power = Math.max(0, power - drain);
      powerEl.textContent = power.toFixed(1);
      if (power <= 0) {
        document.getElementById("gameover").style.display = "flex";
        gameOver = true;
      }
    }, 1000);
  </script>
</body>
</html>
