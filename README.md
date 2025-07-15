<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guardián del Nexo (Pequeño)</title>
    <!-- Tailwind CSS CDN para estilos rápidos y responsivos -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <style>
        /* Estilos personalizados para el juego */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #1a202c; /* Fondo oscuro */
            color: #e2e8f0; /* Texto claro */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden; /* Evita el scroll en el cuerpo */
        }
        #gameContainer {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background-color: #2d3748; /* Fondo del contenedor del juego */
            border-radius: 1rem; /* Bordes redondeados */
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.5);
            padding: 1.5rem;
            max-width: 500px; /* Ancho máximo para responsividad, ajustado para canvas más pequeño */
            width: 100%;
            box-sizing: border-box;
        }
        canvas {
            background-color: #000; /* Fondo del canvas */
            border: 2px solid #4a5568;
            border-radius: 0.5rem;
            display: block;
            touch-action: none; /* Deshabilita el desplazamiento táctil predeterminado */
            cursor: crosshair; /* Cursor para indicar zona de construcción */
        }
        .game-info {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 480px; /* Ancho máximo para la información del juego, ajustado para canvas más pequeño */
            margin-bottom: 1rem;
            font-size: 1.125rem; /* text-lg */
            font-weight: bold;
        }
        .game-controls {
            display: flex;
            gap: 1rem;
            margin-top: 1.5rem;
            flex-wrap: wrap; /* Permite que los botones se envuelvan en pantallas pequeñas */
            justify-content: center;
        }
        .game-button {
            background-color: #4299e1; /* bg-blue-500 */
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem; /* rounded-xl */
            font-weight: bold;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.2s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
            border: none;
            outline: none;
            text-align: center;
        }
        .game-button:hover {
            background-color: #3182ce; /* bg-blue-600 */
            transform: translateY(-2px);
        }
        .game-button:active {
            background-color: #2b6cb0; /* bg-blue-700 */
            transform: translateY(0);
        }

        /* Estilos para el mensaje modal */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            background-color: #2d3748;
            padding: 1.5rem; /* Reduced padding */
            border-radius: 1rem;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.7);
            text-align: center;
            max-width: 400px; /* Reduced max-width */
            width: 90%;
            color: #e2e8f0;
            position: relative;
        }
        .modal-content h2 {
            font-size: 1.8rem; /* Slightly reduced font size */
            margin-bottom: 0.8rem; /* Reduced margin */
            color: #4299e1;
        }
        .modal-content p {
            font-size: 1rem; /* Slightly reduced font size */
            margin-bottom: 1rem; /* Reduced margin */
            line-height: 1.4; /* Adjusted line height */
        }
        .modal-content .game-button {
            margin-top: 0.8rem; /* Reduced margin */
            padding: 0.6rem 1.2rem; /* Reduced padding */
            font-size: 0.9rem; /* Reduced font size */
        }

        /* Estilos para el menú de construcción */
        .build-menu {
            display: flex;
            gap: 0.5rem;
            margin-top: 1rem;
            flex-wrap: wrap;
            justify-content: center;
        }
        .build-button {
            background-color: #63b3ed; /* light blue */
            color: white;
            padding: 0.5rem 1rem;
            border-radius: 0.5rem;
            font-weight: bold;
            cursor: pointer;
            transition: background-color 0.3s ease;
            border: none;
            outline: none;
            text-align: center;
            font-size: 0.9rem;
        }
        .build-button:hover {
            background-color: #4299e1;
        }
        .build-button:disabled {
            background-color: #4a5568;
            cursor: not-allowed;
        }
        .build-button.selected {
            border: 2px solid #f6e05e; /* yellow-300 */
            box-shadow: 0 0 8px #f6e05e;
        }

        /* Estilos para la barra de vida del Nexo */
        #nexusHealthBarContainer {
            width: 100%;
            max-width: 480px; /* Ajustado para canvas más pequeño */
            background-color: #4a5568;
            border-radius: 0.5rem;
            height: 20px;
            margin-top: 1rem;
            overflow: hidden;
            border: 1px solid #718096;
        }
        #nexusHealthBar {
            height: 100%;
            width: 100%; /* Initial width */
            background-color: #48bb78; /* green-500 */
            transition: width 0.2s ease-out;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 0.8rem;
            font-weight: bold;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
        }
    </style>
    <!-- Tone.js para efectos de sonido simples -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
</head>
<body>
    <div id="gameContainer" class="flex flex-col items-center justify-center p-6 rounded-2xl shadow-xl bg-gray-800 text-gray-100">
        <h1 class="text-3xl md:text-4xl font-bold mb-4 text-green-400">Guardián del Nexo</h1>

        <div class="game-info w-full max-w-xl mb-4 text-lg font-bold text-gray-200">
            <span>Oleada: <span id="waveDisplay">0</span></span>
            <span>Energía: <span id="energyDisplay">0</span></span>
        </div>

        <div id="nexusHealthBarContainer">
            <div id="nexusHealthBar">Nexo: 100%</div>
        </div>

        <canvas id="gameCanvas" width="480" height="360" class="rounded-lg mt-4"></canvas>

        <div class="build-menu">
            <button id="buildTurretButton" class="build-button">Torre Básica (50 Energía)</button>
            <button id="upgradeTurretButton" class="build-button" style="display: none;">Mejorar Torre (30 Energía)</button>
            <button id="startWaveButton" class="game-button mt-4">Iniciar Oleada</button>
        </div>

        <div class="game-controls">
            <button id="startButton" class="game-button">Iniciar Juego</button>
            <button id="restartButton" class="game-button" style="display: none;">Reiniciar Juego</button>
            <button id="instructionsButton" class="game-button">Instrucciones</button>
        </div>
    </div>

    <!-- Modal de mensaje -->
    <div id="messageModal" class="modal-overlay hidden">
        <div class="modal-content">
            <h2 id="modalTitle"></h2>
            <p id="modalMessage"></p>
            <button id="modalCloseButton" class="game-button">Cerrar</button>
        </div>
    </div>

    <script>
        // Inicialización de Tone.js para sonidos
        const enemyHitSynth = new Tone.Synth().toDestination();
        const buildSynth = new Tone.Synth().toDestination();
        const upgradeSynth = new Tone.Synth().toDestination();
        const waveStartSynth = new Tone.Synth().toDestination();
        const gameOverSynth = new Tone.Synth().toDestination();
        const nexusHitSynth = new Tone.Synth().toDestination();

        // Configuración de sonidos
        enemyHitSynth.oscillator.type = "sine";
        enemyHitSynth.envelope.attack = 0.01;
        enemyHitSynth.envelope.decay = 0.1;
        enemyHitSynth.envelope.sustain = 0.1;
        enemyHitSynth.envelope.release = 0.2;

        buildSynth.oscillator.type = "triangle";
        buildSynth.envelope.attack = 0.02;
        buildSynth.envelope.decay = 0.1;
        buildSynth.envelope.sustain = 0.1;
        buildSynth.envelope.release = 0.2;

        upgradeSynth.oscillator.type = "sawtooth";
        upgradeSynth.envelope.attack = 0.03;
        upgradeSynth.envelope.decay = 0.15;
        upgradeSynth.envelope.sustain = 0.1;
        upgradeSynth.envelope.release = 0.3;

        waveStartSynth.oscillator.type = "square";
        waveStartSynth.envelope.attack = 0.05;
        waveStartSynth.envelope.decay = 0.2;
        waveStartSynth.envelope.sustain = 0.1;
        waveStartSynth.envelope.release = 0.4;

        gameOverSynth.oscillator.type = "sawtooth";
        gameOverSynth.envelope.attack = 0.1;
        gameOverSynth.envelope.decay = 0.5;
        gameOverSynth.envelope.sustain = 0.3;
        gameOverSynth.envelope.release = 1;

        nexusHitSynth.oscillator.type = "fmsine";
        nexusHitSynth.envelope.attack = 0.01;
        nexusHitSynth.envelope.decay = 0.1;
        nexusHitSynth.envelope.sustain = 0.1;
        nexusHitSynth.envelope.release = 0.2;


        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Elementos de la UI
        const waveDisplay = document.getElementById('waveDisplay');
        const energyDisplay = document.getElementById('energyDisplay');
        const nexusHealthBarContainer = document.getElementById('nexusHealthBarContainer');
        const nexusHealthBar = document.getElementById('nexusHealthBar');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const instructionsButton = document.getElementById('instructionsButton');
        const messageModal = document.getElementById('messageModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalCloseButton = document.getElementById('modalCloseButton');
        const buildTurretButton = document.getElementById('buildTurretButton');
        const upgradeTurretButton = document.getElementById('upgradeTurretButton');
        const startWaveButton = document.getElementById('startWaveButton');

        // Dimensiones del juego
        const TILE_SIZE = 40; // Tamaño de cada celda de la cuadrícula
        const GRID_COLS = canvas.width / TILE_SIZE; // 480 / 40 = 12
        const GRID_ROWS = canvas.height / TILE_SIZE; // 360 / 40 = 9

        // Variables del juego
        let nexus = { x: canvas.width / 2, y: canvas.height / 2, radius: TILE_SIZE * 0.8, color: 'cyan', health: 100, maxHealth: 100 };
        let defenders = [];
        let enemies = [];
        let projectiles = [];
        let energy = 100;
        let currentWave = 0;
        let gameRunning = false;
        let animationFrameId;
        let buildingMode = false; // true si el jugador está en modo construcción
        let selectedDefender = null; // La torre seleccionada para mejorar

        const DEFENDER_COST = 50;
        const UPGRADE_COST = 30;

        // Rutas de los enemigos (ejemplo simple, se pueden hacer más complejas)
        // Los enemigos aparecerán desde los bordes y se dirigirán al centro (Nexo)
        const spawnPoints = [
            { x: TILE_SIZE / 2, y: TILE_SIZE / 2, path: [{ x: TILE_SIZE / 2, y: TILE_SIZE / 2 }, { x: canvas.width / 2, y: canvas.height / 2 }] }, // Esquina superior izquierda
            { x: canvas.width - TILE_SIZE / 2, y: TILE_SIZE / 2, path: [{ x: canvas.width - TILE_SIZE / 2, y: TILE_SIZE / 2 }, { x: canvas.width / 2, y: canvas.height / 2 }] }, // Esquina superior derecha
            { x: TILE_SIZE / 2, y: canvas.height - TILE_SIZE / 2, path: [{ x: TILE_SIZE / 2, y: canvas.height - TILE_SIZE / 2 }, { x: canvas.width / 2, y: canvas.height / 2 }] }, // Esquina inferior izquierda
            { x: canvas.width - TILE_SIZE / 2, y: canvas.height - TILE_SIZE / 2, path: [{ x: canvas.width - TILE_SIZE / 2, y: canvas.height - TILE_SIZE / 2 }, { x: canvas.width / 2, y: canvas.height / 2 }] } // Esquina inferior derecha
        ];

        // Definición de las oleadas
        const waves = [
            { enemies: [{ type: 'basic', count: 5, delay: 500 }], interval: 2000 },
            { enemies: [{ type: 'basic', count: 8, delay: 400 }], interval: 1500 },
            { enemies: [{ type: 'basic', count: 10, delay: 300 }, { type: 'fast', count: 3, delay: 1000 }], interval: 1200 },
            { enemies: [{ type: 'basic', count: 12, delay: 250 }, { type: 'fast', count: 5, delay: 800 }, { type: 'armored', count: 2, delay: 2000 }], interval: 1000 },
            { enemies: [{ type: 'basic', count: 15, delay: 200 }, { type: 'fast', count: 7, delay: 600 }, { type: 'armored', count: 4, delay: 1500 }], interval: 800 },
            // Añade más oleadas para mayor dificultad
        ];

        // --- Clases de Entidades ---

        class Defender {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.size = TILE_SIZE * 0.7;
                this.color = 'green';
                this.range = TILE_SIZE * 4; // Radio de ataque
                this.damage = 10;
                this.fireRate = 60; // Frames entre disparos (60 fps = 1 disparo/seg)
                this.fireCooldown = 0;
                this.level = 1;
                this.cost = DEFENDER_COST;
                this.upgradeCost = UPGRADE_COST;
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x - this.size / 2, this.y - this.size / 2, this.size, this.size);

                // Dibujar cañón
                ctx.fillStyle = '#4a5568'; // Gris oscuro
                ctx.fillRect(this.x - 5, this.y - this.size / 2 - 5, 10, 10); // Parte superior

                // Dibujar nivel
                ctx.fillStyle = 'white';
                ctx.font = 'bold 12px Inter';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(this.level, this.x, this.y + this.size / 2 + 8);
            }

            update() {
                if (this.fireCooldown > 0) {
                    this.fireCooldown--;
                }

                if (this.fireCooldown <= 0) {
                    // Buscar enemigo más cercano dentro del rango
                    let targetEnemy = null;
                    let closestDistance = Infinity;

                    enemies.forEach(enemy => {
                        const dist = Math.sqrt(Math.pow(this.x - enemy.x, 2) + Math.pow(this.y - enemy.y, 2));
                        if (dist < this.range && dist < closestDistance) {
                            targetEnemy = enemy;
                            closestDistance = dist;
                        }
                    });

                    if (targetEnemy) {
                        // Disparar proyectil
                        projectiles.push(new Projectile(this.x, this.y, targetEnemy, this.damage));
                        enemyHitSynth.triggerAttackRelease("C4", "0.05"); // Sonido de disparo
                        this.fireCooldown = this.fireRate;
                    }
                }
            }

            upgrade() {
                if (energy >= this.upgradeCost) {
                    energy -= this.upgradeCost;
                    this.level++;
                    this.damage *= 1.5; // Aumenta el daño
                    this.fireRate = Math.max(10, this.fireRate - 10); // Aumenta la velocidad de disparo
                    this.range += TILE_SIZE; // Aumenta el rango
                    upgradeSynth.triggerAttackRelease("E5", "0.1"); // Sonido de mejora
                    updateUI();
                    return true;
                }
                return false;
            }
        }

        class Enemy {
            constructor(type, path) {
                this.type = type;
                this.path = path;
                this.pathIndex = 0;
                this.x = path[0].x;
                this.y = path[0].y;
                this.radius = TILE_SIZE * 0.3;
                this.color = 'orange';
                this.speed = 1;
                this.health = 50;
                this.maxHealth = 50;
                this.reward = 10; // Energía al ser derrotado

                if (type === 'fast') {
                    this.color = 'purple';
                    this.speed = 2;
                    this.health = 30;
                    this.maxHealth = 30;
                    this.reward = 15;
                } else if (type === 'armored') {
                    this.color = 'brown';
                    this.speed = 0.7;
                    this.health = 150;
                    this.maxHealth = 150;
                    this.reward = 30;
                }
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fill();

                // Dibujar barra de vida
                const healthBarWidth = this.radius * 2;
                const healthBarHeight = 5;
                const healthPercentage = this.health / this.maxHealth;
                ctx.fillStyle = 'red';
                ctx.fillRect(this.x - this.radius, this.y - this.radius - 10, healthBarWidth, healthBarHeight);
                ctx.fillStyle = 'lime';
                ctx.fillRect(this.x - this.radius, this.y - this.radius - 10, healthBarWidth * healthPercentage, healthBarHeight);
            }

            update() {
                const target = this.path[this.pathIndex];
                const dx = target.x - this.x;
                const dy = target.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);

                if (distance < this.speed) {
                    this.x = target.x;
                    this.y = target.y;
                    this.pathIndex++;
                } else {
                    this.x += (dx / distance) * this.speed;
                    this.y += (dy / distance) * this.speed;
                }

                // Si llega al final del camino (Nexo)
                if (this.pathIndex >= this.path.length) {
                    nexus.health -= 10; // Daño al Nexo
                    nexusHitSynth.triggerAttackRelease("D3", "0.1"); // Sonido de golpe al Nexo
                    updateUI();
                    return true; // Indica que el enemigo debe ser removido
                }
                return false;
            }
        }

        class Projectile {
            constructor(x, y, target, damage) {
                this.x = x;
                this.y = y;
                this.target = target;
                this.damage = damage;
                this.radius = 3;
                this.color = 'yellow';
                this.speed = 8;
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fill();
            }

            update() {
                if (!this.target || this.target.health <= 0) {
                    return true; // El objetivo ya no existe o está muerto, remover proyectil
                }

                const dx = this.target.x - this.x;
                const dy = this.target.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);

                if (distance < this.speed) {
                    // Colisión con el objetivo
                    this.target.health -= this.damage;
                    return true; // Indica que el proyectil debe ser removido
                } else {
                    this.x += (dx / distance) * this.speed;
                    this.y += (dy / distance) * this.speed;
                }
                return false;
            }
        }

        // --- Funciones del Juego ---

        // Función para mostrar el modal de mensaje
        function showMessageModal(title, message, onCloseCallback = null) {
            modalTitle.textContent = title;
            modalMessage.innerHTML = message; // Usar innerHTML para permitir etiquetas como <br>
            messageModal.classList.remove('hidden');

            modalCloseButton.onclick = () => {
                messageModal.classList.add('hidden');
                if (onCloseCallback) {
                    onCloseCallback();
                }
            };
        }

        // Función para dibujar todo en el canvas
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Limpiar canvas

            // Dibujar cuadrícula (opcional, para visualización de construcción)
            if (buildingMode) {
                ctx.strokeStyle = 'rgba(255, 255, 255, 0.2)';
                for (let i = 0; i <= GRID_COLS; i++) {
                    ctx.beginPath();
                    ctx.moveTo(i * TILE_SIZE, 0);
                    ctx.lineTo(i * TILE_SIZE, canvas.height);
                    ctx.stroke();
                }
                for (let i = 0; i <= GRID_ROWS; i++) {
                    ctx.beginPath();
                    ctx.moveTo(0, i * TILE_SIZE);
                    ctx.lineTo(canvas.width, i * TILE_SIZE);
                    ctx.stroke();
                }
            }

            // Dibujar defensores
            defenders.forEach(defender => defender.draw());

            // Dibujar Nexo
            ctx.fillStyle = nexus.color;
            ctx.beginPath();
            ctx.arc(nexus.x, nexus.y, nexus.radius, 0, Math.PI * 2);
            ctx.fill();

            // Dibujar enemigos
            enemies.forEach(enemy => enemy.draw());

            // Dibujar proyectiles
            projectiles.forEach(projectile => projectile.draw());
        }

        // Función para actualizar la lógica del juego
        function update() {
            if (!gameRunning) return;

            // Actualizar defensores
            defenders.forEach(defender => defender.update());

            // Actualizar enemigos
            enemies = enemies.filter(enemy => {
                if (enemy.update()) { // Si el enemigo llegó al Nexo
                    return false; // Remover enemigo
                }
                return true;
            });

            // Actualizar proyectiles
            projectiles = projectiles.filter(projectile => {
                if (projectile.update()) { // Si el proyectil golpeó o el objetivo desapareció
                    return false; // Remover proyectil
                }
                return true;
            });

            // Eliminar enemigos muertos y dar energía
            enemies = enemies.filter(enemy => {
                if (enemy.health <= 0) {
                    energy += enemy.reward;
                    updateUI();
                    return false; // Remover enemigo
                }
                return true;
            });

            // Comprobar Game Over
            if (nexus.health <= 0) {
                gameOverSynth.triggerAttackRelease("C2", "1"); // Sonido de Game Over
                gameOver("¡Game Over!", `Tu Nexo ha sido destruido. Alcanzaste la oleada ${currentWave}.`);
            }
        }

        // Bucle principal del juego
        function gameLoop() {
            update();
            draw();
            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // Inicia el juego
        function startGame() {
            if (!gameRunning) {
                gameRunning = true;
                resetGame(); // Asegurarse de que el juego esté en un estado limpio
                updateUI();
                gameLoop();
                startWaveButton.style.display = 'inline-block'; // Mostrar botón de iniciar oleada
            }
        }

        // Reinicia el juego completamente
        function resetGame() {
            cancelAnimationFrame(animationFrameId); // Detener cualquier bucle en curso
            nexus.health = nexus.maxHealth;
            defenders = [];
            enemies = [];
            projectiles = [];
            energy = 100;
            currentWave = 0;
            gameRunning = false;
            buildingMode = false;
            selectedDefender = null;
            updateUI();
            startButton.style.display = 'inline-block';
            restartButton.style.display = 'none';
            buildTurretButton.classList.remove('selected');
            upgradeTurretButton.style.display = 'none';
            startWaveButton.style.display = 'none'; // Ocultar hasta que el juego inicie
        }

        // Actualiza la interfaz de usuario
        function updateUI() {
            waveDisplay.textContent = currentWave;
            energyDisplay.textContent = energy;
            const healthPercentage = (nexus.health / nexus.maxHealth) * 100;
            nexusHealthBar.style.width = `${healthPercentage}%`;
            nexusHealthBar.style.backgroundColor = `hsl(${healthPercentage * 1.2}, 70%, 50%)`; // Color dinámico
            nexusHealthBar.textContent = `Nexo: ${Math.round(healthPercentage)}%`;

            // Habilitar/deshabilitar botón de construir
            buildTurretButton.disabled = energy < DEFENDER_COST;

            // Mostrar/ocultar botón de mejorar
            if (selectedDefender) {
                upgradeTurretButton.style.display = 'inline-block';
                upgradeTurretButton.disabled = energy < UPGRADE_COST;
            } else {
                upgradeTurretButton.style.display = 'none';
            }
        }

        // Game Over
        function gameOver(title, message) {
            gameRunning = false;
            cancelAnimationFrame(animationFrameId);
            showMessageModal(title, message, () => {
                resetGame();
            });
        }

        // --- Lógica de Oleadas ---
        let waveSpawnTimer = null;
        let enemiesSpawnedInWave = 0;
        let currentWaveEnemiesToSpawn = [];

        function startNextWave() {
            if (currentWave >= waves.length) {
                showMessageModal("¡Victoria!", `¡Has defendido el Nexo a través de todas las oleadas!<br>Puntuación final: ${energy}.`, () => {
                    resetGame();
                });
                return;
            }

            waveStartSynth.triggerAttackRelease("G4", "0.2"); // Sonido de inicio de oleada
            currentWave++;
            waveDisplay.textContent = currentWave;
            startWaveButton.style.display = 'none'; // Ocultar botón durante la oleada

            const waveData = waves[currentWave - 1];
            currentWaveEnemiesToSpawn = [];
            waveData.enemies.forEach(enemyType => {
                for (let i = 0; i < enemyType.count; i++) {
                    currentWaveEnemiesToSpawn.push({ type: enemyType.type, delay: enemyType.delay });
                }
            });
            // Mezclar enemigos para que no salgan todos del mismo tipo juntos
            currentWaveEnemiesToSpawn.sort(() => Math.random() - 0.5);

            enemiesSpawnedInWave = 0;
            spawnEnemiesSequentially(waveData.interval);
        }

        function spawnEnemiesSequentially(interval) {
            if (enemiesSpawnedInWave < currentWaveEnemiesToSpawn.length) {
                const enemyToSpawn = currentWaveEnemiesToSpawn[enemiesSpawnedInWave];
                const spawnPoint = spawnPoints[Math.floor(Math.random() * spawnPoints.length)];
                // Clonar el camino para cada enemigo
                const enemyPath = JSON.parse(JSON.stringify(spawnPoint.path));
                enemies.push(new Enemy(enemyToSpawn.type, enemyPath));
                enemiesSpawnedInWave++;
                waveSpawnTimer = setTimeout(() => spawnEnemiesSequentially(interval), interval);
            } else {
                // Todos los enemigos de la oleada han sido generados
                // El botón de iniciar oleada se mostrará cuando no queden enemigos en pantalla
            }
        }

        // Comprobar si la oleada ha terminado
        setInterval(() => {
            if (gameRunning && enemiesSpawnedInWave >= currentWaveEnemiesToSpawn.length && enemies.length === 0) {
                startWaveButton.style.display = 'inline-block'; // Mostrar botón para la siguiente oleada
                // Dar energía extra al final de la oleada
                energy += 50 + (currentWave * 10);
                updateUI();
            }
        }, 1000); // Comprueba cada segundo

        // --- Eventos de UI ---

        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', () => {
            resetGame();
            startGame();
        });
        instructionsButton.addEventListener('click', () => {
            showMessageModal(
                "Instrucciones: Guardián del Nexo",
                "¡Bienvenido a Guardián del Nexo!<br><br>" +
                "**Objetivo:** Protege el **Nexo (círculo cian)** de las oleadas de **Corruptores (esferas rojas/moradas/marrones)**.<br><br>" +
                "**Mecánicas:**<br>" +
                "- **Construcción:** Haz clic en el botón 'Torre Básica' y luego haz clic en el campo de juego para construir una torre.<br>" +
                "- **Mejora:** Haz clic en una torre existente para seleccionarla, luego haz clic en 'Mejorar Torre'.<br>" +
                "- **Energía:** Ganas Energía al derrotar enemigos. Úsala para construir y mejorar.<br>" +
                "- **Oleadas:** Haz clic en 'Iniciar Oleada' para que comiencen los enemigos.<br><br>" +
                "**Tipos de Corruptores:**<br>" +
                "- **Básico (Naranja):** Enemigo estándar.<br>" +
                "- **Rápido (Morado):** Menos vida, pero muy veloz.<br>" +
                "- **Blindado (Marrón):** Mucha vida, pero lento.<br><br>" +
                "¡Planifica tu defensa, gestiona tu energía y detén a los Corruptores!"
            );
        });

        buildTurretButton.addEventListener('click', () => {
            if (energy >= DEFENDER_COST) {
                buildingMode = true;
                buildTurretButton.classList.add('selected');
                selectedDefender = null; // Deseleccionar cualquier torre
                upgradeTurretButton.style.display = 'none';
            } else {
                showMessageModal("Energía Insuficiente", "Necesitas 50 de Energía para construir una Torre Básica.");
            }
        });

        upgradeTurretButton.addEventListener('click', () => {
            if (selectedDefender && selectedDefender.upgrade()) {
                // La mejora se maneja dentro del método upgrade del Defender
                // El botón se deshabilitará si no hay suficiente energía
            } else if (selectedDefender) {
                showMessageModal("Energía Insuficiente", `Necesitas ${UPGRADE_COST} de Energía para mejorar esta torre.`);
            }
        });

        startWaveButton.addEventListener('click', startNextWave);

        canvas.addEventListener('click', (e) => {
            if (!gameRunning) return;

            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;

            if (buildingMode) {
                const gridX = Math.floor(mouseX / TILE_SIZE);
                const gridY = Math.floor(mouseY / TILE_SIZE);
                const centerX = gridX * TILE_SIZE + TILE_SIZE / 2;
                const centerY = gridY * TILE_SIZE + TILE_SIZE / 2;

                // Evitar construir sobre el Nexo o en una posición ya ocupada
                const isOverlappingNexus = Math.sqrt(Math.pow(centerX - nexus.x, 2) + Math.pow(centerY - nexus.y, 2)) < (nexus.radius + TILE_SIZE / 2);
                const isOccupied = defenders.some(d => Math.abs(d.x - centerX) < TILE_SIZE && Math.abs(d.y - centerY) < TILE_SIZE);

                if (!isOverlappingNexus && !isOccupied) {
                    if (energy >= DEFENDER_COST) {
                        defenders.push(new Defender(centerX, centerY));
                        energy -= DEFENDER_COST;
                        buildSynth.triggerAttackRelease("A4", "0.1"); // Sonido de construcción
                        updateUI();
                        buildingMode = false;
                        buildTurretButton.classList.remove('selected');
                    } else {
                        showMessageModal("Energía Insuficiente", "No tienes suficiente energía para construir esta torre.");
                        buildingMode = false; // Salir del modo construcción
                        buildTurretButton.classList.remove('selected');
                    }
                } else {
                    showMessageModal("Posición Inválida", "No puedes construir aquí. La posición está ocupada o demasiado cerca del Nexo.");
                }
            } else {
                // Seleccionar torre para mejorar
                let clickedOnDefender = false;
                for (let i = 0; i < defenders.length; i++) {
                    const d = defenders[i];
                    const dist = Math.sqrt(Math.pow(mouseX - d.x, 2) + Math.pow(mouseY - d.y, 2));
                    if (dist < d.size / 2) {
                        selectedDefender = d;
                        clickedOnDefender = true;
                        upgradeTurretButton.style.display = 'inline-block';
                        upgradeTurretButton.disabled = energy < UPGRADE_COST;
                        break;
                    }
                }
                if (!clickedOnDefender) {
                    selectedDefender = null;
                    upgradeTurretButton.style.display = 'none';
                }
                updateUI();
            }
        });


        // Inicializar el juego al cargar la ventana
        window.onload = function() {
            updateUI(); // Actualizar la UI inicial
            draw(); // Dibujar el estado inicial
            // Mostrar instrucciones al cargar la página por primera vez
            instructionsButton.click();
        };

        // Responsividad del canvas (mantener proporciones y centrar)
        window.addEventListener('resize', () => {
            // No es necesario cambiar el tamaño del canvas, ya que el juego usa un tamaño fijo en píxeles
            // y el CSS lo escala. Solo aseguramos que el contenedor se adapte.
        });

    </script>
</body>
</html>
