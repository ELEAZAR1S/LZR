// Configuración de la escena
let scene, camera, renderer;
let player = {
    height: 1.7,
    speed: 0.1,
    jumpSpeed: 0.15,
    velocity: new THREE.Vector3(),
    canJump: false,
    position: new THREE.Vector3(0, 1.7, 4)
};

// Controles
let moveForward = false, moveBackward = false;
let moveLeft = false, moveRight = false;
let canJump = false;

const keys = {};
const clock = new THREE.Clock();

// Configuración de física
const gravity = -9.8;
const groundLevel = 0;

function init() {
    // Crear escena
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x1a1a1a);
    scene.fog = new THREE.Fog(0x1a1a1a, 1, 50);

    // Crear cámara
    camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
    );
    camera.position.copy(player.position);

    // Crear renderer
    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    // Iluminación
    const ambientLight = new THREE.AmbientLight(0x404040, 1);
    scene.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 0.8, 50);
    pointLight.position.set(0, 4, 0);
    pointLight.castShadow = true;
    scene.add(pointLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
    directionalLight.position.set(5, 10, 5);
    scene.add(directionalLight);

    // Crear habitación
    createRoom();

    // Event listeners
    document.addEventListener('keydown', onKeyDown);
    document.addEventListener('keyup', onKeyUp);
    document.addEventListener('click', onClick);
    window.addEventListener('resize', onWindowResize);

    // Pantalla de inicio
    const startButton = document.getElementById('startButton');
    startButton.addEventListener('click', startGame);
}

function createRoom() {
    const textureLoader = new THREE.TextureLoader();

    // Suelo
    const floorGeometry = new THREE.PlaneGeometry(10, 10);
    const floorMaterial = new THREE.MeshStandardMaterial({
        color: 0x4a3428,
        roughness: 0.8,
        metalness: 0.2
    });
    const floor = new THREE.Mesh(floorGeometry, floorMaterial);
    floor.rotation.x = -Math.PI / 2;
    floor.receiveShadow = true;
    scene.add(floor);

    // Paredes
    const wallMaterial = new THREE.MeshStandardMaterial({
        color: 0x8b8b8b,
        roughness: 0.7
    });

    // Pared trasera
    const backWall = new THREE.Mesh(
        new THREE.BoxGeometry(10, 5, 0.2),
        wallMaterial
    );
    backWall.position.set(0, 2.5, -5);
    backWall.receiveShadow = true;
    scene.add(backWall);

    // Pared izquierda
    const leftWall = new THREE.Mesh(
        new THREE.BoxGeometry(0.2, 5, 10),
        wallMaterial
    );
    leftWall.position.set(-5, 2.5, 0);
    leftWall.receiveShadow = true;
    scene.add(leftWall);

    // Pared derecha
    const rightWall = new THREE.Mesh(
        new THREE.BoxGeometry(0.2, 5, 10),
        wallMaterial
    );
    rightWall.position.set(5, 2.5, 0);
    rightWall.receiveShadow = true;
    scene.add(rightWall);

    // Techo
    const ceilingGeometry = new THREE.PlaneGeometry(10, 10);
    const ceilingMaterial = new THREE.MeshStandardMaterial({
        color: 0xa0a0a0,
        roughness: 0.5,
        side: THREE.DoubleSide
    });
    const ceiling = new THREE.Mesh(ceilingGeometry, ceilingMaterial);
    ceiling.rotation.x = Math.PI / 2;
    ceiling.position.y = 5;
    scene.add(ceiling);

    // Cama
    const bedFrame = new THREE.Mesh(
        new THREE.BoxGeometry(2, 0.5, 3),
        new THREE.MeshStandardMaterial({ color: 0x3d2817 })
    );
    bedFrame.position.set(2.5, 0.25, -3);
    bedFrame.castShadow = true;
    scene.add(bedFrame);

    const mattress = new THREE.Mesh(
        new THREE.BoxGeometry(2, 0.3, 3),
        new THREE.MeshStandardMaterial({ color: 0xcccccc })
    );
    mattress.position.set(2.5, 0.65, -3);
    mattress.castShadow = true;
    scene.add(mattress);

    // Mesa de noche
    const nightstand = new THREE.Mesh(
        new THREE.BoxGeometry(0.8, 1, 0.8),
        new THREE.MeshStandardMaterial({ color: 0x4a3428 })
    );
    nightstand.position.set(-2, 0.5, -3.5);
    nightstand.castShadow = true;
    scene.add(nightstand);

    // Cuadros en la pared
    const frameMaterial = new THREE.MeshStandardMaterial({ color: 0x1a1a1a });
    
    const frame1 = new THREE.Mesh(
        new THREE.BoxGeometry(1, 1.5, 0.1),
        frameMaterial
    );
    frame1.position.set(-3, 2, -4.9);
    scene.add(frame1);

    const frame2 = new THREE.Mesh(
        new THREE.BoxGeometry(1.5, 1, 0.1),
        frameMaterial
    );
    frame2.position.set(0, 2, -4.9);
    scene.add(frame2);

    const frame3 = new THREE.Mesh(
        new THREE.BoxGeometry(1, 1.5, 0.1),
        frameMaterial
    );
    frame3.position.set(3, 2, -4.9);
    scene.add(frame3);

    // Guitarra en la pared
    const guitarBody = new THREE.Mesh(
        new THREE.BoxGeometry(0.4, 0.7, 0.1),
        new THREE.MeshStandardMaterial({ color: 0x8b4513 })
    );
    guitarBody.position.set(3.5, 2.5, -4.9);
    scene.add(guitarBody);

    const guitarNeck = new THREE.Mesh(
        new THREE.BoxGeometry(0.1, 1.2, 0.05),
        new THREE.MeshStandardMaterial({ color: 0x654321 })
    );
    guitarNeck.position.set(3.5, 3.5, -4.9);
    scene.add(guitarNeck);

    // Lámpara
    const lampPost = new THREE.Mesh(
        new THREE.CylinderGeometry(0.05, 0.05, 1.5),
        new THREE.MeshStandardMaterial({ color: 0x333333 })
    );
    lampPost.position.set(0, 4.25, 0);
    scene.add(lampPost);

    const lampShade = new THREE.Mesh(
        new THREE.SphereGeometry(0.3, 16, 16),
        new THREE.MeshStandardMaterial({
            color: 0xffffdd,
            emissive: 0xffffaa,
            emissiveIntensity: 0.5
        })
    );
    lampShade.position.set(0, 3.5, 0);
    scene.add(lampShade);
}

function startGame() {
    document.getElementById('startScreen').style.display = 'none';
    
    // Pointer lock
    renderer.domElement.requestPointerLock();
    
    document.addEventListener('mousemove', onMouseMove);
    animate();
}

function onMouseMove(event) {
    if (document.pointerLockElement === renderer.domElement) {
        const movementX = event.movementX || 0;
        const movementY = event.movementY || 0;

        camera.rotation.y -= movementX * 0.002;
        camera.rotation.x -= movementY * 0.002;

        // Limitar rotación vertical
        camera.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, camera.rotation.x));
    }
}

function onClick() {
    if (document.getElementById('startScreen').style.display === 'none') {
        renderer.domElement.requestPointerLock();
    }
}

function onKeyDown(event) {
    keys[event.code] = true;

    switch (event.code) {
        case 'KeyW': moveForward = true; break;
        case 'KeyS': moveBackward = true; break;
        case 'KeyA': moveLeft = true; break;
        case 'KeyD': moveRight = true; break;
        case 'Space':
            if (player.canJump) {
                player.velocity.y = player.jumpSpeed;
                player.canJump = false;
            }
            break;
        case 'Escape':
            document.exitPointerLock();
            break;
    }
}

function onKeyUp(event) {
    keys[event.code] = false;

    switch (event.code) {
        case 'KeyW': moveForward = false; break;
        case 'KeyS': moveBackward = false; break;
        case 'KeyA': moveLeft = false; break;
        case 'KeyD': moveRight = false; break;
    }
}

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
}

function updatePlayer(delta) {
    // Aplicar gravedad
    player.velocity.y += gravity * delta;

    // Movimiento
    const direction = new THREE.Vector3();
    
    if (moveForward) direction.z -= 1;
    if (moveBackward) direction.z += 1;
    if (moveLeft) direction.x -= 1;
    if (moveRight) direction.x += 1;

    direction.normalize();

    // Aplicar rotación de la cámara al movimiento
    const rotatedDirection = direction.applyAxisAngle(
        new THREE.Vector3(0, 1, 0),
        camera.rotation.y
    );

    camera.position.x += rotatedDirection.x * player.speed;
    camera.position.z += rotatedDirection.z * player.speed;
    camera.position.y += player.velocity.y;

    // Colisión con el suelo
    if (camera.position.y <= player.height) {
        camera.position.y = player.height;
        player.velocity.y = 0;
        player.canJump = true;
    }

    // Límites de la habitación
    camera.position.x = Math.max(-4.5, Math.min(4.5, camera.position.x));
    camera.position.z = Math.max(-4.5, Math.min(4.5, camera.position.z));
}

function animate() {
    requestAnimationFrame(animate);

    const delta = clock.getDelta();
    updatePlayer(delta);

    renderer.render(scene, camera);
}

// Inicializar cuando cargue la página
window.addEventListener('load', init);
