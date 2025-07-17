<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mejora con Juegos</title>
    <!-- Tailwind CSS CDN para un estilo moderno y responsivo -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <!-- Tone.js CDN para la generación de audio (necesario para ambos minijuegos) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
    <style>
        /* Regla global para ocultar elementos */
        .hidden {
            display: none !important; /* Asegura que el elemento no ocupe espacio */
        }

        /* Estilos generales para el cuerpo y la fuente pixel art */
        body {
            font-family: 'Press Start 2P', cursive;
            background-color: #1a202c; /* Fondo oscuro */
            color: #e2e8f0; /* Texto claro */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 10px; /* Padding reducido para móviles */
            box-sizing: border-box;
            overflow-x: hidden; /* Evitar desbordamiento horizontal */
            background-image: radial-gradient(circle at center, #2d3748, #1a202c); /* Fondo con gradiente radial */
        }

        /* Contenedor principal de la aplicación */
        .main-app-container {
            /* Mejoras estéticas: más sombra, bordes más definidos, padding generoso */
            @apply flex flex-col items-center justify-center p-6 sm:p-8 rounded-xl shadow-2xl bg-gray-800;
            max-width: 800px;
            width: 100%;
            border: 4px solid #4a5568;
            box-shadow: 0 12px 25px rgba(0, 0, 0, 0.8); /* Sombra más profunda y extendida */
            position: relative; /* Para posibles efectos de brillo */
            overflow: hidden; /* Asegura que los efectos internos no se desborden */
        }
        .main-app-container::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle at center, rgba(66, 153, 225, 0.1), transparent 70%);
            animation: rotateBackground 20s linear infinite;
            z-index: 0;
            opacity: 0.3;
        }

        @keyframes rotateBackground {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }


        /* Título principal de la aplicación */
        #mainTitle {
            @apply text-3xl sm:text-4xl font-bold mb-6 sm:mb-8 text-white text-center;
            text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.6); /* Sombra de texto para un efecto más retro */
            z-index: 1; /* Asegura que el título esté por encima del fondo animado */
            position: relative;
        }

        /* Descripción de los minijuegos en la página principal */
        .game-description {
            @apply text-base sm:text-lg text-gray-300 mb-6 text-center px-2 sm:px-4;
            z-index: 1;
            position: relative;
        }

        /* Estilos para los botones de la aplicación principal */
        .app-button {
            /* Mejoras estéticas: gradiente, más sombra, efecto de burbuja al presionar */
            @apply px-6 py-3 sm:px-8 sm:py-4 rounded-xl font-bold text-lg sm:text-xl transition-all duration-200;
            background-image: linear-gradient(to bottom right, #4299e1, #2b6cb0); /* Gradiente azul */
            color: #fff;
            border: 2px solid #1a365d; /* Borde más oscuro */
            box-shadow: 6px 6px 0px #1a365d; /* Sombra más pronunciada */
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
            letter-spacing: 0.05em; /* Espaciado entre letras */
            z-index: 1;
            position: relative;
        }

        .app-button:hover {
            background-image: linear-gradient(to bottom right, #3182ce, #2c5282);
            box-shadow: 3px 3px 0px #1a365d;
            transform: translate(3px, 3px) scale(1.02); /* Escala sutil en hover */
        }

        .app-button:active {
            background-image: linear-gradient(to bottom right, #2c5282, #1a365d);
            box-shadow: none;
            transform: translate(6px, 6px) scale(0.98); /* Escala sutil en active */
        }

        /* Contenedor para el área de juego activa */
        #gameArea {
            @apply mt-6 sm:mt-8 w-full flex flex-col items-center; /* Cambiado a flex-col para centrar el botón de regreso */
            display: none; /* Inicialmente oculto */
        }

        /* --- Estilos Comunes para Ambos Minijuegos (dentro de gameArea) --- */
        #gameArea .game-container {
            /* Mejoras estéticas: más sombra, bordes más definidos */
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #2d3748;
            border-radius: 12px; /* Bordes más redondeados */
            box-shadow: 0 0 25px rgba(0, 0, 0, 0.6); /* Sombra más pronunciada */
            padding: 20px; /* Padding aumentado */
            width: 100%;
            box-sizing: border-box;
            position: relative;
            border: 4px solid #4a5568;
            background: linear-gradient(to bottom right, #2d3748, #1a202c); /* Gradiente para el contenedor del juego */
        }

        #gameArea .info-panel {
            @apply flex flex-col sm:flex-row justify-between items-center w-full mt-2 p-3 rounded-lg bg-gray-700 text-gray-200 text-sm sm:text-base; /* Padding y tamaño de fuente ajustados */
            border: 2px solid #4a5568;
            box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.3); /* Sombra interna para efecto de profundidad */
            background-color: rgba(45, 55, 72, 0.8); /* Fondo semi-transparente */
            backdrop-filter: blur(3px); /* Efecto de desenfoque */
        }
        #gameArea .info-panel span {
            @apply mb-1 sm:mb-0;
        }

        #gameArea .game-button {
            /* Mejoras estéticas: similar a app-button pero más pequeño */
            @apply px-4 py-2 text-base rounded-lg font-bold transition-all duration-200;
            background-image: linear-gradient(to bottom right, #4299e1, #2b6cb0);
            color: #fff;
            border: 2px solid #1a365d;
            box-shadow: 4px 4px 0px #1a365d;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
            letter-spacing: 0.03em;
        }
        #gameArea .game-button:hover {
            background-image: linear-gradient(to bottom right, #3182ce, #2c5282);
            box-shadow: 2px 2px 0px #1a365d;
            transform: translate(2px, 2px) scale(1.02);
        }

        #gameArea .game-button:active {
            background-image: linear-gradient(to bottom right, #2c5282, #1a365d);
            box-shadow: none;
            transform: translate(4px, 4px) scale(0.98);
        }
        #gameArea .game-button:disabled {
            background-color: #a0aec0;
            border-color: #718096;
            box-shadow: 3px 3px 0px #718096;
            cursor: not-allowed;
        }

        /* Este 'hidden' es específico para las pantallas de inicio dentro de los juegos */
        #gameArea #startScreen.hidden {
            opacity: 0;
            pointer-events: none;
        }
        #gameArea #startScreenTitle {
            @apply text-3xl sm:text-4xl font-bold mb-4 sm:mb-6 text-white text-center;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
        #gameArea #startScreenMessage {
            @apply text-base sm:text-lg text-gray-300 mb-6 sm:mb-8 text-center px-2 sm:px-4;
        }
        #gameArea #startScreenButton {
            @apply text-lg sm:text-xl px-4 py-2 sm:px-6 sm:py-3;
        }
        #gameArea #startScreen p.text-sm {
            @apply text-xs sm:text-sm mt-2;
        }

        /* --- Estilos Específicos para Esquiva los Asteroides --- */
        #asteroidGame .game-container {
            max-width: 640px;
        }
        #asteroidGame canvas {
            background-color: #000000;
            border: 4px solid #a0aec0;
            border-radius: 8px;
            display: block;
            width: 100%;
            max-width: 640px;
            height: auto;
            image-rendering: optimizeSpeed;
            image-rendering: -webkit-crisp-edges;
            image-rendering: -moz-crisp-edges;
            image-rendering: crisp-edges;
        }
        #asteroidGame .touch-controls {
            @apply flex justify-center gap-4 sm:gap-8 w-full mt-4;
        }
        #asteroidGame .touch-button {
            /* Mejoras estéticas: más grande, sombra más suave */
            @apply p-4 sm:p-5 rounded-full bg-gray-700 text-white text-3xl sm:text-4xl font-bold transition-all duration-200;
            box-shadow: 4px 4px 0px #4a5568; /* Sombra más grande */
            user-select: none;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            touch-action: manipulation;
        }
        #asteroidGame .touch-button:active {
            background-color: #4a5568;
            box-shadow: 2px 2px 0px #4a5568;
            transform: translate(2px, 2px);
        }

        /* --- Estilos Específicos para Memorama --- */
        #memoramaGame .game-container {
            max-width: 500px; /* Ancho máximo ajustado para el tablero de memorama */
        }
        #memoramaGame .game-board {
            @apply grid p-5 sm:p-6 bg-gray-900 rounded-xl border-4 border-gray-700; /* Padding ajustado */
            grid-gap: 16px; /* Aún mayor espacio entre las celdas */
            width: 100%;
            max-width: 460px; /* Ancho máximo del tablero ajustado */
            aspect-ratio: 1 / 1; /* Mantener proporción cuadrada */
            display: grid;
            grid-template-columns: repeat(6, 1fr); /* 6 columnas */
            grid-template-rows: repeat(4, 1fr); /* 4 filas */
            box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.4); /* Sombra interna para el tablero */
        }
        /* New: Pulsating background for low time warning */
        #memoramaGame .game-container.low-time-warning {
            animation: pulseBackground 1s infinite alternate;
        }

        @keyframes pulseBackground {
            from { background: linear-gradient(to bottom right, #2d3748, #1a202c); }
            to { background: linear-gradient(to bottom right, #4a5568, #333); } /* Slightly lighter/more intense */
        }

        #memoramaGame .card {
            /* Mejoras estéticas: sombra más pronunciada, bordes más suaves */
            @apply relative w-full h-full rounded-xl cursor-pointer transition-transform duration-500 transform-gpu; /* Bordes más redondeados */
            transform-style: preserve-3d;
            box-shadow: 5px 5px 0px #2d3748; /* Sombra más grande y color de sombra más oscuro */
            border: 2px solid #4a5568; /* Borde más definido */
            background-color: #a0aec0; /* Fondo inicial gris */
            display: flex; /* Centrar contenido */
            align-items: center; /* Centrar verticalmente */
            justify-content: center; /* Centrar horizontalmente */
            font-size: 3.5rem; /* Tamaño del signo de interrogación inicial */
            color: #1a202c; /* Color del signo de interrogación */
            font-weight: bold;
            text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.2);
        }
        #memoramaGame .card.flipped {
            transform: rotateY(180deg);
            box-shadow: 2px 2px 0px #2d3748; /* Sombra reducida al voltear */
            background-color: #4299e1; /* Fondo azul al voltear */
            font-size: 2rem; /* Tamaño del emoji volteado */
        }
        #memoramaGame .card.matched {
            pointer-events: none;
            box-shadow: none;
            opacity: 0.6; /* Ligeramente más opaco para "matched" */
            transform: scale(0.95); /* Menos reducción de escala */
            background-color: #4299e1; /* Mantener fondo azul */
        }
        /* Ajustar tamaño de fuente de emoji para pantallas pequeñas */
        @media (max-width: 640px) {
            #memoramaGame .card {
                font-size: 2.8rem; /* Ajuste para móviles para el signo de interrogación */
            }
            #memoramaGame .card.flipped {
                font-size: 1.5rem; /* Ajuste para móviles para el emoji */
            }
            #memoramaGame .game-board {
                max-width: 300px; /* Ancho máximo del tablero para móviles */
                grid-gap: 10px; /* Ajuste de gap para móviles */
            }
        }

        /* Nueva clase para el símbolo de verificación */
        #memoramaGame .card-matched-symbol {
            color: #48bb78; /* Verde */
            font-size: 2.8rem; /* Tamaño grande para el checkmark */
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
        @media (max-width: 640px) {
            #memoramaGame .card-matched-symbol {
                font-size: 2.2rem; /* Ajuste para móviles */
            }
        }


        /* Estilos para el modal */
        .modal {
            @apply fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50;
            opacity: 0; /* Inicialmente transparente */
            transition: opacity 0.3s ease-out; /* Transición para la opacidad */
        }
        .modal.hidden {
            opacity: 0;
            pointer-events: none; /* Deshabilita interacciones cuando está oculto */
        }
        .modal:not(.hidden) {
            opacity: 1; /* Muestra el modal */
        }


        .modal-content {
            @apply bg-gray-800 p-8 rounded-lg shadow-xl text-center; /* Increased padding */
            border: 4px solid #4299e1;
            animation: fadeIn 0.3s ease-out;
            max-width: 90%; /* Responsive width */
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.8); /* Sombra más dramática */
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }

        .modal-title {
            @apply text-3xl sm:text-4xl font-bold mb-4 text-white; /* Larger font size */
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .modal-message {
            @apply text-lg sm:text-xl mb-6 text-gray-300; /* Larger font size */
        }
        .modal-content .game-button { /* Style modal button */
            @apply px-6 py-3 text-lg;
        }

    </style>
</head>
<body>
    <div class="main-app-container">
        <h1 id="mainTitle" class="text-4xl font-bold mb-8 text-white text-center">🎮 Mejora con Juegos 🚀</h1>

        <!-- Botón de silencio global -->
        <button id="globalMuteButton" class="app-button mb-8">Silenciar Todo</button>

        <!-- Sección de Esquiva los Asteroides -->
        <p id="asteroidGameDescription" class="game-description">
            ¡Esquiva los Asteroides! 🚀 Sobrevive a la lluvia de rocas espaciales. Este minijuego mejorará tus reflejos y tiempo de reacción. ¡A jugar! ✨
        </p>
        <button id="startGameAsteroidsButton" class="app-button mb-8">Jugar Esquiva los Asteroides</button>

        <!-- Divisor entre minijuegos -->
        <div class="w-full h-2 my-8 rounded-full shadow-lg" style="background-image: linear-gradient(to right, transparent, #00BCD4, #8A2BE2, transparent);"></div>

        <!-- Sección de Memorama -->
        <p id="memoramaGameDescription" class="game-description">
            ¡Memorama! 🧠 Pon a prueba tu memoria y encuentra todas las parejas. ¡Un clásico para agudizar tu mente! 💡
        </p>
        <button id="startGameMemoramaButton" class="app-button mb-8">Jugar Memorama</button>


        <!-- Área donde se cargará el minijuego activo -->
        <div id="gameArea">
            <!-- Contenido del juego de Asteroides (inicialmente oculto) -->
            <div id="asteroidGame" class="game-container hidden">
                <h1 class="text-3xl font-bold mb-4 text-white">Esquiva los Asteroides</h1>
                <!-- El botón de silencio específico del juego de asteroides se ha eliminado ya que ahora hay uno global -->
                <div id="gameUIAsteroids" class="flex flex-col items-center w-full hidden">
                    <div class="info-panel">
                        <span>Tiempo Restante: <span id="timeDisplayAsteroids">60s</span></span>
                        <span>Vidas: <span id="livesDisplayAsteroids">2</span></span>
                    </div>
                    <canvas id="gameCanvasAsteroids" width="640" height="320"></canvas>
                    <div class="touch-controls flex md:hidden">
                        <button id="leftButtonAsteroids" class="touch-button">◀</button>
                        <button id="rightButtonAsteroids" class="touch-button">▶</button>
                    </div>
                    <button id="backToMenuButtonAsteroids" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <div id="startScreenAsteroids">
                    <h2 id="startScreenTitleAsteroids" class="text-4xl font-bold mb-6 text-white text-center">Esquiva los Asteroides</h2>
                    <p id="startScreenMessageAsteroids" class="text-lg text-gray-300 mb-8 text-center px-4">Sobrevive 60 segundos a una lluvia de asteroides. Este minijuego te ayudará a mejorar tus reflejos y tu tiempo de reacción. 🚀✨</p>
                    <button id="startScreenButtonAsteroids" class="game-button text-xl px-6 py-3">¡Comenzar!</button>
                    <p class="text-sm text-gray-400 mt-4">Controles: Flechas izquierda/derecha o A/D</p>
                    <p class="text-sm text-gray-400">Controles táctiles: Botones ◀ ▶</p>
                </div>
            </div>

            <!-- Contenido del juego de Memorama (inicialmente oculto) -->
            <div id="memoramaGame" class="game-container hidden">
                <h1 class="text-3xl font-bold mb-4 text-white">Memorama</h1>
                <!-- El botón de silencio específico del juego de memorama se ha eliminado ya que ahora hay uno global -->
                <div id="gameUIMemorama" class="flex flex-col items-center w-full hidden">
                    <div class="info-panel">
                        <span>Tiempo: <span id="timeDisplayMemorama">0s</span></span>
                        <span>Pares Encontrados: <span id="matchedPairsDisplayMemorama">0</span>/<span id="totalPairsDisplayMemorama">0</span></span>
                    </div>
                    <div id="gameBoardMemorama" class="game-board">
                        <!-- Las cartas se generarán aquí con JavaScript -->
                    </div>
                    <button id="backToMenuButtonMemorama" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <div id="startScreenMemorama">
                    <h2 id="startScreenTitleMemorama" class="text-4xl font-bold mb-6 text-white text-center">Memorama</h2>
                    <p id="startScreenMessageMemorama" class="text-lg text-gray-300 mb-8 text-center px-4">Encuentra todas las parejas de emojis antes de que se acabe el tiempo. ¡Entrena tu memoria!</p>
                    <button id="startScreenButtonMemorama" class="game-button text-xl px-6 py-3">¡Comenzar!</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal para resultados del juego -->
    <div id="gameModal" class="modal hidden">
        <div class="modal-content">
            <h2 id="modalTitle" class="modal-title"></h2>
            <p id="modalMessage" class="modal-message"></p>
            <button id="modalButton" class="game-button">Cerrar</button>
        </div>
    </div>

    <script>
        // --- Variables de control de audio globales ---
        let isMasterMuted = false;
        let mainMenuSynth;
        let mainMenuSequence;

        // --- Referencias y Lógica de la Aplicación Principal ---
        const mainTitle = document.getElementById('mainTitle');
        const asteroidGameDescription = document.getElementById('asteroidGameDescription');
        const startGameAsteroidsButton = document.getElementById('startGameAsteroidsButton');
        const memoramaGameDescription = document.getElementById('memoramaGameDescription');
        const startGameMemoramaButton = document.getElementById('startGameMemoramaButton');
        const gameArea = document.getElementById('gameArea');
        const asteroidGameContainer = document.getElementById('asteroidGame');
        const memoramaGameContainer = document.getElementById('memoramaGame');
        const globalMuteButton = document.getElementById('globalMuteButton'); // Nuevo botón de silencio global

        // Modal elements
        const gameModal = document.getElementById('gameModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalButton = document.getElementById('modalButton');

        function showModal(title, message, buttonText = "Cerrar", callback = null) {
            modalTitle.textContent = title;
            modalMessage.textContent = message;
            modalButton.textContent = buttonText;
            modalButton.onclick = () => {
                gameModal.classList.add('hidden');
                if (callback) callback();
            };
            gameModal.classList.remove('hidden');
        }

        function hideMainMenu() {
            mainTitle.style.display = 'none';
            asteroidGameDescription.style.display = 'none';
            startGameAsteroidsButton.style.display = 'none';
            memoramaGameDescription.style.display = 'none';
            startGameMemoramaButton.style.display = 'none';
            globalMuteButton.style.display = 'none'; // Ocultar el botón de silencio global en el juego
            gameArea.style.display = 'flex'; // Show the overall game area
            stopMainMenuMusic(); // Detener la música del menú principal
        }

        function showMainMenu() {
            mainTitle.style.display = 'block';
            asteroidGameDescription.style.display = 'block';
            startGameAsteroidsButton.style.display = 'block';
            memoramaGameDescription.style.display = 'block';
            startGameMemoramaButton.style.display = 'block';
            globalMuteButton.style.display = 'block'; // Mostrar el botón de silencio global
            gameArea.style.display = 'none'; // Hide the overall game area
            
            // Also stop and hide any running games
            stopGameAsteroidsInternal(); // Call internal stop function
            asteroidGameContainer.classList.add('hidden'); // Ensure it's hidden
            stopGameMemoramaInternal(); // Call internal stop function for Memorama
            memoramaGameContainer.classList.add('hidden'); // Ensure it's hidden
            startMainMenuMusic(); // Iniciar la música del menú principal
        }

        // Internal stop function for Asteroids
        function stopGameAsteroidsInternal() {
            gameRunningAsteroids = false;
            cancelAnimationFrame(animationFrameIdAsteroids);
            stopBackgroundMusicAsteroids(); // Usar la función de parada de música específica del juego
            gameUIAsteroids.style.display = 'none'; // Ensure game UI is hidden
            startScreenAsteroids.classList.remove('hidden'); // Ensure start screen is visible
        }

        // Internal stop function for Memorama (New)
        function stopGameMemoramaInternal() {
            gameRunningMemorama = false;
            clearInterval(timerIntervalMemorama);
            stopBackgroundMusicMemorama(); // Usar la función de parada de música específica del juego
            stopLowTimeMusicMemorama(); // Stop low time music
            memoramaGameContainer.classList.remove('low-time-warning'); // Remove visual warning
            gameUIMemorama.style.display = 'none'; // Ensure game UI is hidden
            startScreenMemorama.classList.remove('hidden'); // Ensure start screen is visible
        }

        // Function to hide all game containers
        function hideAllGames() {
            asteroidGameContainer.classList.add('hidden');
            memoramaGameContainer.classList.add('hidden'); // Hide Memorama container
            // Stop any running game processes when hiding all
            stopGameAsteroidsInternal();
            stopGameMemoramaInternal();
        }

        // --- Lógica de Audio Global y del Menú Principal ---
        function setupMainMenuAudio() {
            if (!mainMenuSynth) {
                mainMenuSynth = new Tone.Synth({
                    oscillator: { type: "triangle" },
                    envelope: { attack: 0.02, decay: 0.5, sustain: 0.1, release: 0.8 }
                }).toDestination();
                mainMenuSynth.volume.value = -25; // Volumen más bajo para el menú

                const mainMenuNotes = [
                    ["C3", "4n"], ["E3", "4n"], ["G3", "4n"], ["C4", "4n"],
                    ["A2", "4n"], ["C3", "4n"], ["E3", "4n"], ["A3", "4n"]
                ];

                mainMenuSequence = new Tone.Sequence((time, note) => {
                    try {
                        mainMenuSynth.triggerAttackRelease(note, "8n", time);
                    } catch (e) {
                        console.error("Error processing note in main menu sequence:", note, e);
                    }
                }, mainMenuNotes.flat(), "2n");

                Tone.Transport.bpm.value = 60;
                Tone.Transport.loop = true;
            }
        }

        function startMainMenuMusic() {
            if (mainMenuSequence && typeof mainMenuSequence.start === 'function') {
                mainMenuSequence.start(Tone.Transport.now()); // Changed to start at current transport time
                mainMenuSynth.volume.value = isMasterMuted ? -100 : -25;
                Tone.Transport.bpm.value = 60;
            }
        }

        function stopMainMenuMusic() {
            if (mainMenuSequence && typeof mainMenuSequence.stop === 'function') {
                mainMenuSequence.stop();
            }
        }

        function toggleGlobalMute() {
            isMasterMuted = !isMasterMuted;
            globalMuteButton.textContent = isMasterMuted ? "Activar Sonido" : "Silenciar Todo";

            // Actualizar volumen de todos los synths
            if (mainMenuSynth) mainMenuSynth.volume.value = isMasterMuted ? -100 : -25;
            if (backgroundSynthAsteroids) backgroundSynthAsteroids.volume.value = isMasterMuted ? -100 : -15;
            if (memoramaSynth) memoramaSynth.volume.value = isMasterMuted ? -100 : -15; // Ajustado a -15 para que sea audible
            if (lowTimeSynthMemorama) lowTimeSynthMemorama.volume.value = isMasterMuted ? -100 : -10; // New: low time synth
        }

        globalMuteButton.addEventListener('click', toggleGlobalMute);


        // --- Lógica del Minijuego Esquiva los Asteroides ---
        const canvasAsteroids = document.getElementById('gameCanvasAsteroids');
        const ctxAsteroids = canvasAsteroids.getContext('2d');

        const timeDisplayAsteroids = document.getElementById('timeDisplayAsteroids');
        const livesDisplayAsteroids = document.getElementById('livesDisplayAsteroids');

        const startScreenAsteroids = document.getElementById('startScreenAsteroids');
        const startScreenButtonAsteroids = document.getElementById('startScreenButtonAsteroids');
        const startScreenTitleAsteroids = document.getElementById('startScreenTitleAsteroids');
        const startScreenMessageAsteroids = document.getElementById('startScreenMessageAsteroids');
        const gameUIAsteroids = document.getElementById('gameUIAsteroids');
        const backToMenuButtonAsteroids = document.getElementById('backToMenuButtonAsteroids');


        let gameRunningAsteroids = false;
        let gameTimeAsteroids = 0;
        let lastFrameTimeAsteroids = 0;
        let animationFrameIdAsteroids;
        let asteroids = [];
        let holograms = [];
        let stars = [];
        let keysAsteroids = {};
        let confetti = [];
        let awaitingMenuTapAsteroids = false;

        const PLAYER_SIZE_ASTEROIDS = 24;
        let playerAsteroids;
        let playerLivesAsteroids = 2;
        let isInvincibleAsteroids = false;
        let invincibilityEndTimeAsteroids = 0;

        const BASE_ASTEROID_SPAWN_RATE = 1500;
        const BASE_MIN_ASTEROID_SPEED = 0.8;
        const BASE_MAX_ASTEROID_SPEED = 2.5;
        const BASE_MIN_ASTEROID_SIZE = 10;
        const BASE_MAX_ASTEROID_SIZE = 30;

        let currentAsteroidSpawnRate;
        let currentMinAsteroidSpeed;
        let currentMaxAsteroidSpeed;
        let currentMinAsteroidSize;
        let currentMaxAsteroidSize;

        const HOLOGRAM_SPAWN_RATE = 1500;
        let lastHologramSpawnTime = 0;

        let currentWarningAsteroids = { message: "", displayUntil: 0 };

        let backgroundSynthAsteroids;
        let backgroundSequenceAsteroids;

        function setupAudioAsteroids() {
            // Only initialize if not already initialized
            if (!backgroundSynthAsteroids) {
                backgroundSynthAsteroids = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sawtooth" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.3, release: 0.5 },
                    filter: { type: "lowpass", frequency: 1000, rolloff: -12 }
                }).toDestination();
                backgroundSynthAsteroids.volume.value = -15;

                const freneticNotes = [
                    ["C4", "8n"], ["G3", "8n"], ["A3", "8n"], ["E3", "8n"],
                    ["C4", "8n"], ["G3", "8n"], ["A3", "8n"], ["E3", "8n"],
                    ["D4", "8n"], ["A3", "8n"], ["B3", "8n"], ["F#3", "8n"],
                    ["D4", "8n"], ["A3", "8n"], ["B3", "8n"], ["F#3", "8n"]
                ];

                backgroundSequenceAsteroids = new Tone.Sequence((time, note) => {
                    try {
                        backgroundSynthAsteroids.triggerAttackRelease(note, "16n", time);
                    } catch (e) {
                        console.error("Error processing note in background sequence:", note, e);
                    }
                }, freneticNotes.flat(), "8n");

                Tone.Transport.bpm.value = 140;
                Tone.Transport.loop = true;
            }
        }

        function startBackgroundMusicAsteroids() {
            if (backgroundSequenceAsteroids && typeof backgroundSequenceAsteroids.start === 'function') {
                backgroundSequenceAsteroids.start(Tone.Transport.now()); // Changed to start at current transport time
                backgroundSynthAsteroids.volume.value = isMasterMuted ? -100 : -15;
                Tone.Transport.bpm.value = 140;
            }
        }

        function stopBackgroundMusicAsteroids() {
            if (backgroundSequenceAsteroids && typeof backgroundSequenceAsteroids.stop === 'function') {
                backgroundSequenceAsteroids.stop();
            }
        }

        function checkCollisionAsteroids(rect1, rect2) {
            return rect1.x < rect2.x + rect2.size &&
                   rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.size &&
                   rect1.y + rect1.height > rect2.y;
        }

        function getDifficultyPhaseAsteroids(time) {
            if (time >= 30) return 3;
            if (time >= 10) return 2;
            return 1;
        }

        function showWarningAsteroids(message) {
            currentWarningAsteroids.message = message;
            currentWarningAsteroids.displayUntil = performance.now() + 2000;
        }

        function updateUIAsteroids() {
            timeDisplayAsteroids.textContent = `${Math.max(0, 60 - Math.floor(gameTimeAsteroids))}s`;
            livesDisplayAsteroids.textContent = playerLivesAsteroids;
        }

        function endGameAsteroids(won, cheatWin = false) {
            gameRunningAsteroids = false;
            cancelAnimationFrame(animationFrameIdAsteroids);
            stopBackgroundMusicAsteroids();

            gameUIAsteroids.style.display = 'none';

            let title, message;
            if (won) {
                title = "¡VICTORIA!";
                message = "¡Has sobrevivido a la lluvia de asteroides!";
                for (let i = 0; i < 100; i++) {
                    confetti.push(new ConfettiParticle(Math.random() * canvasAsteroids.width, Math.random() * canvasAsteroids.height / 2));
                }
            } else {
                title = "¡PERDISTE!";
                message = "Los asteroides te han alcanzado.";
            }
            showModal(title, message, "Volver al Menú Principal", showMainMenu);
            awaitingMenuTapAsteroids = true; // Keep this true to prevent gameLoop from restarting
        }

        // Improved Player Ship Drawing
        class PlayerAsteroids {
            constructor(x, y, width, height, speed) {
                this.x = x;
                this.y = y;
                this.width = width;
                this.height = height;
                this.speed = speed;
            }

            draw() {
                if (isInvincibleAsteroids) {
                    ctxAsteroids.globalAlpha = (Math.floor(performance.now() / 100) % 2 === 0) ? 0.5 : 1;
                }
                
                // Main body of the ship
                ctxAsteroids.fillStyle = '#C0C0C0'; // Grey
                ctxAsteroids.beginPath();
                ctxAsteroids.moveTo(this.x + this.width / 2, this.y); // Top point
                ctxAsteroids.lineTo(this.x, this.y + this.height); // Bottom-left
                ctxAsteroids.lineTo(this.x + this.width / 2, this.y + this.height * 0.75); // Mid-bottom
                ctxAsteroids.lineTo(this.x + this.width, this.y + this.height); // Bottom-right
                ctxAsteroids.closePath();
                ctxAsteroids.fill();

                // Cockpit
                ctxAsteroids.fillStyle = '#ADD8E6'; // Light Blue
                ctxAsteroids.beginPath();
                ctxAsteroids.arc(this.x + this.width / 2, this.y + this.height * 0.4, this.width * 0.2, 0, Math.PI * 2);
                ctxAsteroids.fill();

                // Thrusters
                ctxAsteroids.fillStyle = '#FF4500'; // Orange-Red
                const thrusterWidth = this.width * 0.15;
                const thrusterHeight = this.height * 0.2;
                ctxAsteroids.fillRect(this.x + this.width * 0.1, this.y + this.height - thrusterHeight, thrusterWidth, thrusterHeight);
                ctxAsteroids.fillRect(this.x + this.width * 0.75, this.y + this.height - thrusterHeight, thrusterWidth, thrusterHeight);

                // Flames (dynamic)
                ctxAsteroids.fillStyle = `rgba(255, ${Math.floor(Math.random() * 150) + 100}, 0, ${Math.random() * 0.5 + 0.5})`; // Orange-yellow
                const flameLength = Math.random() * 10 + 5;
                ctxAsteroids.beginPath();
                ctxAsteroids.moveTo(this.x + this.width * 0.1 + thrusterWidth / 2, this.y + this.height);
                ctxAsteroids.lineTo(this.x + this.width * 0.1 + thrusterWidth / 2 - flameLength / 2, this.y + this.height + flameLength);
                ctxAsteroids.lineTo(this.x + this.width * 0.1 + thrusterWidth / 2 + flameLength / 2, this.y + this.height + flameLength);
                ctxAsteroids.closePath();
                ctxAsteroids.fill();

                ctxAsteroids.beginPath();
                ctxAsteroids.moveTo(this.x + this.width * 0.75 + thrusterWidth / 2, this.y + this.height);
                ctxAsteroids.lineTo(this.x + this.width * 0.75 + thrusterWidth / 2 - flameLength / 2, this.y + this.height + flameLength);
                ctxAsteroids.lineTo(this.x + this.width * 0.75 + thrusterWidth / 2 + flameLength / 2, this.y + this.height + flameLength);
                ctxAsteroids.closePath();
                ctxAsteroids.fill();

                ctxAsteroids.globalAlpha = 1;
            }

            update() {
                if (keysAsteroids['ArrowLeft'] || keysAsteroids['a']) {
                    this.x -= this.speed;
                }
                if (keysAsteroids['ArrowRight'] || keysAsteroids['d']) {
                    this.x += this.speed;
                }
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvasAsteroids.width) this.x = canvasAsteroids.width - this.width;
            }
        }

        // Improved Asteroid Drawing
        class FallingObjectAsteroids {
            constructor(type, x, y, size, speed, color, rotationSpeed) {
                this.type = type;
                this.x = x;
                this.y = y;
                this.size = size;
                this.speed = speed;
                this.color = color;
                this.rotation = Math.random() * Math.PI * 2;
                this.rotationSpeed = rotationSpeed;
                this.markedForRemoval = false;
            }

            draw() {
                ctxAsteroids.save();
                ctxAsteroids.translate(this.x + this.size / 2, this.y + this.size / 2);
                ctxAsteroids.rotate(this.rotation);

                // Draw irregular asteroid shape
                ctxAsteroids.beginPath();
                const numPoints = Math.floor(Math.random() * 4) + 5; // 5 to 8 points
                for (let i = 0; i < numPoints; i++) {
                    const angle = (i / numPoints) * Math.PI * 2;
                    const radius = this.size / 2 * (0.8 + Math.random() * 0.4); // Vary radius for irregularity
                    ctxAsteroids.lineTo(Math.cos(angle) * radius, Math.sin(angle) * radius);
                }
                ctxAsteroids.closePath();
                ctxAsteroids.fillStyle = this.color;
                ctxAsteroids.fill();
                ctxAsteroids.strokeStyle = '#333';
                ctxAsteroids.lineWidth = 1;
                ctxAsteroids.stroke();

                // Add some craters/details
                ctxAsteroids.fillStyle = 'rgba(0,0,0,0.3)';
                ctxAsteroids.beginPath();
                ctxAsteroids.arc(this.size * 0.1, this.size * 0.1, this.size * 0.15, 0, Math.PI * 2);
                ctxAsteroids.fill();
                ctxAsteroids.beginPath();
                ctxAsteroids.arc(-this.size * 0.2, -this.size * 0.1, this.size * 0.1, 0, Math.PI * 2);
                ctxAsteroids.fill();

                ctxAsteroids.restore();
            }

            update() {
                if (this.type === 'guided' && playerAsteroids) {
                    const targetX = playerAsteroids.x + playerAsteroids.width / 2;
                    const currentX = this.x + this.size / 2;
                    const GUIDED_RANGE = 200;

                    const horizontalDistance = Math.abs(targetX - currentX);
                    if (horizontalDistance < GUIDED_RANGE) {
                        const guidanceStrength = 0.1;
                        if (targetX < currentX) {
                            this.x -= this.speed * guidanceStrength;
                        } else if (targetX > currentX) {
                            this.x += this.speed * guidanceStrength;
                        }
                    }
                }
                this.y += this.speed;
                this.rotation += this.rotationSpeed;
            }
        }

        class ConfettiParticle {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.size = Math.random() * 5 + 3;
                this.color = `hsl(${Math.random() * 360}, 100%, 70%)`;
                this.speedY = Math.random() * 3 + 1;
                this.speedX = (Math.random() - 0.5) * 4;
                this.rotation = Math.random() * Math.PI * 2;
                this.rotationSpeed = (Math.random() - 0.5) * 0.2;
                this.alpha = 1;
                this.gravity = 0.1;
            }

            draw() {
                ctxAsteroids.save();
                ctxAsteroids.globalAlpha = this.alpha;
                ctxAsteroids.fillStyle = this.color;
                ctxAsteroids.translate(this.x + this.size / 2, this.y + this.size / 2);
                ctxAsteroids.rotate(this.rotation);
                ctxAsteroids.fillRect(-this.size / 2, -this.size / 2, this.size, this.size);
                ctxAsteroids.restore();
            }

            update() {
                this.x += this.speedX;
                this.y += this.speedY;
                this.speedY += this.gravity;
                this.rotation += this.rotationSpeed;
                this.alpha -= 0.01;
            }
        }

        function startGameAsteroidsLogic() {
            if (gameRunningAsteroids) return;
            startScreenAsteroids.classList.add('hidden');
            gameUIAsteroids.classList.remove('hidden');
            gameUIAsteroids.style.display = 'flex';
            resetGameAsteroids();
            gameRunningAsteroids = true;
            lastFrameTimeAsteroids = performance.now();
            
            // Stop other game music before starting this one
            stopMainMenuMusic();
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama();

            startBackgroundMusicAsteroids();
            gameLoopAsteroids(lastFrameTimeAsteroids);
        }

        function resetGameAsteroids() {
            cancelAnimationFrame(animationFrameIdAsteroids);
            gameTimeAsteroids = 0;
            asteroids = [];
            holograms = [];
            stars = [];
            confetti = [];
            awaitingMenuTapAsteroids = false;

            startScreenTitleAsteroids.textContent = "Esquiva los Asteroides";
            startScreenMessageAsteroids.textContent = "Sobrevive 60 segundos a una lluvia de asteroides. Este minijuego te ayudará a mejorar tus reflejos y tu tiempo de reacción. 🚀✨";
            startScreenButtonAsteroids.textContent = "¡Comenzar!";

            playerAsteroids = new PlayerAsteroids(canvasAsteroids.width / 2 - PLAYER_SIZE_ASTEROIDS / 2, canvasAsteroids.height - PLAYER_SIZE_ASTEROIDS * 2, PLAYER_SIZE_ASTEROIDS, PLAYER_SIZE_ASTEROIDS, 5);
            playerLivesAsteroids = 2;
            isInvincibleAsteroids = false;
            invincibilityEndTimeAsteroids = 0;

            currentAsteroidSpawnRate = BASE_ASTEROID_SPAWN_RATE;
            currentMinAsteroidSpeed = BASE_MIN_ASTEROID_SPEED;
            currentMaxAsteroidSpeed = BASE_MAX_ASTEROID_SPEED;
            currentMinAsteroidSize = BASE_MIN_ASTEROID_SIZE;
            currentMaxAsteroidSize = BASE_MAX_ASTEROID_SIZE;

            lastAsteroidSpawnTime = performance.now();
            lastHologramSpawnTime = performance.now();

            // Clear existing stars and re-populate with layers
            stars = [];
            for (let i = 0; i < 50; i++) { // Distant, slow stars
                stars.push({
                    x: Math.random() * canvasAsteroids.width,
                    y: Math.random() * canvasAsteroids.height,
                    size: Math.random() * 1 + 0.5,
                    speed: Math.random() * 0.1 + 0.05
                });
            }
            for (let i = 0; i < 50; i++) { // Closer, faster stars
                stars.push({
                    x: Math.random() * canvasAsteroids.width,
                    y: Math.random() * canvasAsteroids.height,
                    size: Math.random() * 2 + 1,
                    speed: Math.random() * 0.3 + 0.1
                });
            }

            currentWarningAsteroids = { message: "", displayUntil: 0 };

            updateUIAsteroids();
            drawAsteroids();
        }

        function gameLoopAsteroids(currentTime) {
            if (!gameRunningAsteroids && !awaitingMenuTapAsteroids) return; // Added awaitingMenuTapAsteroids check

            const deltaTime = currentTime - lastFrameTimeAsteroids;
            lastFrameTimeAsteroids = currentTime;

            if (gameRunningAsteroids) {
                gameTimeAsteroids += deltaTime / 1000;
            }
            
            updateAsteroids(deltaTime);
            drawAsteroids();

            if (gameRunningAsteroids && gameTimeAsteroids >= 60) {
                endGameAsteroids(true);
                return;
            }

            animationFrameIdAsteroids = requestAnimationFrame(gameLoopAsteroids);
        }

        function updateAsteroids(deltaTime) {
            if (gameRunningAsteroids) {
                playerAsteroids.update();

                if (isInvincibleAsteroids && performance.now() > invincibilityEndTimeAsteroids) {
                    isInvincibleAsteroids = false;
                }

                const oldDifficultyPhase = getDifficultyPhaseAsteroids(Math.floor(gameTimeAsteroids - (deltaTime / 1000)));
                const newDifficultyPhase = getDifficultyPhaseAsteroids(Math.floor(gameTimeAsteroids));

                if (gameTimeAsteroids >= 30) {
                    // Increased difficulty for the final phase
                    currentAsteroidSpawnRate = 100; // More frequent asteroids
                    currentMinAsteroidSpeed = 5;    // Faster minimum speed
                    currentMaxAsteroidSpeed = 10;   // Faster maximum speed
                    currentMinAsteroidSize = 25;    // Larger minimum size
                    currentMaxAsteroidSize = 55;    // Larger maximum size
                    if (oldDifficultyPhase < 3 && newDifficultyPhase === 3) {
                        showWarningAsteroids("¡INFIRERNO TOTAL!");
                        Tone.Transport.bpm.value = 180;
                    }
                } else if (gameTimeAsteroids >= 10) {
                    currentAsteroidSpawnRate = 400;
                    currentMinAsteroidSpeed = 2;
                    currentMaxAsteroidSpeed = 5;
                    currentMinAsteroidSize = 15;
                    currentMaxAsteroidSize = 35;
                    if (oldDifficultyPhase < 2 && newDifficultyPhase === 2) {
                        showWarningAsteroids("¡DIFICULTAD MEDIA!");
                        Tone.Transport.bpm.value = 140;
                    }
                } else {
                    currentAsteroidSpawnRate = BASE_ASTEROID_SPAWN_RATE;
                    currentMinAsteroidSpeed = BASE_MIN_ASTEROID_SPEED;
                    currentMaxAsteroidSpeed = BASE_MAX_ASTEROID_SPEED;
                    currentMinAsteroidSize = BASE_MIN_ASTEROID_SIZE;
                    currentMaxAsteroidSize = BASE_MAX_ASTEROID_SIZE;
                    Tone.Transport.bpm.value = 140;
                }

                if (performance.now() - lastAsteroidSpawnTime > currentAsteroidSpawnRate) {
                    const typeRoll = Math.random();
                    if (typeRoll < 0.08) {
                        asteroids.push(new FallingObjectAsteroids('guided', Math.random() * (canvasAsteroids.width - 30), -30, Math.random() * 15 + 20, Math.random() * 1.5 + 1.5, '#0000FF', (Math.random() - 0.5) * 0.05));
                    } else if (typeRoll < 0.18 && gameTimeAsteroids > 5) {
                        asteroids.push(new FallingObjectAsteroids('fast', Math.random() * (canvasAsteroids.width - 20), -20, Math.random() * 10 + 10, Math.random() * 5 + 5, '#00FF00', (Math.random() - 0.5) * 0.1));
                    } else if (typeRoll < 0.33 && gameTimeAsteroids > 15) {
                        asteroids.push(new FallingObjectAsteroids('absorber', Math.random() * (canvasAsteroids.width - 60), -60, 60, currentMinAsteroidSpeed * 0.8, 'rgba(128, 0, 128, 0.7)', (Math.random() - 0.5) * 0.02));
                    } else {
                        asteroids.push(new FallingObjectAsteroids('normal', Math.random() * (canvasAsteroids.width - currentMaxAsteroidSize), -currentMaxAsteroidSize, Math.random() * (currentMaxAsteroidSize - currentMinAsteroidSize) + currentMinAsteroidSize, Math.random() * (currentMaxAsteroidSpeed - currentMinAsteroidSpeed) + currentMinAsteroidSpeed, '#A52A2A', (Math.random() - 0.5) * 0.05));
                    }
                    lastAsteroidSpawnTime = performance.now();
                }

                if (performance.now() - lastHologramSpawnTime > HOLOGRAM_SPAWN_RATE) {
                    holograms.push(new FallingObjectAsteroids('hologram', Math.random() * (canvasAsteroids.width - 40), -40, Math.random() * 30 + 20, Math.random() * 0.8 + 0.2, 'rgba(128, 128, 128, 0.3)', (Math.random() - 0.5) * 0.03));
                    lastHologramSpawnTime = performance.now();
                }

                let newAsteroids = [];
                for (let i = 0; i < asteroids.length; i++) {
                    let asteroid = asteroids[i];
                    asteroid.update();

                    if (asteroid.type === 'normal' && !asteroid.markedForRemoval) {
                        for (let j = 0; j < asteroids.length; j++) {
                            let otherAsteroid = asteroids[j];
                            if (otherAsteroid.type === 'absorber' && !otherAsteroid.markedForRemoval && asteroid !== otherAsteroid) {
                                if (checkCollisionAsteroids(asteroid, otherAsteroid)) {
                                    asteroid.markedForRemoval = true;
                                    break;
                                }
                            }
                        }
                    }

                    if (asteroid.y > canvasAsteroids.height || asteroid.markedForRemoval) {
                        continue;
                    }

                    if (asteroid.type !== 'hologram' && checkCollisionAsteroids(playerAsteroids, asteroid)) {
                        if (!isInvincibleAsteroids) {
                            if (asteroid.type === 'absorber') {
                                playerLivesAsteroids = Math.max(0, playerLivesAsteroids - 2);
                                invincibilityEndTimeAsteroids = performance.now() + 1000;
                            } else {
                                playerLivesAsteroids--;
                                invincibilityEndTimeAsteroids = performance.now() + 3000;
                            }
                            isInvincibleAsteroids = true;

                            updateUIAsteroids();
                            if (playerLivesAsteroids <= 0) {
                                if (gameTimeAsteroids <= 10) { // If player dies very early, consider it a win for testing/fun
                                    endGameAsteroids(true, true);
                                } else {
                                    endGameAsteroids(false);
                                }
                                return;
                            }
                        }
                        continue;
                    }
                    newAsteroids.push(asteroid);
                }
                asteroids = newAsteroids;

                holograms = holograms.filter(hologram => {
                    hologram.update();
                    return hologram.y <= canvasAsteroids.height;
                });
            }

            stars.forEach(star => {
                star.y += star.speed * (deltaTime / 16);
                if (star.y > canvasAsteroids.height) {
                    star.y = 0;
                    star.x = Math.random() * canvasAsteroids.width;
                }
            });

            confetti = confetti.filter(p => {
                p.update();
                return p.alpha > 0 && p.y < canvasAsteroids.height;
            });

            updateUIAsteroids();
        }

        function drawAsteroids() {
            ctxAsteroids.clearRect(0, 0, canvasAsteroids.width, canvasAsteroids.height);

            // Draw stars with parallax
            ctxAsteroids.fillStyle = '#FFFFFF';
            stars.forEach(star => {
                ctxAsteroids.fillRect(star.x, star.y, star.size, star.size);
            });

            if (gameRunningAsteroids) {
                const difficultyPhase = getDifficultyPhaseAsteroids(gameTimeAsteroids);
                let barColor;
                let barHeightPercentage;

                if (difficultyPhase === 1) {
                    barColor = '#00FF00';
                    barHeightPercentage = gameTimeAsteroids / 10;
                } else if (difficultyPhase === 2) {
                    barColor = '#FFFF00';
                    barHeightPercentage = (gameTimeAsteroids - 10) / 20;
                } else {
                    barColor = '#FF0000';
                    barHeightPercentage = (gameTimeAsteroids - 30) / 30;
                }
                barHeightPercentage = Math.min(1, Math.max(0, barHeightPercentage));

                const barWidth = 10;
                const barHeight = canvasAsteroids.height * barHeightPercentage;

                ctxAsteroids.fillStyle = barColor;
                ctxAsteroids.fillRect(0, canvasAsteroids.height - barHeight, barWidth, barHeight);
                ctxAsteroids.fillRect(canvasAsteroids.width - barWidth, canvasAsteroids.height - barHeight, barWidth, barHeight);
            }

            holograms.forEach(hologram => hologram.draw());

            // Draw asteroid shadows
            asteroids.forEach(asteroid => {
                if (asteroid.type !== 'hologram' && !asteroid.markedForRemoval) {
                    ctxAsteroids.save();
                    // Shadow size and opacity based on asteroid's Y position
                    // Ensure shadowScale is never negative
                    const shadowScale = Math.max(0, Math.min(1, (asteroid.y + asteroid.size) / canvasAsteroids.height));
                    const shadowRadiusX = asteroid.size * 0.5 * shadowScale;
                    const shadowRadiusY = asteroid.size * 0.25 * shadowScale;
                    const shadowAlpha = 0.3 + (0.7 * shadowScale); // More opaque as it gets closer
                    
                    ctxAsteroids.fillStyle = `rgba(0, 0, 0, ${shadowAlpha})`;
                    ctxAsteroids.beginPath();
                    // Ensure radii are positive before drawing the ellipse
                    if (shadowRadiusX > 0 && shadowRadiusY > 0) {
                        ctxAsteroids.ellipse(asteroid.x + asteroid.size / 2, canvasAsteroids.height - 15, shadowRadiusX, shadowRadiusY, 0, 0, Math.PI * 2);
                        ctxAsteroids.fill();
                    }
                    ctxAsteroids.restore();
                }
            });

            if (playerAsteroids) {
                playerAsteroids.draw();
            }
            asteroids.forEach(asteroid => asteroid.draw());

            if (performance.now() < currentWarningAsteroids.displayUntil) {
                ctxAsteroids.fillStyle = 'rgba(255, 255, 255, 0.8)';
                ctxAsteroids.font = '24px "Press Start 2P"';
                ctxAsteroids.textAlign = 'center';
                ctxAsteroids.textBaseline = 'middle';
                ctxAsteroids.fillText(currentWarningAsteroids.message, canvasAsteroids.width / 2, canvasAsteroids.height / 2);
            }
            confetti.forEach(p => p.draw());
        }

        startScreenButtonAsteroids.addEventListener('click', startGameAsteroidsLogic);
        backToMenuButtonAsteroids.addEventListener('click', showMainMenu); // Event listener for back button

        canvasAsteroids.addEventListener('touchstart', (e) => {
            e.preventDefault();
        });

        window.addEventListener('keydown', (e) => {
            keysAsteroids[e.key] = true;
        });

        window.addEventListener('keyup', (e) => {
            keysAsteroids[e.key] = false;
        });

        const leftButtonAsteroids = document.getElementById('leftButtonAsteroids');
        const rightButtonAsteroids = document.getElementById('rightButtonAsteroids');

        leftButtonAsteroids.addEventListener('touchstart', (e) => {
            e.preventDefault();
            keysAsteroids['ArrowLeft'] = true;
        });
        leftButtonAsteroids.addEventListener('touchend', () => {
            keysAsteroids['ArrowLeft'] = false;
        });
        leftButtonAsteroids.addEventListener('touchcancel', () => {
            keysAsteroids['ArrowLeft'] = false;
        });

        rightButtonAsteroids.addEventListener('touchstart', (e) => {
            e.preventDefault();
            keysAsteroids['ArrowRight'] = true;
        });
        rightButtonAsteroids.addEventListener('touchend', () => {
            keysAsteroids['ArrowRight'] = false;
        });
        rightButtonAsteroids.addEventListener('touchcancel', () => {
            keysAsteroids['ArrowRight'] = false;
        });

        function resizeCanvasAsteroids() {
            const containerWidth = canvasAsteroids.parentElement.clientWidth;
            const aspectRatio = 640 / 320;
            let newWidth = containerWidth;
            let newHeight = containerWidth / aspectRatio;

            if (newWidth > 640) {
                newWidth = 640;
                newHeight = 320;
            }

            canvasAsteroids.style.width = `${newWidth}px`;
            canvasAsteroids.style.height = `${newHeight}px`;
        }
        window.addEventListener('resize', resizeCanvasAsteroids);


        // --- Lógica del Minijuego Memorama (New) ---
        const gameBoardMemorama = document.getElementById('gameBoardMemorama');
        const timeDisplayMemorama = document.getElementById('timeDisplayMemorama');
        const matchedPairsDisplayMemorama = document.getElementById('matchedPairsDisplayMemorama');
        const totalPairsDisplayMemorama = document.getElementById('totalPairsDisplayMemorama');

        const startScreenMemorama = document.getElementById('startScreenMemorama');
        const startScreenButtonMemorama = document.getElementById('startScreenButtonMemorama');
        const startScreenTitleMemorama = document.getElementById('startScreenTitleMemorama');
        const startScreenMessageMemorama = document.getElementById('startScreenMessageMemorama');
        const gameUIMemorama = document.getElementById('gameUIMemorama');
        const backToMenuButtonMemorama = document.getElementById('backToMenuButtonMemorama');


        let memoramaCards = [];
        let flippedCards = [];
        let matchedPairs = 0;
        let canFlip = true;
        let timerIntervalMemorama;
        let timeElapsedMemorama = 0;
        let gameRunningMemorama = false;
        const GAME_DURATION_MEMORAMA = 60; // Límite de tiempo en segundos
        const LOW_TIME_THRESHOLD_MEMORAMA = 10; // New: Threshold for low time warning

        let memoramaSynth;
        let memoramaSequence;
        let lowTimeSynthMemorama; // New: Synth for low time warning
        let lowTimeSequenceMemorama; // New: Sequence for low time warning
        let isLowTimeMusicPlaying = false; // Flag to track low time music state

        // Aumentada la cantidad de emojis para más cartas (12 pares para 24 cartas)
        const EMOJIS = ['🍎', '🍌', '🍒', '🍇', '🍋', '🥝', '🍓', '🍍', '🍉', '🍊', '🍐', '🍑'];

        function setupAudioMemorama() {
            if (!memoramaSynth) {
                memoramaSynth = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "triangle" },
                    envelope: {
                        attack: 0.005,
                        decay: 0.1,
                        sustain: 0.05,
                        release: 0.2
                    },
                    volume: -10
                }).toDestination();
                memoramaSynth.volume.value = isMasterMuted ? -100 : -15;

                const memoramaNotes = [
                    ["C4", "8n"], ["E4", "8n"], ["G4", "8n"], ["C5", "8n"],
                    ["G3", "8n"], ["B3", "8n"], ["D4", "8n"], ["G4", "8n"]
                ];

                memoramaSequence = new Tone.Sequence((time, note) => {
                    try {
                        memoramaSynth.triggerAttackRelease(note, "8n", time);
                    } catch (e) {
                        console.error("Error processing note in memorama sequence:", note, e);
                    }
                }, memoramaNotes.flat(), "4n");

                Tone.Transport.bpm.value = 70;
                Tone.Transport.loop = true;
            }

            // New: Initialize low time audio
            if (!lowTimeSynthMemorama) {
                lowTimeSynthMemorama = new Tone.Synth({
                    oscillator: { type: "square" },
                    envelope: { attack: 0.01, decay: 0.05, sustain: 0.0, release: 0.1 }
                }).toDestination();
                lowTimeSynthMemorama.volume.value = isMasterMuted ? -100 : -10; // Louder for urgency

                const lowTimeNotes = [
                    ["C4", "16n"], ["F4", "16n"], ["C4", "16n"], ["F4", "16n"],
                    ["C#4", "16n"], ["G4", "16n"], ["C#4", "16n"], ["G4", "16n"]
                ];

                lowTimeSequenceMemorama = new Tone.Sequence((time, note) => {
                    try {
                        lowTimeSynthMemorama.triggerAttackRelease(note, "32n", time);
                    } catch (e) {
                        console.error("Error processing note in low time sequence:", note, e);
                    }
                }, lowTimeNotes.flat(), "8n"); // Faster, more urgent
                lowTimeSequenceMemorama.loop = true;
                lowTimeSequenceMemorama.loopEnd = "1n"; // Loop every quarter note
            }
        }

        function startBackgroundMusicMemorama() {
            if (memoramaSequence && typeof memoramaSequence.start === 'function') {
                memoramaSequence.start(Tone.Transport.now()); // Changed to start at current transport time
                Tone.Transport.bpm.value = 70; // Set BPM for background music
                memoramaSynth.volume.value = isMasterMuted ? -100 : -15;
            }
        }

        function stopBackgroundMusicMemorama() {
            if (memoramaSequence && typeof memoramaSequence.stop === 'function') {
                memoramaSequence.stop();
            }
        }

        // New: Functions to start/stop low time music
        function startLowTimeMusicMemorama() {
            if (lowTimeSequenceMemorama && typeof lowTimeSequenceMemorama.start === 'function') {
                lowTimeSequenceMemorama.start(Tone.Transport.now()); // Changed to start at current transport time
                Tone.Transport.bpm.value = 180; // Set BPM for urgency
                lowTimeSynthMemorama.volume.value = isMasterMuted ? -100 : -10;
            }
        }

        function stopLowTimeMusicMemorama() {
            if (lowTimeSequenceMemorama && typeof lowTimeSequenceMemorama.stop === 'function') {
                lowTimeSequenceMemorama.stop();
            }
        }

        function startGameMemoramaLogic() {
            if (gameRunningMemorama) return;
            startScreenMemorama.classList.add('hidden');
            gameUIMemorama.classList.remove('hidden');
            gameUIMemorama.style.display = 'flex';
            resetGameMemorama();
            gameRunningMemorama = true;

            // Stop other game music before starting this one
            stopMainMenuMusic();
            stopBackgroundMusicAsteroids();

            startBackgroundMusicMemorama();
            timerIntervalMemorama = setInterval(updateTimerMemorama, 1000);
        }

        function resetGameMemorama() {
            clearInterval(timerIntervalMemorama);
            memoramaCards = [];
            flippedCards = [];
            matchedPairs = 0;
            canFlip = true;
            timeElapsedMemorama = 0;
            isLowTimeMusicPlaying = false; // Reset flag
            memoramaGameContainer.classList.remove('low-time-warning'); // Ensure warning is off
            gameBoardMemorama.innerHTML = ''; // Clear previous cards

            startScreenTitleMemorama.textContent = "Memorama";
            startScreenMessageMemorama.textContent = `Encuentra todas las parejas de emojis en ${GAME_DURATION_MEMORAMA} segundos. ¡Entrena tu memoria!`;
            startScreenButtonMemorama.textContent = "¡Comenzar!";

            const shuffledEmojis = [...EMOJIS, ...EMOJIS].sort(() => Math.random() - 0.5);
            totalPairsDisplayMemorama.textContent = EMOJIS.length;

            shuffledEmojis.forEach((emoji, index) => {
                const cardElement = document.createElement('div');
                cardElement.classList.add('card');
                cardElement.dataset.emoji = emoji; // Store emoji in dataset
                cardElement.dataset.index = index;
                cardElement.textContent = '?'; // Initial content is '?'

                cardElement.addEventListener('click', () => flipCard(cardElement, index));
                gameBoardMemorama.appendChild(cardElement);
                memoramaCards.push({ element: cardElement, emoji: emoji, isFlipped: false, isMatched: false });
            });

            updateUIMemorama();
        }

        function flipCard(cardElement, index) {
            if (!canFlip || memoramaCards[index].isFlipped || memoramaCards[index].isMatched) {
                return;
            }

            cardElement.classList.add('flipped');
            cardElement.textContent = cardElement.dataset.emoji; // Show emoji
            memoramaCards[index].isFlipped = true;
            flippedCards.push(memoramaCards[index]);

            if (flippedCards.length === 2) {
                canFlip = false;
                setTimeout(() => {
                    const [card1, card2] = flippedCards;
                    if (card1.emoji === card2.emoji) {
                        // Match!
                        card1.element.classList.add('matched');
                        card2.element.classList.add('matched');
                        card1.isMatched = true;
                        card2.isMatched = true;

                        // Mostrar el checkmark
                        card1.element.textContent = '✔';
                        card1.element.classList.add('card-matched-symbol');
                        card2.element.textContent = '✔';
                        card2.element.classList.add('card-matched-symbol');

                        memoramaSynth.triggerAttackRelease("C5", "8n"); // Play a success sound
                        matchedPairs++;
                        updateUIMemorama();
                        checkWinConditionMemorama();
                    } else {
                        // No match
                        card1.element.classList.remove('flipped');
                        card2.element.classList.remove('flipped');
                        card1.element.textContent = '?'; // Revert to '?'
                        card2.element.textContent = '?'; // Revert to '?'
                        card1.isFlipped = false;
                        card2.isFlipped = false;
                        memoramaSynth.triggerAttackRelease("C3", "8n"); // Play a failure sound
                    }
                    flippedCards = [];
                    canFlip = true;
                }, 1000); // Keep cards flipped for 1 second
            }
        }

        function updateTimerMemorama() {
            timeElapsedMemorama++;
            const timeLeft = GAME_DURATION_MEMORAMA - timeElapsedMemorama;
            timeDisplayMemorama.textContent = `${Math.max(0, timeLeft)}s`;

            // New: Low time warning logic
            if (timeLeft <= LOW_TIME_THRESHOLD_MEMORAMA && timeLeft > 0) {
                memoramaGameContainer.classList.add('low-time-warning');
                if (!isLowTimeMusicPlaying) {
                    stopBackgroundMusicMemorama(); // Stop regular background music
                    startLowTimeMusicMemorama();
                    isLowTimeMusicPlaying = true;
                }
            } else {
                memoramaGameContainer.classList.remove('low-time-warning');
                if (isLowTimeMusicPlaying) {
                    stopLowTimeMusicMemorama();
                    startBackgroundMusicMemorama(); // Resume regular background music
                    isLowTimeMusicPlaying = false;
                }
            }

            if (timeLeft <= 0 && matchedPairs < EMOJIS.length) {
                endGameMemorama(false); // Perdió por tiempo
            }
        }

        function updateUIMemorama() {
            const timeLeft = GAME_DURATION_MEMORAMA - timeElapsedMemorama;
            timeDisplayMemorama.textContent = `${Math.max(0, timeLeft)}s`;
            matchedPairsDisplayMemorama.textContent = matchedPairs;
        }

        function checkWinConditionMemorama() {
            if (matchedPairs === EMOJIS.length) {
                endGameMemorama(true);
            }
        }

        function endGameMemorama(won) {
            gameRunningMemorama = false;
            clearInterval(timerIntervalMemorama);
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama(); // Ensure low time music stops
            memoramaGameContainer.classList.remove('low-time-warning'); // Ensure visual warning is off
            gameUIMemorama.style.display = 'none';

            let title, message;
            if (won) {
                title = "¡FELICIDADES!";
                message = `¡Has encontrado todas las parejas en ${timeElapsedMemorama} segundos!`;
            } else {
                title = "¡TIEMPO AGOTADO!";
                message = `No lograste encontrar todas las parejas a tiempo. Te quedaste en ${matchedPairs} pares.`;
            }
            showModal(title, message, "Volver al Menú Principal", showMainMenu);
        }

        startScreenButtonMemorama.addEventListener('click', startGameMemoramaLogic);
        backToMenuButtonMemorama.addEventListener('click', showMainMenu); // Event listener for back button


        // --- Lógica de Inicio y Cambio de Juegos ---
        startGameAsteroidsButton.addEventListener('click', () => {
            hideMainMenu();
            hideAllGames(); // Ensure all other games are hidden
            asteroidGameContainer.classList.remove('hidden'); // Show asteroid game
            
            // Re-inicializa el juego de asteroides para mostrar su pantalla de inicio
            resetGameAsteroids();
            setupAudioAsteroids(); // Configura el audio para Asteroides
            resizeCanvasAsteroids(); // Redimensiona el canvas para Asteroids
        });

        startGameMemoramaButton.addEventListener('click', () => {
            hideMainMenu();
            hideAllGames(); // Ensure all other games are hidden
            memoramaGameContainer.classList.remove('hidden'); // Show memorama game

            // Re-inicializa el juego de memorama para mostrar su pantalla de inicio
            resetGameMemorama();
            setupAudioMemorama(); // Configura el audio para Memorama
            // No hay canvas para Memorama, así que no se necesita resizeCanvasMemorama
        });

        // Configuración inicial al cargar la página principal
        window.onload = () => {
            // Start Tone.Transport once on user interaction to comply with autoplay policies
            document.body.addEventListener('click', () => {
                if (Tone.context.state !== 'running') {
                    Tone.start();
                    Tone.Transport.start(); // Start global transport once
                    startMainMenuMusic(); // Start main menu music after transport is running
                }
            }, { once: true }); // Only run this listener once

            setupMainMenuAudio(); // Inicializar audio del menú principal
            setupAudioAsteroids(); // Initialize audio components for Asteroids
            setupAudioMemorama(); // Initialize audio components for Memorama
            showMainMenu(); // Ensure main menu is visible at start and music starts
            hideAllGames(); // Ensure no games are visible initially, only main menu
        };
    </script>
</body>
</html>
