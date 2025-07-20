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
    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, orderBy, limit, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase variables (will be populated by Canvas environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let app;
        let db;
        let auth;
        let currentUserId = null;

        // Initialize Firebase and Auth
        if (Object.keys(firebaseConfig).length > 0) {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);

            onAuthStateChanged(auth, async (user) => {
                if (user) {
                    currentUserId = user.uid;
                    const userIdDisplayElement = document.getElementById('userIdDisplay');
                    if (userIdDisplayElement) {
                        userIdDisplayElement.textContent = `Tu ID: ${currentUserId}`;
                    } else {
                        console.warn("Element with ID 'userIdDisplay' not found.");
                    }
                    console.log("User signed in:", currentUserId);
                } else {
                    console.log("No user signed in. Attempting anonymous sign-in...");
                    try {
                        if (initialAuthToken) {
                            await signInWithCustomToken(auth, initialAuthToken);
                        } else {
                            await signInAnonymously(auth);
                        }
                    } catch (error) {
                        console.error("Error during Firebase authentication:", error);
                    }
                }
            });
        } else {
            console.warn("Firebase config not found. Leaderboard functionality will be disabled.");
            const userIdDisplayElement = document.getElementById('userIdDisplay');
            if (userIdDisplayElement) {
                userIdDisplayElement.textContent = 'Firebase no configurado.';
            } else {
                console.warn("Element with ID 'userIdDisplay' not found.");
            }
        }

        // Expose to global scope for use in the main script
        window.firebase = {
            app, db, auth, currentUserId,
            getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged,
            getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, orderBy, limit, getDocs,
            appId // Expose appId for collection paths
        };
    </script>
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
        /* Debug display for keys (new) */
        #keyDebugDisplay {
            @apply text-sm text-gray-400 mt-2 text-center;
            min-height: 20px; /* Ensure space even when empty */
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
            line-height: 1; /* Asegura que el contenido esté centrado verticalmente */
        }
        #memoramaGame .card.flipped {
            transform: rotateY(180deg);
            box-shadow: 2px 2px 0px #2d3748; /* Sombra reducida al voltear */
            background-color: #4299e1; /* Fondo azul al voltear */
            font-size: 2.8rem; /* Reducido para que el emoji no parezca agrandarse */
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
                font-size: 3rem; /* Ajuste para móviles para el signo de interrogación */
            }
            #memoramaGame .card.flipped {
                font-size: 2.3rem; /* Ajuste para móviles para el emoji */
            }
            #memoramaGame .game-board {
                max-width: 300px; /* Ancho máximo del tablero para móviles */
                grid-gap: 10px; /* Ajuste de gap para móviles */
            }
        }

        /* Nueva clase para el símbolo de verificación */
        #memoramaGame .card-matched-symbol {
            color: #48bb78; /* Verde */
            font-size: 2.8rem; /* Tamaño grande para el checkmark, ajustado al tamaño del emoji volteado */
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
        @media (max-width: 640px) {
            #memoramaGame .card-matched-symbol {
                font-size: 2.3rem; /* Ajuste para móviles */
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

        /* --- Estilos Específicos para Ortografía Correcta (Nuevo Minijuego) --- */
        #spellingGame .game-container {
            max-width: 600px;
        }
        #spellingGame #wordDisplay {
            @apply text-5xl sm:text-6xl font-bold mb-8 text-white text-center;
            min-height: 100px; /* Asegura un espacio para la palabra */
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 10px;
            background-color: #1a202c;
            border-radius: 8px;
            border: 3px solid #4a5568;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
            word-break: break-word; /* Permite que las palabras largas se rompan */
        }
        #spellingGame .answer-buttons {
            @apply flex flex-col sm:flex-row gap-4 sm:gap-8 mt-6 w-full justify-center;
        }
        #spellingGame .answer-button {
            @apply px-8 py-4 text-2xl sm:text-3xl rounded-xl font-bold transition-all duration-200;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
            letter-spacing: 0.05em;
            flex-grow: 1; /* Permite que los botones crezcan */
            max-width: 200px; /* Limita el ancho máximo */
            min-width: 150px; /* Asegura un ancho mínimo */
        }
        #spellingGame #correctButton {
            background-image: linear-gradient(to bottom right, #48bb78, #38a169); /* Verde */
            border: 2px solid #2f855a;
            box-shadow: 6px 6px 0px #2f855a;
            color: #fff;
        }
        #spellingGame #correctButton:hover {
            background-image: linear-gradient(to bottom right, #38a169, #2f855a);
            box-shadow: 3px 3px 0px #2f855a;
            transform: translate(3px, 3px) scale(1.02);
        }
        #spellingGame #correctButton:active {
            background-image: linear-gradient(to bottom right, #2f855a, #276749);
            box-shadow: none;
            transform: translate(6px, 6px) scale(0.98);
        }

        #spellingGame #incorrectButton {
            background-image: linear-gradient(to bottom right, #ef4444, #dc2626); /* Rojo */
            border: 2px solid #b91c1c;
            box-shadow: 6px 6px 0px #b91c1c;
            color: #fff;
        }
        #spellingGame #incorrectButton:hover {
            background-image: linear-gradient(to bottom right, #dc2626, #b91c1c);
            box-shadow: 3px 3px 0px #b91c1c;
            transform: translate(3px, 3px) scale(1.02);
        }
        #spellingGame #incorrectButton:active {
            background-image: linear-gradient(to bottom right, #b91c1c, #991b1b);
            box-shadow: none;
            transform: translate(6px, 6px) scale(0.98);
        }

        /* Loading spinner */
        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #4299e1;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Estilos para la tabla de resultados */
        .results-table {
            @apply w-full text-left border-collapse mt-6;
            font-size: 0.9rem; /* Slightly smaller font for table */
        }
        .results-table th, .results-table td {
            @apply p-2 border-b border-gray-600;
        }
        .results-table th {
            @apply bg-gray-700 text-gray-200 uppercase font-bold;
        }
        .results-table tr:nth-child(even) {
            @apply bg-gray-700;
        }
        .results-table tr:hover {
            @apply bg-gray-600;
        }
        .results-container {
            @apply w-full p-4 bg-gray-800 rounded-lg shadow-inner mt-4;
            border: 2px solid #4a5568;
        }
        /* Style for in-game messages */
        .game-message {
            @apply absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-blue-700 text-white text-3xl sm:text-4xl font-bold px-6 py-3 rounded-lg shadow-lg z-10;
            animation: fadeOut 2s forwards;
            opacity: 0; /* Start hidden */
            display: none; /* Hidden by default */
        }

        @keyframes fadeOut {
            0% { opacity: 1; display: block; }
            80% { opacity: 1; display: block; }
            100% { opacity: 0; display: none; }
        }

        /* --- Estilos Específicos para Sumas Rápidas (Nuevo Minijuego) --- */
        #quickMathGame .game-container {
            max-width: 600px;
        }
        #quickMathGame #problemDisplay {
            @apply text-5xl sm:text-6xl font-bold mb-8 text-white text-center;
            min-height: 100px;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 10px;
            background-color: #1a202c;
            border-radius: 8px;
            border: 3px solid #4a5568;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
        }
        #quickMathGame #answerInput {
            @apply w-full max-w-xs p-4 text-center text-4xl rounded-lg bg-gray-700 text-white border-2 border-blue-500 focus:outline-none focus:border-blue-300 transition-colors duration-200;
            box-shadow: inset 0 0 8px rgba(0,0,0,0.4);
            font-family: 'Press Start 2P', cursive;
        }
        #quickMathGame #submitAnswerButton {
            @apply mt-6 px-8 py-4 text-2xl sm:text-3xl rounded-xl font-bold transition-all duration-200;
            background-image: linear-gradient(to bottom right, #63b3ed, #4299e1); /* Azul claro */
            color: #fff;
            border: 2px solid #2b6cb0;
            box-shadow: 6px 6px 0px #2b6cb0;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
            letter-spacing: 0.05em;
        }
        #quickMathGame #submitAnswerButton:hover {
            background-image: linear-gradient(to bottom right, #4299e1, #3182ce);
            box-shadow: 3px 3px 0px #2b6cb0;
            transform: translate(3px, 3px) scale(1.02);
        }
        #quickMathGame #submitAnswerButton:active {
            background-image: linear-gradient(to bottom right, #3182ce, #2c5282);
            box-shadow: none;
            transform: translate(6px, 6px) scale(0.98);
        }

        /* New: Leaderboard Submission Modal */
        #leaderboardSubmissionModal .modal-content {
            max-width: 400px;
        }
        #leaderboardSubmissionModal input[type="text"] {
            @apply w-full p-3 text-center text-xl rounded-lg bg-gray-700 text-white border-2 border-blue-500 focus:outline-none focus:border-blue-300;
            font-family: 'Press Start 2P', cursive;
            margin-bottom: 1rem;
        }
        #leaderboardSubmissionModal .game-button {
            @apply px-6 py-3 text-lg;
        }
        #userIdDisplay {
            @apply text-xs text-gray-400 mt-4 text-center;
        }
        /* Version display at the bottom */
        #versionDisplay {
            @apply text-xs text-gray-500 mt-4 mb-2;
        }
    </style>
</head>
<body>
    <div class="main-app-container">
        <h1 id="mainTitle" class="text-4xl font-bold mb-8 text-white text-center">🎮 Mejora con Juegos 🚀</h1>

        <!-- Botón de prueba de JavaScript (NUEVO) -->
        <button id="testJsButton" class="app-button mb-8" onclick="console.log('Botón de prueba de JS clicado!'); alert('Botón de prueba de JS clicado!');">
            Test JS Button
        </button>

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

        <!-- Divisor entre minijuegos -->
        <div class="w-full h-2 my-8 rounded-full shadow-lg" style="background-image: linear-gradient(to right, transparent, #FFD700, #FF6347, transparent);"></div>

        <!-- Sección de Ortografía Correcta (Nuevo Minijuego) -->
        <p id="spellingGameDescription" class="game-description">
            ¡Ortografía Correcta! ✍️ ¿Está bien escrita la palabra? ¡Demuestra tu conocimiento y mejora tu ortografía! 📚
        </p>
        <button id="startGameSpellingButton" class="app-button mb-8">Jugar Ortografía Correcta</button>

        <!-- Divisor entre minijuegos -->
        <div class="w-full h-2 my-8 rounded-full shadow-lg" style="background-image: linear-gradient(to right, transparent, #48bb78, #3182ce, transparent);"></div>

        <!-- Sección de Sumas Rápidas (Nuevo Minijuego) -->
        <p id="quickMathGameDescription" class="game-description">
            ¡Sumas Rápidas! ➕ Resuelve problemas de suma lo más rápido posible. ¡Agiliza tu mente y mejora tu cálculo mental! ⚡
        </p>
        <button id="startGameQuickMathButton" class="app-button mb-8">Jugar Sumas Rápidas</button>


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
                    <!-- Debug display for keys (new) -->
                    <div id="keyDebugDisplay"></div>
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

            <!-- Contenido del juego de Ortografía Correcta (Nuevo) -->
            <div id="spellingGame" class="game-container hidden">
                <h1 class="text-3xl font-bold mb-4 text-white">Ortografía Correcta</h1>
                <div id="gameUISpelling" class="flex flex-col items-center w-full hidden">
                    <div class="info-panel">
                        <span>Tiempo: <span id="timeDisplaySpelling">0s</span></span>
                        <span>Puntuación: <span id="scoreDisplaySpelling">0</span></span>
                        <span>Vidas: <span id="livesDisplaySpelling">5</span></span> <!-- New lives display -->
                    </div>
                    <div id="wordDisplay" class="mb-6">Cargando palabra...</div>
                    <div class="answer-buttons">
                        <button id="correctButton" class="answer-button">Correcto</button>
                        <button id="incorrectButton" class="answer-button">Incorrecto</button>
                    </div>
                    <button id="backToMenuButtonSpelling" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <div id="startScreenSpelling">
                    <h2 id="startScreenTitleSpelling" class="text-4xl font-bold mb-6 text-white text-center">Ortografía Correcta</h2>
                    <p id="startScreenMessageSpelling" class="text-lg text-gray-300 mb-8 text-center px-4">Se te mostrarán palabras. Decide si están bien escritas o no.</p>
                    <div class="flex flex-col sm:flex-row gap-4 sm:gap-6 mb-6">
                        <button id="combinedLevelButtonSpelling" class="game-button text-xl px-6 py-3">Iniciar Nivel Combinado (5 vidas)</button> <!-- New combined level button -->
                        <button id="freeModeButtonSpelling" class="game-button text-xl px-6 py-3">Modo Libre (30s)</button>
                    </div>
                    <div id="spellingLoadingIndicator" class="spinner hidden"></div>
                    <p id="spellingLoadingMessage" class="text-gray-400 text-sm mt-4 hidden">Generando palabras con IA...</p>
                    <div id="spellingResultsContainer" class="results-container hidden">
                        <h3 class="text-2xl font-bold mb-4 text-white">Mejores Puntuaciones Locales</h3>
                        <table id="spellingResultsTableLocal" class="results-table">
                            <thead>
                                <tr>
                                    <th>Modo/Nivel</th>
                                    <th>Puntuación</th>
                                </tr>
                            </thead>
                            <tbody>
                                <!-- Resultados locales se insertarán aquí -->
                            </tbody>
                        </table>
                        <h3 class="text-2xl font-bold mb-4 mt-6 text-white">Mejores Puntuaciones Globales (Modo Libre)</h3>
                        <table id="spellingResultsTableGlobal" class="results-table">
                            <thead>
                                <tr>
                                    <th>Nombre</th>
                                    <th>Puntuación</th>
                                    <th>Fecha</th>
                                </tr>
                            </thead>
                            <tbody>
                                <!-- Resultados globales se insertarán aquí -->
                            </tbody>
                        </table>
                    </div>
                    <button id="backToMenuButtonSpellingStartScreen" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <!-- New element for in-game messages -->
                <div id="spellingGameMessage" class="game-message hidden"></div>
            </div>

            <!-- Contenido del juego de Sumas Rápidas (Nuevo) -->
            <div id="quickMathGame" class="game-container hidden">
                <h1 class="text-3xl font-bold mb-4 text-white">Sumas Rápidas</h1>
                <div id="gameUIQuickMath" class="flex flex-col items-center w-full hidden">
                    <div class="info-panel">
                        <span>Tiempo: <span id="timeDisplayQuickMath">0s</span></span>
                        <span>Puntuación: <span id="scoreDisplayQuickMath">0</span></span>
                        <span>Vidas: <span id="livesDisplayQuickMath">3</span></span>
                    </div>
                    <div id="problemDisplay" class="mb-6">Cargando problema...</div>
                    <input type="number" id="answerInput" class="mb-6" placeholder="Tu respuesta">
                    <button id="submitAnswerButton" class="game-button">Responder</button>
                    <button id="backToMenuButtonQuickMath" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <div id="startScreenQuickMath">
                    <h2 id="startScreenTitleQuickMath" class="text-4xl font-bold mb-6 text-white text-center">Sumas Rápidas</h2>
                    <p id="startScreenMessageQuickMath" class="text-lg text-gray-300 mb-8 text-center px-4">Resuelve problemas de suma lo más rápido posible. ¡Agiliza tu mente y mejora tu cálculo mental!</p>
                    <div class="flex flex-col sm:flex-row gap-4 sm:gap-6 mb-6">
                        <button id="combinedLevelButtonQuickMath" class="game-button text-xl px-6 py-3">Iniciar Nivel Combinado (5 vidas)</button>
                        <button id="freeModeButtonQuickMath" class="game-button text-xl px-6 py-3">Modo Libre (60s)</button>
                    </div>
                    <div id="quickMathResultsContainer" class="results-container hidden">
                        <h3 class="text-2xl font-bold mb-4 text-white">Mejores Puntuaciones Locales</h3>
                        <table id="quickMathResultsTableLocal" class="results-table">
                            <thead>
                                <tr>
                                    <th>Modo/Nivel</th>
                                    <th>Puntuación</th>
                                </tr>
                            </thead>
                            <tbody>
                                <!-- Resultados locales se insertarán aquí -->
                            </tbody>
                        </table>
                        <h3 class="text-2xl font-bold mb-4 mt-6 text-white">Mejores Puntuaciones Globales (Modo Libre)</h3>
                        <table id="quickMathResultsTableGlobal" class="results-table">
                            <thead>
                                <tr>
                                    <th>Nombre</th>
                                    <th>Puntuación</th>
                                    <th>Fecha</th>
                                </tr>
                            </thead>
                            <tbody>
                                <!-- Resultados globales se insertarán aquí -->
                            </tbody>
                        </table>
                    </div>
                    <button id="backToMenuButtonQuickMathStartScreen" class="game-button mt-10">Volver al Menú Principal</button>
                </div>
                <!-- New element for in-game messages -->
                <div id="quickMathGameMessage" class="game-message hidden"></div>
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

    <!-- New: Leaderboard Submission Modal -->
    <div id="leaderboardSubmissionModal" class="modal hidden">
        <div class="modal-content">
            <h2 class="modal-title">¡Nuevo Récord!</h2>
            <p id="leaderboardMessage" class="modal-message">¡Felicidades! Has logrado una nueva puntuación alta. Ingresa tu nombre para el ranking global:</p>
            <input type="text" id="playerNameInput" placeholder="Tu nombre" maxlength="20">
            <button id="submitLeaderboardScoreButton" class="game-button">Publicar Puntuación</button>
            <button id="cancelLeaderboardSubmissionButton" class="game-button mt-2">No Publicar</button>
        </div>
    </div>

    <!-- User ID Display -->
    <div id="userIdDisplay" class="text-xs text-gray-400 mt-4 text-center">Cargando ID de usuario...</div>

    <!-- Version Display (New) -->
    <div id="versionDisplay" class="text-xs text-gray-500 mt-4 mb-2">v1.0.0</div>

    <script type="module">
        // Import Firebase functions from the global 'firebase' object exposed by the script above
        const { db, auth, appId, onAuthStateChanged, signInAnonymously, signInWithCustomToken,
                getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, orderBy, limit, getDocs } = window.firebase;

        let currentUserId = null; // Will be set by onAuthStateChanged listener

        onAuthStateChanged(auth, (user) => {
            if (user) {
                currentUserId = user.uid;
                const userIdDisplayElement = document.getElementById('userIdDisplay');
                if (userIdDisplayElement) {
                    userIdDisplayElement.textContent = `Tu ID: ${currentUserId}`;
                } else {
                    console.warn("Element with ID 'userIdDisplay' not found on auth state change.");
                }
                console.log("User ID set:", currentUserId);
            } else {
                currentUserId = null;
                const userIdDisplayElement = document.getElementById('userIdDisplay');
                if (userIdDisplayElement) {
                    userIdDisplayElement.textContent = 'ID no disponible (error de autenticación).';
                } else {
                    console.warn("Element with ID 'userIdDisplay' not found on auth state change (user null).");
                }
            }
        });

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
        const spellingGameDescription = document.getElementById('spellingGameDescription');
        const startGameSpellingButton = document.getElementById('startGameSpellingButton');
        const quickMathGameDescription = document.getElementById('quickMathGameDescription');
        const startGameQuickMathButton = document.getElementById('startGameQuickMathButton');
        const gameArea = document.getElementById('gameArea');
        const asteroidGameContainer = document.getElementById('asteroidGame');
        const memoramaGameContainer = document.getElementById('memoramaGame');
        const spellingGameContainer = document.getElementById('spellingGame');
        const quickMathGameContainer = document.getElementById('quickMathGame');
        const globalMuteButton = document.getElementById('globalMuteButton');

        // Modal elements
        const gameModal = document.getElementById('gameModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalButton = document.getElementById('modalButton');

        // Leaderboard Submission Modal elements
        const leaderboardSubmissionModal = document.getElementById('leaderboardSubmissionModal');
        const playerNameInput = document.getElementById('playerNameInput');
        const submitLeaderboardScoreButton = document.getElementById('submitLeaderboardScoreButton');
        const cancelLeaderboardSubmissionButton = document.getElementById('cancelLeaderboardSubmissionButton');
        let currentScoreToSubmit = { game: '', score: 0, time: 0 }; // To hold score data for submission

        function showModal(title, message, buttonText = "Cerrar", callback = null) {
            if (!modalTitle || !modalMessage || !modalButton || !gameModal) {
                console.error("Modal elements not found for showModal.");
                alert(`Modal Error: ${title} - ${message}`); // Fallback to alert if modal elements are missing
                if (callback) callback();
                return;
            }
            modalTitle.textContent = title;
            modalMessage.textContent = message;
            modalButton.textContent = buttonText;
            modalButton.onclick = () => {
                gameModal.classList.add('hidden');
                if (callback) callback();
            };
            gameModal.classList.remove('hidden');
        }

        function showLeaderboardSubmissionModal(game, score, time) {
            if (!leaderboardSubmissionModal || !playerNameInput || !submitLeaderboardScoreButton || !cancelLeaderboardSubmissionButton) {
                console.error("Leaderboard submission modal elements not found.");
                showModal('Error', 'No se puede mostrar el modal de puntuación alta.', 'Cerrar', showMainMenu);
                return;
            }
            currentScoreToSubmit = { game, score, time };
            playerNameInput.value = ''; // Clear previous input
            leaderboardSubmissionModal.classList.remove('hidden');
            playerNameInput.focus();
        }

        function hideLeaderboardSubmissionModal() {
            if (leaderboardSubmissionModal) {
                leaderboardSubmissionModal.classList.add('hidden');
            } else {
                console.warn("Leaderboard submission modal element not found for hiding.");
            }
        }

        if (submitLeaderboardScoreButton) {
            submitLeaderboardScoreButton.addEventListener('click', async () => {
                const playerName = playerNameInput.value.trim();
                if (playerName.length < 1) {
                    showModal("Nombre Requerido", "Por favor, ingresa un nombre para el ranking.", "Entendido");
                    return;
                }
                if (!db || !currentUserId) {
                    showModal('Error', 'Firebase no está inicializado o no hay usuario autenticado. No se puede guardar la puntuación.', 'Cerrar', showMainMenu);
                    hideLeaderboardSubmissionModal();
                    return;
                }

                const scoreData = {
                    name: playerName,
                    score: currentScoreToSubmit.score,
                    time: currentScoreToSubmit.time,
                    timestamp: Date.now(),
                    userId: currentUserId // Store user ID for potential future features
                };

                let collectionPath;
                if (currentScoreToSubmit.game === 'spellingFreeMode') {
                    collectionPath = `artifacts/${appId}/public/data/spellingFreeModeLeaderboard`;
                } else if (currentScoreToSubmit.game === 'quickMathFreeMode') {
                    collectionPath = `artifacts/${appId}/public/data/quickMathFreeModeLeaderboard`;
                } else {
                    console.error("Unknown game mode for leaderboard submission:", currentScoreToSubmit.game);
                    showModal('Error', 'Modo de juego desconocido para el ranking.', 'Cerrar', showMainMenu);
                    hideLeaderboardSubmissionModal();
                    return;
                }

                try {
                    await addDoc(collection(db, collectionPath), scoreData);
                    hideLeaderboardSubmissionModal();
                    showModal('¡Puntuación Publicada!', `Tu puntuación ha sido publicada como ${playerName}.`, 'Cerrar', showMainMenu);
                } catch (e) {
                    console.error("Error adding document: ", e);
                    showModal('Error al Publicar', 'No se pudo publicar tu puntuación. Intenta de nuevo.', 'Cerrar', showMainMenu);
                    hideLeaderboardSubmissionModal();
                }
            });
        } else {
            console.warn("Element with ID 'submitLeaderboardScoreButton' not found.");
        }

        if (cancelLeaderboardSubmissionButton) {
            cancelLeaderboardSubmissionButton.addEventListener('click', () => {
                hideLeaderboardSubmissionModal();
                showMainMenu(); // Go back to main menu if user cancels submission
            });
        } else {
            console.warn("Element with ID 'cancelLeaderboardSubmissionButton' not found.");
        }


        function hideMainMenu() {
            if (mainTitle) mainTitle.style.display = 'none'; else console.warn("mainTitle not found.");
            if (asteroidGameDescription) asteroidGameDescription.style.display = 'none'; else console.warn("asteroidGameDescription not found.");
            if (startGameAsteroidsButton) startGameAsteroidsButton.style.display = 'none'; else console.warn("startGameAsteroidsButton not found.");
            if (memoramaGameDescription) memoramaGameDescription.style.display = 'none'; else console.warn("memoramaGameDescription not found.");
            if (startGameMemoramaButton) startGameMemoramaButton.style.display = 'none'; else console.warn("startGameMemoramaButton not found.");
            if (spellingGameDescription) spellingGameDescription.style.display = 'none'; else console.warn("spellingGameDescription not found.");
            if (startGameSpellingButton) startGameSpellingButton.style.display = 'none'; else console.warn("startGameSpellingButton not found.");
            if (quickMathGameDescription) quickMathGameDescription.style.display = 'none'; else console.warn("quickMathGameDescription not found.");
            if (startGameQuickMathButton) startGameQuickMathButton.style.display = 'none'; else console.warn("startGameQuickMathButton not found.");
            if (globalMuteButton) globalMuteButton.style.display = 'none'; else console.warn("globalMuteButton not found.");
            if (gameArea) gameArea.style.display = 'flex'; else console.warn("gameArea not found.");
            stopMainMenuMusic();
        }

        function showMainMenu() {
            if (mainTitle) mainTitle.style.display = 'block';
            if (asteroidGameDescription) asteroidGameDescription.style.display = 'block';
            if (startGameAsteroidsButton) startGameAsteroidsButton.style.display = 'block';
            if (memoramaGameDescription) memoramaGameDescription.style.display = 'block';
            if (startGameMemoramaButton) startGameMemoramaButton.style.display = 'block';
            if (spellingGameDescription) spellingGameDescription.style.display = 'block';
            if (startGameSpellingButton) startGameSpellingButton.style.display = 'block';
            if (quickMathGameDescription) quickMathGameDescription.style.display = 'block';
            if (startGameQuickMathButton) startGameQuickMathButton.style.display = 'block';
            if (globalMuteButton) globalMuteButton.style.display = 'block';
            if (gameArea) gameArea.style.display = 'none';

            // Ensure all game containers are hidden when returning to main menu
            stopGameAsteroidsInternal();
            if (asteroidGameContainer) asteroidGameContainer.classList.add('hidden');
            stopGameMemoramaInternal();
            if (memoramaGameContainer) memoramaGameContainer.classList.add('hidden');
            stopGameSpellingInternal();
            if (spellingGameContainer) spellingGameContainer.classList.add('hidden');
            stopGameQuickMathInternal();
            if (quickMathGameContainer) quickMathGameContainer.classList.add('hidden');

            if (Tone.context.state === 'running') {
                startMainMenuMusic();
            }
        }

        // Internal stop function for Asteroids
        function stopGameAsteroidsInternal() {
            gameRunningAsteroids = false;
            cancelAnimationFrame(animationFrameIdAsteroids);
            stopBackgroundMusicAsteroids();
            if (gameUIAsteroids) gameUIAsteroids.style.display = 'none';
            if (startScreenAsteroids) startScreenAsteroids.classList.remove('hidden');
            // Clear keys state on game stop
            keysAsteroids = {};
            updateKeyDebugDisplay();
        }

        // Internal stop function for Memorama
        function stopGameMemoramaInternal() {
            gameRunningMemorama = false;
            clearInterval(timerIntervalMemorama);
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama();
            if (memoramaGameContainer) memoramaGameContainer.classList.remove('low-time-warning');
            if (gameUIMemorama) gameUIMemorama.style.display = 'none';
            if (startScreenMemorama) startScreenMemorama.classList.remove('hidden');
        }

        // Internal stop function for Spelling Game
        function stopGameSpellingInternal() {
            gameRunningSpelling = false;
            clearInterval(timerIntervalSpelling);
            stopBackgroundMusicSpelling();
            if (wordDisplay) wordDisplay.textContent = 'Cargando palabra...'; // Reset text
            if (gameUISpelling) gameUISpelling.classList.add('hidden');
            if (startScreenSpelling) startScreenSpelling.classList.remove('hidden');
            if (spellingLoadingIndicator) spellingLoadingIndicator.classList.add('hidden');
            if (spellingLoadingMessage) spellingLoadingMessage.classList.add('hidden');
            // Re-enable all spelling game start buttons
            if (combinedLevelButtonSpelling) combinedLevelButtonSpelling.disabled = false;
            if (freeModeButtonSpelling) freeModeButtonSpelling.disabled = false;
            if (backToMenuButtonSpellingStartScreen) backToMenuButtonSpellingStartScreen.disabled = false;

            updateSpellingResultsTable(); // Show latest results on start screen
            if (spellingResultsContainer) spellingResultsContainer.classList.remove('hidden'); // Show results table
        }

        // Internal stop function for Quick Math Game (New)
        function stopGameQuickMathInternal() {
            quickMathGameRunning = false;
            clearInterval(quickMathTimerInterval);
            stopBackgroundMusicQuickMath();
            if (problemDisplay) problemDisplay.textContent = 'Cargando problema...'; // Reset text
            if (answerInput) answerInput.value = ''; // Clear input
            if (gameUIQuickMath) gameUIQuickMath.classList.add('hidden');
            if (startScreenQuickMath) startScreenQuickMath.classList.remove('hidden');
            // Re-enable all quick math game start buttons
            if (combinedLevelButtonQuickMath) combinedLevelButtonQuickMath.disabled = false;
            if (freeModeButtonQuickMath) freeModeButtonQuickMath.disabled = false;
            if (backToMenuButtonQuickMathStartScreen) backToMenuButtonQuickMathStartScreen.disabled = false;

            updateQuickMathResultsTable(); // Show latest results on start screen
            if (quickMathResultsContainer) quickMathResultsContainer.classList.remove('hidden'); // Show results table
        }


        // Function to hide all game containers (used when switching games or returning to main menu)
        function hideAllGames() {
            if (asteroidGameContainer) asteroidGameContainer.classList.add('hidden');
            if (memoramaGameContainer) memoramaGameContainer.classList.add('hidden');
            if (spellingGameContainer) spellingGameContainer.classList.add('hidden');
            if (quickMathGameContainer) quickMathGameContainer.classList.add('hidden');
            // Stop any running game processes when hiding all
            stopGameAsteroidsInternal();
            stopGameMemoramaInternal();
            stopGameSpellingInternal();
            stopGameQuickMathInternal();
        }

        // --- Lógica de Audio Global y del Menú Principal ---
        function setupMainMenuAudio() {
            if (!mainMenuSynth) {
                mainMenuSynth = new Tone.Synth({
                    oscillator: { type: "triangle" },
                    envelope: { attack: 0.02, decay: 0.5, sustain: 0.1, release: 0.8 }
                }).toDestination();
                mainMenuSynth.volume.value = -25;

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
            if (Tone.context.state === 'running' && mainMenuSequence && typeof mainMenuSequence.start === 'function') {
                mainMenuSequence.start(Tone.Transport.now());
                mainMenuSynth.volume.value = isMasterMuted ? -100 : -25;
                Tone.Transport.bpm.value = 60;
            }
        }

        function stopMainMenuMusic() {
            if (mainMenuSequence && typeof mainMenuSequence.stop === 'function') {
                mainMenuSequence.stop();
            }
        }

        if (globalMuteButton) {
            globalMuteButton.addEventListener('click', toggleGlobalMute);
        } else {
            console.warn("Element with ID 'globalMuteButton' not found.");
        }


        function toggleGlobalMute() {
            isMasterMuted = !isMasterMuted;
            if (globalMuteButton) {
                globalMuteButton.textContent = isMasterMuted ? "Activar Sonido" : "Silenciar Todo";
            }

            // Actualizar volumen de todos los synths
            if (mainMenuSynth) mainMenuSynth.volume.value = isMasterMuted ? -100 : -25;
            if (backgroundSynthAsteroids) backgroundSynthAsteroids.volume.value = isMasterMuted ? -100 : -15;
            if (memoramaSynth) memoramaSynth.volume.value = isMasterMuted ? -100 : -15;
            if (lowTimeSynthMemorama) lowTimeSynthMemorama.volume.value = isMasterMuted ? -100 : -10;
            if (spellingCorrectSynth) spellingCorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            if (spellingIncorrectSynth) spellingIncorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            if (backgroundSynthSpelling) backgroundSynthSpelling.volume.value = isMasterMuted ? -100 : -20;
            if (quickMathCorrectSynth) quickMathCorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            if (quickMathIncorrectSynth) quickMathIncorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            if (quickMathBackgroundSynth) quickMathBackgroundSynth.volume.value = isMasterMuted ? -100 : -20;
        }


        // --- Lógica del Minijuego Esquiva los Asteroides ---
        const canvasAsteroids = document.getElementById('gameCanvasAsteroids');
        const ctxAsteroids = canvasAsteroids ? canvasAsteroids.getContext('2d') : null;

        const timeDisplayAsteroids = document.getElementById('timeDisplayAsteroids');
        const livesDisplayAsteroids = document.getElementById('livesDisplayAsteroids');
        const keyDebugDisplay = document.getElementById('keyDebugDisplay'); // New debug element

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
        let keysAsteroids = {}; // Object to track pressed keys
        let confetti = [];
        let awaitingMenuTapAsteroids = false;
        let lastAsteroidSpawnTime = 0;

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

        // Keyboard input handling for Asteroids game
        document.addEventListener('keydown', (e) => {
            if (gameRunningAsteroids) {
                keysAsteroids[e.key] = true;
                updateKeyDebugDisplay(); // Update debug display
            }
        });

        document.addEventListener('keyup', (e) => {
            if (gameRunningAsteroids) {
                keysAsteroids[e.key] = false;
                updateKeyDebugDisplay(); // Update debug display
            }
        });

        function updateKeyDebugDisplay() {
            if (keyDebugDisplay) {
                const activeKeys = Object.keys(keysAsteroids).filter(key => keysAsteroids[key]);
                keyDebugDisplay.textContent = `Teclas: ${activeKeys.join(', ')}`;
            }
        }

        // Function to resize the canvas dynamically
        function resizeCanvasAsteroids() {
            if (!canvasAsteroids) {
                console.warn("Canvas for Asteroids game not found for resizing.");
                return;
            }
            const containerWidth = canvasAsteroids.parentElement.offsetWidth;
            // Maintain aspect ratio (e.g., 640x320, so width is 2x height)
            canvasAsteroids.width = containerWidth;
            canvasAsteroids.height = containerWidth / 2;

            // Re-position player after resize
            if (playerAsteroids) {
                playerAsteroids.x = canvasAsteroids.width / 2 - PLAYER_SIZE_ASTEROIDS / 2;
                playerAsteroids.y = canvasAsteroids.height - PLAYER_SIZE_ASTEROIDS * 2;
            }
        }

        // Add event listener for window resize
        window.addEventListener('resize', resizeCanvasAsteroids);


        function setupAudioAsteroids() {
            // Only initialize if not already initialized
            if (!backgroundSynthAsteroids) {
                backgroundSynthAsteroids = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sawtooth" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.3, release: 0.5 },
                    filter: { type: "lowpass", frequency: 1000, rolloff: -12 }
                }).toDestination();
                backgroundSynthAsteroids.volume.value = isMasterMuted ? -100 : -15;

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
            if (Tone.context.state === 'running' && backgroundSequenceAsteroids && typeof backgroundSequenceAsteroids.start === 'function') {
                backgroundSequenceAsteroids.start(Tone.Transport.now());
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
            if (timeDisplayAsteroids) timeDisplayAsteroids.textContent = `${Math.max(0, 60 - Math.floor(gameTimeAsteroids))}s`;
            if (livesDisplayAsteroids) livesDisplayAsteroids.textContent = playerLivesAsteroids;
        }

        function endGameAsteroids(won, cheatWin = false) {
            gameRunningAsteroids = false;
            cancelAnimationFrame(animationFrameIdAsteroids);
            stopBackgroundMusicAsteroids();

            if (gameUIAsteroids) gameUIAsteroids.style.display = 'none';

            let title, message;
            if (won) {
                title = "¡VICTORIA!";
                message = "¡Has sobrevivido a la lluvia de asteroides!";
                if (canvasAsteroids) {
                    for (let i = 0; i < 100; i++) {
                        confetti.push(new ConfettiParticle(Math.random() * canvasAsteroids.width, Math.random() * canvasAsteroids.height / 2));
                    }
                }
            } else {
                title = "¡PERDISTE!";
                message = "Los asteroides te han alcanzado.";
            }
            showModal(title, message, "Volver al Menú Principal", showMainMenu);
            awaitingMenuTapAsteroids = true;
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
                if (!ctxAsteroids) return; // Ensure context exists
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
                if (!canvasAsteroids) return; // Ensure canvas exists

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
                if (!ctxAsteroids) return; // Ensure context exists
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
                if (!canvasAsteroids) return; // Ensure canvas exists
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
                if (!ctxAsteroids) return; // Ensure context exists
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
            if (startScreenAsteroids) startScreenAsteroids.classList.add('hidden'); else console.warn("startScreenAsteroids not found.");
            if (gameUIAsteroids) gameUIAsteroids.classList.remove('hidden'); else console.warn("gameUIAsteroids not found.");
            if (gameUIAsteroids) gameUIAsteroids.style.display = 'flex';
            resetGameAsteroids();
            gameRunningAsteroids = true;
            lastFrameTimeAsteroids = performance.now();
            
            stopMainMenuMusic();
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama();
            stopBackgroundMusicSpelling();
            stopBackgroundMusicQuickMath();

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
            keysAsteroids = {}; // Reset keys state
            updateKeyDebugDisplay(); // Update debug display

            if (startScreenTitleAsteroids) startScreenTitleAsteroids.textContent = "Esquiva los Asteroides";
            if (startScreenMessageAsteroids) startScreenMessageAsteroids.textContent = "Sobrevive 60 segundos a una lluvia de asteroides. Este minijuego te ayudará a mejorar tus reflejos y tu tiempo de reacción. 🚀✨";
            if (startScreenButtonAsteroids) startScreenButtonAsteroids.textContent = "¡Comenzar!";

            resizeCanvasAsteroids(); 
            
            if (canvasAsteroids) {
                playerAsteroids = new PlayerAsteroids(canvasAsteroids.width / 2 - PLAYER_SIZE_ASTEROIDS / 2, canvasAsteroids.height - PLAYER_SIZE_ASTEROIDS * 2, PLAYER_SIZE_ASTEROIDS, PLAYER_SIZE_ASTEROIDS, 5);
            } else {
                console.error("Canvas for Asteroids game not found, cannot initialize player.");
                return;
            }
            playerLivesAsteroids = 2;
            isInvincibleAsteroids = false;
            invincibilityEndTimeAsteroids = 0;

            currentAsteroidSpawnRate = BASE_ASTEROID_SPAWN_RATE;
            currentMinAsteroidSpeed = BASE_MIN_ASTEROID_SPEED;
            currentMaxAsteroidSpeed = BASE_MAX_ASTEROID_SPEED;
            currentMinAsteroidSize = BASE_MIN_ASTEROID_SIZE;
            currentMaxAsteroidSize = BASE_MAX_ASTEROID_SIZE;

            lastAsteroidSpawnTime = performance.now(); // Initialize here
            lastHologramSpawnTime = performance.now();

            stars = [];
            if (canvasAsteroids) {
                for (let i = 0; i < 50; i++) {
                    stars.push({
                        x: Math.random() * canvasAsteroids.width,
                        y: Math.random() * canvasAsteroids.height,
                        size: Math.random() * 1 + 0.5,
                        speed: Math.random() * 0.1 + 0.05
                    });
                }
                for (let i = 0; i < 50; i++) {
                    stars.push({
                        x: Math.random() * canvasAsteroids.width,
                        y: Math.random() * canvasAsteroids.height,
                        size: Math.random() * 2 + 1,
                        speed: Math.random() * 0.3 + 0.1
                    });
                }
            }

            currentWarningAsteroids = { message: "", displayUntil: 0 };

            updateUIAsteroids();
            drawAsteroids();
        }

        function gameLoopAsteroids(currentTime) {
            if (!gameRunningAsteroids && !awaitingMenuTapAsteroids) return;
            if (!ctxAsteroids || !canvasAsteroids) {
                console.error("Canvas context or element missing for Asteroids game loop.");
                return;
            }

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
            if (!canvasAsteroids) return; // Ensure canvas exists

            if (gameRunningAsteroids) {
                playerAsteroids.update();

                if (isInvincibleAsteroids && performance.now() > invincibilityEndTimeAsteroids) {
                    isInvincibleAsteroids = false;
                }

                const oldDifficultyPhase = getDifficultyPhaseAsteroids(Math.floor(gameTimeAsteroids - (deltaTime / 1000)));
                const newDifficultyPhase = getDifficultyPhaseAsteroids(Math.floor(gameTimeAsteroids));

                if (gameTimeAsteroids >= 30) {
                    currentAsteroidSpawnRate = 100;
                    currentMinAsteroidSpeed = 5;
                    currentMaxAsteroidSpeed = 10;
                    currentMinAsteroidSize = 25;
                    currentMaxAsteroidSize = 55;
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
                                if (gameTimeAsteroids <= 10) {
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
            if (!ctxAsteroids || !canvasAsteroids) return; // Ensure context and canvas exist

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
                    const shadowAlpha = 0.3 + (0.7 * shadowScale);
                    
                    ctxAsteroids.fillStyle = `rgba(0, 0, 0, ${shadowAlpha})`;
                    ctxAsteroids.beginPath();
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

        if (startScreenButtonAsteroids) {
            startScreenButtonAsteroids.addEventListener('click', startGameAsteroidsLogic);
        } else {
            console.warn("Element with ID 'startScreenButtonAsteroids' not found.");
        }
        if (backToMenuButtonAsteroids) {
            backToMenuButtonAsteroids.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonAsteroids' not found.");
        }


        // --- Lógica del Minijuego Memorama ---
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
        const GAME_DURATION_MEMORAMA = 60;
        const LOW_TIME_THRESHOLD_MEMORAMA = 10;

        let memoramaSynth;
        let memoramaSequence;
        let lowTimeSynthMemorama;
        let lowTimeSequenceMemorama;
        let isLowTimeMusicPlaying = false;

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

            if (!lowTimeSynthMemorama) {
                lowTimeSynthMemorama = new Tone.Synth({
                    oscillator: { type: "square" },
                    envelope: { attack: 0.01, decay: 0.05, sustain: 0.0, release: 0.1 }
                }).toDestination();
                lowTimeSynthMemorama.volume.value = isMasterMuted ? -100 : -10;

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
                }, lowTimeNotes.flat(), "8n");
                lowTimeSequenceMemorama.loop = true;
                lowTimeSequenceMemorama.loopEnd = "1n";
            }
        }

        function startBackgroundMusicMemorama() {
            if (Tone.context.state === 'running' && memoramaSequence && typeof memoramaSequence.start === 'function') {
                memoramaSequence.start(Tone.Transport.now());
                Tone.Transport.bpm.value = 70;
                memoramaSynth.volume.value = isMasterMuted ? -100 : -15;
            }
        }

        function stopBackgroundMusicMemorama() {
            if (memoramaSequence && typeof memoramaSequence.stop === 'function') {
                memoramaSequence.stop();
            }
        }

        function startLowTimeMusicMemorama() {
            if (Tone.context.state === 'running' && lowTimeSequenceMemorama && typeof lowTimeSequenceMemorama.start === 'function') {
                lowTimeSequenceMemorama.start(Tone.Transport.now());
                Tone.Transport.bpm.value = 180;
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
            if (startScreenMemorama) startScreenMemorama.classList.add('hidden'); else console.warn("startScreenMemorama not found.");
            if (gameUIMemorama) gameUIMemorama.classList.remove('hidden'); else console.warn("gameUIMemorama not found.");
            if (gameUIMemorama) gameUIMemorama.style.display = 'flex';
            resetGameMemorama();
            gameRunningMemorama = true;

            stopMainMenuMusic();
            stopBackgroundMusicAsteroids();
            stopBackgroundMusicSpelling();
            stopBackgroundMusicQuickMath();

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
            isLowTimeMusicPlaying = false;
            if (memoramaGameContainer) memoramaGameContainer.classList.remove('low-time-warning');
            if (gameBoardMemorama) gameBoardMemorama.innerHTML = ''; else console.warn("gameBoardMemorama not found.");

            if (startScreenTitleMemorama) startScreenTitleMemorama.textContent = "Memorama";
            if (startScreenMessageMemorama) startScreenMessageMemorama.textContent = `Encuentra todas las parejas de emojis en ${GAME_DURATION_MEMORAMA} segundos. ¡Entrena tu memoria!`;
            if (startScreenButtonMemorama) startScreenButtonMemorama.textContent = "¡Comenzar!";

            const shuffledEmojis = [...EMOJIS, ...EMOJIS].sort(() => Math.random() - 0.5);
            if (totalPairsDisplayMemorama) totalPairsDisplayMemorama.textContent = EMOJIS.length;

            if (gameBoardMemorama) {
                shuffledEmojis.forEach((emoji, index) => {
                    const cardElement = document.createElement('div');
                    cardElement.classList.add('card');
                    cardElement.dataset.emoji = emoji;
                    cardElement.dataset.index = index;
                    cardElement.textContent = '?';

                    cardElement.addEventListener('click', () => flipCard(cardElement, index));
                    gameBoardMemorama.appendChild(cardElement);
                    memoramaCards.push({ element: cardElement, emoji: emoji, isFlipped: false, isMatched: false });
                });
            } else {
                console.error("gameBoardMemorama not found, cannot create cards.");
            }

            updateUIMemorama();
        }

        function flipCard(cardElement, index) {
            if (!canFlip || memoramaCards[index].isFlipped || memoramaCards[index].isMatched) {
                return;
            }

            cardElement.classList.add('flipped');
            cardElement.textContent = cardElement.dataset.emoji;
            memoramaCards[index].isFlipped = true;
            flippedCards.push(memoramaCards[index]);

            if (flippedCards.length === 2) {
                canFlip = false;
                setTimeout(() => {
                    const [card1, card2] = flippedCards;
                    if (card1.emoji === card2.emoji) {
                        card1.element.classList.add('matched');
                        card2.element.classList.add('matched');
                        card1.isMatched = true;
                        card2.isMatched = true;

                        card1.element.textContent = '✔';
                        card1.element.classList.add('card-matched-symbol');
                        card2.element.textContent = '✔';
                        card2.element.classList.add('card-matched-symbol');

                        if (memoramaSynth) memoramaSynth.triggerAttackRelease("C5", "8n");
                        matchedPairs++;
                        updateUIMemorama();
                        checkWinConditionMemorama();
                    } else {
                        card1.element.classList.remove('flipped');
                        card2.element.classList.remove('flipped');
                        card1.element.textContent = '?';
                        card2.element.textContent = '?';
                        card1.isFlipped = false;
                        card2.isFlipped = false;
                        if (memoramaSynth) memoramaSynth.triggerAttackRelease("C3", "8n");
                    }
                    flippedCards = [];
                    canFlip = true;
                }, 1000);
            }
        }

        function updateTimerMemorama() {
            timeElapsedMemorama++;
            const timeLeft = GAME_DURATION_MEMORAMA - timeElapsedMemorama;
            if (timeDisplayMemorama) timeDisplayMemorama.textContent = `${Math.max(0, timeLeft)}s`;

            if (timeLeft <= LOW_TIME_THRESHOLD_MEMORAMA && timeLeft > 0) {
                if (memoramaGameContainer) memoramaGameContainer.classList.add('low-time-warning');
                if (!isLowTimeMusicPlaying) {
                    stopBackgroundMusicMemorama();
                    startLowTimeMusicMemorama();
                    isLowTimeMusicPlaying = true;
                }
            } else {
                if (memoramaGameContainer) memoramaGameContainer.classList.remove('low-time-warning');
                if (isLowTimeMusicPlaying) {
                    stopLowTimeMusicMemorama();
                    startBackgroundMusicMemorama();
                    isLowTimeMusicPlaying = false;
                }
            }

            if (timeLeft <= 0 && matchedPairs < EMOJIS.length) {
                endGameMemorama(false);
            }
        }

        function updateUIMemorama() {
            const timeLeft = GAME_DURATION_MEMORAMA - timeElapsedMemorama;
            if (timeDisplayMemorama) timeDisplayMemorama.textContent = `${Math.max(0, timeLeft)}s`;
            if (matchedPairsDisplayMemorama) matchedPairsDisplayMemorama.textContent = matchedPairs;
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
            stopLowTimeMusicMemorama();
            if (memoramaGameContainer) memoramaGameContainer.classList.remove('low-time-warning');
            if (gameUIMemorama) gameUIMemorama.style.display = 'none';

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

        if (startScreenButtonMemorama) {
            startScreenButtonMemorama.addEventListener('click', startGameMemoramaLogic);
        } else {
            console.warn("Element with ID 'startScreenButtonMemorama' not found.");
        }
        if (backToMenuButtonMemorama) {
            backToMenuButtonMemorama.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonMemorama' not found.");
        }


        // --- Lógica del Minijuego Ortografía Correcta (Nuevo) ---
        const wordDisplay = document.getElementById('wordDisplay');
        const correctButton = document.getElementById('correctButton');
        const incorrectButton = document.getElementById('incorrectButton');
        const timeDisplaySpelling = document.getElementById('timeDisplaySpelling');
        const scoreDisplaySpelling = document.getElementById('scoreDisplaySpelling');
        const livesDisplaySpelling = document.getElementById('livesDisplaySpelling');
        const spellingGameMessage = document.getElementById('spellingGameMessage');

        const combinedLevelButtonSpelling = document.getElementById('combinedLevelButtonSpelling');
        const freeModeButtonSpelling = document.getElementById('freeModeButtonSpelling');

        const startScreenSpelling = document.getElementById('startScreenSpelling');
        const gameUISpelling = document.getElementById('gameUISpelling');
        const backToMenuButtonSpelling = document.getElementById('backToMenuButtonSpelling');
        const backToMenuButtonSpellingStartScreen = document.getElementById('backToMenuButtonSpellingStartScreen');
        const spellingLoadingIndicator = document.getElementById('spellingLoadingIndicator');
        const spellingLoadingMessage = document.getElementById('spellingLoadingMessage');
        const spellingResultsContainer = document.getElementById('spellingResultsContainer');
        const spellingResultsTableLocalBody = document.querySelector('#spellingResultsTableLocal tbody');
        const spellingResultsTableGlobalBody = document.querySelector('#spellingResultsTableGlobal tbody');

        let gameRunningSpelling = false;
        let scoreSpelling = 0;
        let playerLivesSpelling = 0;
        let timeElapsedSpelling = 0;
        let timerIntervalSpelling;
        let currentWordIndex = 0;
        let wordsForGame = [];
        let currentSpellingMode = '';
        let currentSpellingLevel = 1;

        const INITIAL_LIVES_COMBINED_LEVEL = 5;
        const GAME_DURATIONS_SPELLING = {
            'freeMode': 30
        };
        const TOTAL_WORDS_COMBINED_LEVEL = 30; 

        const DIFFICULTY_CONFIG_SPELLING = {
            'level1': { count: 5, prompt: "5 palabras simples" },
            'level2': { count: 10, prompt: "10 palabras de dificultad media" },
            'level3': { count: 15, prompt: "15 palabras complicadísimas (ej. 'impío', 'subrepticio', 'vicisitud', 'efímero', 'ubérrimo', 'concomitante')" },
            'freeMode': { count: 20, prompt: "palabras de dificultad variada" }
        };

        const MODE_NAMES = {
            'combinedLevel': 'Nivel Combinado',
            'freeMode': 'Modo Libre'
        };

        // Load local scores from localStorage
        let spellingScores = JSON.parse(localStorage.getItem('spellingScores')) || {
            'combinedLevel': { score: 0, total: 0, time: 0 },
            'freeMode': 0 // Local best score for free mode
        };

        let spellingCorrectSynth;
        let spellingIncorrectSynth;
        let backgroundSynthSpelling;
        let backgroundSequenceSpelling;

        function setupAudioSpelling() {
            if (!spellingCorrectSynth) {
                spellingCorrectSynth = new Tone.Synth({
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.0, release: 0.1 }
                }).toDestination();
                spellingCorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            }
            if (!spellingIncorrectSynth) {
                spellingIncorrectSynth = new Tone.Synth({
                    oscillator: { type: "sawtooth" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.0, release: 0.1 }
                }).toDestination();
                spellingIncorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            }
            if (!backgroundSynthSpelling) {
                backgroundSynthSpelling = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "square" },
                    envelope: { attack: 0.05, decay: 0.2, sustain: 0.5, release: 0.8 },
                    filter: { type: "lowpass", frequency: 800, rolloff: -12 }
                }).toDestination();
                backgroundSynthSpelling.volume.value = isMasterMuted ? -100 : -20;

                const calmNotes = [
                    ["C3", "4n"], ["E3", "4n"], ["G3", "4n"], ["A3", "4n"],
                    ["F3", "4n"], ["A3", "4n"], ["C4", "4n"], ["E4", "4n"]
                ];

                backgroundSequenceSpelling = new Tone.Sequence((time, note) => {
                    try {
                        backgroundSynthSpelling.triggerAttackRelease(note, "8n", time);
                    } catch (e) {
                        console.error("Error processing note in spelling background sequence:", note, e);
                    }
                }, calmNotes.flat(), "2n");

                Tone.Transport.bpm.value = 50;
                Tone.Transport.loop = true;
            }
        }

        function startBackgroundMusicSpelling() {
            if (Tone.context.state === 'running' && backgroundSequenceSpelling && typeof backgroundSequenceSpelling.start === 'function') {
                backgroundSequenceSpelling.start(Tone.Transport.now());
                backgroundSynthSpelling.volume.value = isMasterMuted ? -100 : -20;
                Tone.Transport.bpm.value = 50;
            }
        }

        function stopBackgroundMusicSpelling() {
            if (backgroundSequenceSpelling && typeof backgroundSequenceSpelling.stop === 'function') {
                backgroundSequenceSpelling.stop();
            }
        }

        function showSpellingGameMessage(message) {
            if (!spellingGameMessage) {
                console.warn("Spelling game message element not found.");
                return;
            }
            spellingGameMessage.textContent = message;
            spellingGameMessage.classList.remove('hidden');
            spellingGameMessage.style.animation = 'none';
            void spellingGameMessage.offsetWidth;
            spellingGameMessage.style.animation = 'fadeOut 2s forwards';
            setTimeout(() => {
                spellingGameMessage.classList.add('hidden');
            }, 2000);
        }

        async function fetchWordsFromGemini(mode) {
            if (spellingLoadingIndicator) spellingLoadingIndicator.classList.remove('hidden');
            if (spellingLoadingMessage) spellingLoadingMessage.classList.remove('hidden');
            if (combinedLevelButtonSpelling) combinedLevelButtonSpelling.disabled = true;
            if (freeModeButtonSpelling) freeModeButtonSpelling.disabled = true;
            if (backToMenuButtonSpellingStartScreen) backToMenuButtonSpellingStartScreen.disabled = true;

            if (wordDisplay) wordDisplay.textContent = 'Cargando palabras...';
            if (spellingResultsContainer) spellingResultsContainer.classList.add('hidden');
            wordsForGame = [];

            try {
                if (mode === 'combinedLevel') {
                    let promptL1 = `Genera una lista de ${DIFFICULTY_CONFIG_SPELLING.level1.count} palabras simples en español. La mitad de las palabras deben estar correctamente escritas y la otra mitad deben contener un error ortográfico sutil. Formato: [{\"word\": \"arbol\", \"correct\": false}, {\"word\": \"casa\", \"correct\": true}]`;
                    let wordsL1 = await callGeminiAPI(promptL1);
                    wordsForGame = wordsForGame.concat(wordsL1);

                    let promptL2 = `Genera una lista de ${DIFFICULTY_CONFIG_SPELLING.level2.count} palabras de dificultad media en español. La mitad de las palabras deben estar correctamente escritas y la otra mitad deben contener un error ortográfico sutil. Formato: [{\"word\": \"arbol\", \"correct\": false}, {\"word\": \"casa\", \"correct\": true}]`;
                    let wordsL2 = await callGeminiAPI(promptL2);
                    wordsForGame = wordsForGame.concat(wordsL2);

                    let promptL3 = `Genera una lista de ${DIFFICULTY_CONFIG_SPELLING.level3.count} palabras complicadísimas (ej. 'impío', 'subrepticio', 'vicisitud', 'efímero', 'ubérrimo', 'concomitante') en español. La mitad de las palabras deben estar correctamente escritas y la otra mitad deben contener un error ortográfico sutil. Formato: [{\"word\": \"arbol\", \"correct\": false}, {\"word\": \"casa\", \"correct\": true}]`;
                    let wordsL3 = await callGeminiAPI(promptL3);
                    wordsForGame = wordsForGame.concat(wordsL3);

                } else if (mode === 'freeMode') {
                    let promptFree = `Genera una lista de ${DIFFICULTY_CONFIG_SPELLING.freeMode.count} ${DIFFICULTY_CONFIG_SPELLING.freeMode.prompt} en español. La mitad de las palabras deben estar correctamente escritas y la otra mitad deben contener un error ortográfico sutil. Formato: [{\"word\": \"arbol\", \"correct\": false}, {\"word\": \"casa\", \"correct\": true}]`;
                    let wordsFree = await callGeminiAPI(promptFree);
                    wordsForGame = wordsFree.sort(() => Math.random() - 0.5);
                }
                
                if (combinedLevelButtonSpelling) combinedLevelButtonSpelling.disabled = false;
                if (freeModeButtonSpelling) freeModeButtonSpelling.disabled = false;
                if (backToMenuButtonSpellingStartScreen) backToMenuButtonSpellingStartScreen.disabled = false;

                startGameSpellingLogic();

            } catch (error) {
                console.error("Error fetching words from Gemini API:", error);
                if (wordDisplay) wordDisplay.textContent = 'Error al cargar palabras. Intenta de nuevo.';
                showModal('Error de Conexión', 'No se pudieron cargar las palabras debido a un error de red. Verifica tu conexión e intenta de nuevo.', 'Cerrar', showMainMenu);
            } finally {
                if (spellingLoadingIndicator) spellingLoadingIndicator.classList.add('hidden');
                if (spellingLoadingMessage) spellingLoadingMessage.classList.add('hidden');
            }
        }

        async function callGeminiAPI(prompt) {
            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });
            const payload = {
                contents: chatHistory,
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "ARRAY",
                        items: {
                            type: "OBJECT",
                            properties: {
                                "word": { "type": "STRING" },
                                "correct": { "type": "BOOLEAN" }
                            },
                            "propertyOrdering": ["word", "correct"]
                        }
                    }
                }
            };

            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const json = result.candidates[0].content.parts[0].text;
                return JSON.parse(json);
            } else {
                throw new Error("Unexpected API response structure.");
            }
        }


        function startGameSpellingLogic() {
            if (gameRunningSpelling) return;
            if (startScreenSpelling) startScreenSpelling.classList.add('hidden');
            if (gameUISpelling) gameUISpelling.classList.remove('hidden');
            if (gameUISpelling) gameUISpelling.style.display = 'flex';
            resetGameSpelling();
            gameRunningSpelling = true;

            stopMainMenuMusic();
            stopBackgroundMusicAsteroids();
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama();
            stopBackgroundMusicQuickMath();

            startBackgroundMusicSpelling();
            
            if (currentSpellingMode === 'freeMode') {
                timeElapsedSpelling = GAME_DURATIONS_SPELLING['freeMode'];
                timerIntervalSpelling = setInterval(updateTimerSpelling, 1000);
                if (livesDisplaySpelling && livesDisplaySpelling.parentElement) livesDisplaySpelling.parentElement.classList.add('hidden');
                if (timeDisplaySpelling && timeDisplaySpelling.parentElement) timeDisplaySpelling.parentElement.classList.remove('hidden');
            } else if (currentSpellingMode === 'combinedLevel') {
                playerLivesSpelling = INITIAL_LIVES_COMBINED_LEVEL;
                timeElapsedSpelling = 0;
                timerIntervalSpelling = setInterval(updateTimerSpelling, 1000);
                if (livesDisplaySpelling && livesDisplaySpelling.parentElement) livesDisplaySpelling.parentElement.classList.remove('hidden');
                if (timeDisplaySpelling && timeDisplaySpelling.parentElement) timeDisplaySpelling.parentElement.classList.remove('hidden');
                currentSpellingLevel = 1;
            }
            displayWord();
        }

        function resetGameSpelling() {
            clearInterval(timerIntervalSpelling);
            scoreSpelling = 0;
            timeElapsedSpelling = 0;
            currentWordIndex = 0;
            updateUISpelling();
            if (correctButton) correctButton.disabled = false;
            if (incorrectButton) incorrectButton.disabled = false;
        }

        function displayWord() {
            if (currentSpellingMode === 'combinedLevel' && (playerLivesSpelling <= 0 || currentWordIndex >= wordsForGame.length)) {
                endGameSpelling();
                return;
            }
            if (currentSpellingMode === 'freeMode' && timeElapsedSpelling <= 0) {
                endGameSpelling();
                return;
            }

            if (wordsForGame.length > 0 && currentWordIndex < wordsForGame.length) {
                if (wordDisplay) wordDisplay.textContent = wordsForGame[currentWordIndex].word;

                if (currentSpellingMode === 'combinedLevel') {
                    if (currentWordIndex === DIFFICULTY_CONFIG_SPELLING.level1.count && currentSpellingLevel === 1) {
                        showSpellingGameMessage("¡Pasaste al Nivel Medio!");
                        currentSpellingLevel = 2;
                    } else if (currentWordIndex === (DIFFICULTY_CONFIG_SPELLING.level1.count + DIFFICULTY_CONFIG_SPELLING.level2.count) && currentSpellingLevel === 2) {
                        showSpellingGameMessage("¡Pasaste a Infierno!");
                        currentSpellingLevel = 3;
                    }
                }

            } else if (currentSpellingMode === 'freeMode' && timeElapsedSpelling > 0) {
                 fetchWordsFromGemini('freeMode');
                 if (wordDisplay) wordDisplay.textContent = 'Cargando más palabras...';
            } else {
                if (wordDisplay) wordDisplay.textContent = 'No hay palabras disponibles.';
                if (correctButton) correctButton.disabled = true;
                if (incorrectButton) incorrectButton.disabled = true;
            }
        }

        function checkAnswer(isCorrectGuess) {
            if (!gameRunningSpelling) return;
            if (currentWordIndex >= wordsForGame.length) { // Prevent error if wordsForGame is empty or index out of bounds
                console.warn("Attempted to check answer with no words or out of bounds index.");
                return;
            }

            const currentWord = wordsForGame[currentWordIndex];
            if ((currentWord.correct && isCorrectGuess) || (!currentWord.correct && !isCorrectGuess)) {
                scoreSpelling++;
                if (spellingCorrectSynth) spellingCorrectSynth.triggerAttackRelease("G4", "8n");
            } else {
                if (currentSpellingMode === 'combinedLevel') {
                    playerLivesSpelling--;
                    if (spellingIncorrectSynth) spellingIncorrectSynth.triggerAttackRelease("C#3", "8n");
                    if (playerLivesSpelling <= 0) {
                        endGameSpelling();
                        return;
                    }
                } else {
                    if (spellingIncorrectSynth) spellingIncorrectSynth.triggerAttackRelease("C#3", "8n");
                }
            }
            updateUISpelling();
            currentWordIndex++;
            displayWord();
        }

        function updateTimerSpelling() {
            if (currentSpellingMode === 'freeMode') {
                timeElapsedSpelling--;
                if (timeDisplaySpelling) timeDisplaySpelling.textContent = `${Math.max(0, timeElapsedSpelling)}s`;
                if (timeElapsedSpelling <= 0) {
                    endGameSpelling();
                }
            } else if (currentSpellingMode === 'combinedLevel') {
                timeElapsedSpelling++;
                if (timeDisplaySpelling) timeDisplaySpelling.textContent = `${timeElapsedSpelling}s`;
                if (currentWordIndex >= wordsForGame.length && playerLivesSpelling > 0) {
                    endGameSpelling();
                }
            }
        }

        function updateUISpelling() {
            if (scoreDisplaySpelling) scoreDisplaySpelling.textContent = scoreSpelling;
            if (currentSpellingMode === 'combinedLevel') {
                if (livesDisplaySpelling) livesDisplaySpelling.textContent = playerLivesSpelling;
            }
        }

        async function endGameSpelling() {
            gameRunningSpelling = false;
            clearInterval(timerIntervalSpelling);
            stopBackgroundMusicSpelling();
            if (gameUISpelling) gameUISpelling.style.display = 'none';

            let title, message;
            if (currentSpellingMode === 'freeMode') {
                title = "¡Tiempo Agotado!";
                message = `Tu puntuación final en Modo Libre es: ${scoreSpelling} palabras correctas.`;
                if (scoreSpelling > spellingScores['freeMode']) {
                    spellingScores['freeMode'] = scoreSpelling;
                    localStorage.setItem('spellingScores', JSON.stringify(spellingScores));
                    message += " ¡Nuevo récord local!";
                    showLeaderboardSubmissionModal('spellingFreeMode', scoreSpelling, timeElapsedSpelling);
                    return; // Don't show generic modal, show submission modal
                }
            } else if (currentSpellingMode === 'combinedLevel') {
                const totalWordsPresented = currentWordIndex;
                if (playerLivesSpelling > 0) {
                    title = "¡Nivel Combinado Completado!";
                    message = `¡Has acertado ${scoreSpelling} de ${totalWordsPresented} palabras con ${playerLivesSpelling} vidas restantes en ${timeElapsedSpelling} segundos!`;
                    const currentCombinedScore = { score: scoreSpelling, total: totalWordsPresented, time: timeElapsedSpelling };
                    const previousBest = spellingScores['combinedLevel'];

                    if (currentCombinedScore.score > previousBest.score ||
                        (currentCombinedScore.score === previousBest.score && currentCombinedScore.total < previousBest.total) ||
                        (currentCombinedScore.score === previousBest.score && currentCombinedScore.total === previousBest.total && currentCombinedScore.time < previousBest.time) ||
                        (previousBest.score === 0 && previousBest.total === 0)) {
                        spellingScores['combinedLevel'] = currentCombinedScore;
                        localStorage.setItem('spellingScores', JSON.stringify(spellingScores));
                        message += " ¡Nuevo récord!";
                    }
                } else {
                    title = "¡Juego Terminado!";
                    message = `Te quedaste sin vidas. Acertaste ${scoreSpelling} de ${totalWordsPresented} palabras. ¡Sigue practicando!`;
                }
            }
            showModal(title, message, "Volver al Menú Principal", showMainMenu);
        }

        async function updateSpellingResultsTable() {
            if (spellingResultsTableLocalBody) spellingResultsTableLocalBody.innerHTML = '';
            if (spellingResultsTableGlobalBody) spellingResultsTableGlobalBody.innerHTML = '';

            // Local Scores
            if (spellingResultsTableLocalBody) {
                const localRow = spellingResultsTableLocalBody.insertRow();
                const localModeCell = localRow.insertCell();
                const localScoreCell = localRow.insertCell();
                localModeCell.textContent = MODE_NAMES['combinedLevel'];
                const combinedScore = spellingScores['combinedLevel'];
                if (combinedScore.score === 0 && combinedScore.total === 0) {
                    localScoreCell.textContent = 'N/A';
                } else {
                    localScoreCell.textContent = `${combinedScore.score}/${combinedScore.total} en ${combinedScore.time}s`;
                }

                const localFreeModeRow = spellingResultsTableLocalBody.insertRow();
                const localFreeModeCell = localFreeModeRow.insertCell();
                const localFreeModeScoreCell = localFreeModeRow.insertCell();
                localFreeModeCell.textContent = MODE_NAMES['freeMode'];
                localFreeModeScoreCell.textContent = spellingScores['freeMode'] === 0 ? 'N/A' : `${spellingScores['freeMode']} correctas`;
            }


            // Global Leaderboard (Free Mode)
            if (db && spellingResultsTableGlobalBody) {
                try {
                    // Fetch top 10 by score, then sort by timestamp in JS if scores are tied
                    // NOTE: This query requires appropriate Firebase Security Rules to allow read access.
                    // Example rule for public data:
                    // match /artifacts/{appId}/public/data/{collectionId}/{document=**} {
                    //   allow read: if request.auth != null;
                    //   allow write: if request.auth != null;
                    // }
                    const q = query(collection(db, `artifacts/${appId}/public/data/spellingFreeModeLeaderboard`), orderBy("score", "desc"), limit(10));
                    const querySnapshot = await getDocs(q);
                    let globalScores = [];
                    querySnapshot.forEach((doc) => {
                        globalScores.push(doc.data());
                    });

                    // Sort by score (desc) then timestamp (asc) in JavaScript
                    globalScores.sort((a, b) => {
                        if (a.score !== b.score) {
                            return b.score - a.score;
                        }
                        return a.timestamp - b.timestamp;
                    });

                    if (globalScores.length === 0) {
                        const row = spellingResultsTableGlobalBody.insertRow();
                        const cell = row.insertCell();
                        cell.colSpan = 3;
                        cell.textContent = 'No hay puntuaciones globales aún.';
                        cell.classList.add('text-center', 'text-gray-400');
                    } else {
                        globalScores.forEach((data) => {
                            const row = spellingResultsTableGlobalBody.insertRow();
                            row.insertCell().textContent = data.name;
                            row.insertCell().textContent = `${data.score} correctas`;
                            row.insertCell().textContent = new Date(data.timestamp).toLocaleDateString();
                        });
                    }
                } catch (e) {
                    console.error("Error fetching global spelling leaderboard:", e);
                    const row = spellingResultsTableGlobalBody.insertRow();
                    const cell = row.insertCell();
                    cell.colSpan = 3;
                    cell.textContent = 'Error al cargar el ranking global.';
                    cell.classList.add('text-center', 'text-red-400');
                }
            } else if (spellingResultsTableGlobalBody) {
                const row = spellingResultsTableGlobalBody.insertRow();
                const cell = row.insertCell();
                cell.colSpan = 3;
                cell.textContent = 'Firebase no configurado para ranking global.';
                cell.classList.add('text-center', 'text-gray-400');
            }

            if (spellingResultsContainer) spellingResultsContainer.classList.remove('hidden');
        }

        if (combinedLevelButtonSpelling) {
            combinedLevelButtonSpelling.addEventListener('click', () => {
                currentSpellingMode = 'combinedLevel';
                fetchWordsFromGemini('combinedLevel');
            });
        } else {
            console.warn("Element with ID 'combinedLevelButtonSpelling' not found.");
        }
        if (freeModeButtonSpelling) {
            freeModeButtonSpelling.addEventListener('click', () => {
                currentSpellingMode = 'freeMode';
                fetchWordsFromGemini('freeMode');
            });
        } else {
            console.warn("Element with ID 'freeModeButtonSpelling' not found.");
        }

        if (correctButton) {
            correctButton.addEventListener('click', () => checkAnswer(true));
        } else {
            console.warn("Element with ID 'correctButton' not found.");
        }
        if (incorrectButton) {
            incorrectButton.addEventListener('click', () => checkAnswer(false));
        } else {
            console.warn("Element with ID 'incorrectButton' not found.");
        }
        if (backToMenuButtonSpelling) {
            backToMenuButtonSpelling.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonSpelling' not found.");
        }
        if (backToMenuButtonSpellingStartScreen) {
            backToMenuButtonSpellingStartScreen.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonSpellingStartScreen' not found.");
        }


        // --- Lógica del Minijuego Sumas Rápidas (Nuevo) ---
        const problemDisplay = document.getElementById('problemDisplay');
        const answerInput = document.getElementById('answerInput');
        const submitAnswerButton = document.getElementById('submitAnswerButton');
        const timeDisplayQuickMath = document.getElementById('timeDisplayQuickMath');
        const scoreDisplayQuickMath = document.getElementById('scoreDisplayQuickMath');
        const livesDisplayQuickMath = document.getElementById('livesDisplayQuickMath');
        const quickMathGameMessage = document.getElementById('quickMathGameMessage');

        const combinedLevelButtonQuickMath = document.getElementById('combinedLevelButtonQuickMath');
        const freeModeButtonQuickMath = document.getElementById('freeModeButtonQuickMath');

        const startScreenQuickMath = document.getElementById('startScreenQuickMath');
        const gameUIQuickMath = document.getElementById('gameUIQuickMath');
        const backToMenuButtonQuickMath = document.getElementById('backToMenuButtonQuickMath');
        const backToMenuButtonQuickMathStartScreen = document.getElementById('backToMenuButtonQuickMathStartScreen');
        const quickMathResultsContainer = document.getElementById('quickMathResultsContainer');
        const quickMathResultsTableLocalBody = document.querySelector('#quickMathResultsTableLocal tbody');
        const quickMathResultsTableGlobalBody = document.querySelector('#quickMathResultsTableGlobal tbody');

        let quickMathGameRunning = false;
        let quickMathScore = 0;
        let quickMathLives = 0;
        let quickMathTimeElapsed = 0;
        let quickMathTimerInterval;
        let currentQuickMathProblem = null;
        let currentQuickMathMode = '';
        let currentQuickMathLevel = 1;
        let quickMathProblemsAnswered = 0;

        const INITIAL_LIVES_QUICKMATH = 5;
        const GAME_DURATION_QUICKMATH_FREEMODE = 60;

        const DIFFICULTY_CONFIG_QUICKMATH = {
            'level1': { count: 5, min1: 1, max1: 9, min2: 1, max2: 9 },
            'level2': { count: 10, min1: 10, max1: 99, min2: 10, max2: 99 },
            'level3': { count: 15, min1: 100, max1: 999, min2: 100, max2: 999 }
        };
        const TOTAL_PROBLEMS_QUICKMATH_COMBINED = DIFFICULTY_CONFIG_QUICKMATH.level1.count + DIFFICULTY_CONFIG_QUICKMATH.level2.count + DIFFICULTY_CONFIG_QUICKMATH.level3.count;


        // Load local scores from localStorage
        let quickMathScores = JSON.parse(localStorage.getItem('quickMathScores')) || {
            'combinedLevel': { score: 0, total: 0, time: 0 },
            'freeMode': 0 // Local best score for free mode
        };

        let quickMathCorrectSynth;
        let quickMathIncorrectSynth;
        let quickMathBackgroundSynth;
        let quickMathBackgroundSequence; // Declared here

        function setupAudioQuickMath() {
            if (!quickMathCorrectSynth) {
                quickMathCorrectSynth = new Tone.Synth({
                    oscillator: { type: "square" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.0, release: 0.1 }
                }).toDestination();
                quickMathCorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            }
            if (!quickMathIncorrectSynth) {
                quickMathIncorrectSynth = new Tone.Synth({
                    oscillator: { type: "triangle" },
                    envelope: { attack: 0.01, decay: 0.1, sustain: 0.0, release: 0.1 }
                }).toDestination();
                quickMathIncorrectSynth.volume.value = isMasterMuted ? -100 : -15;
            }
            if (!quickMathBackgroundSynth) {
                quickMathBackgroundSynth = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.05, decay: 0.2, sustain: 0.5, release: 0.8 },
                    filter: { type: "lowpass", frequency: 600, rolloff: -12 }
                }).toDestination();
                quickMathBackgroundSynth.volume.value = isMasterMuted ? -100 : -20;

                const mathNotes = [
                    ["C4", "4n"], ["F4", "4n"], ["G4", "4n"], ["C5", "4n"],
                    ["D4", "4n"], ["G4", "4n"], ["A4", "4n"], ["D5", "4n"]
                ];

                quickMathBackgroundSequence = new Tone.Sequence((time, note) => {
                    try {
                        quickMathBackgroundSynth.triggerAttackRelease(note, "8n", time);
                    } catch (e) {
                        console.error("Error processing note in quick math background sequence:", note, e);
                    }
                }, mathNotes.flat(), "2n");

                Tone.Transport.bpm.value = 80;
                Tone.Transport.loop = true;
            }
        }

        function startBackgroundMusicQuickMath() {
            if (Tone.context.state === 'running' && quickMathBackgroundSequence && typeof quickMathBackgroundSequence.start === 'function') {
                quickMathBackgroundSequence.start(Tone.Transport.now());
                quickMathBackgroundSynth.volume.value = isMasterMuted ? -100 : -20;
                Tone.Transport.bpm.value = 80;
            }
        }

        function stopBackgroundMusicQuickMath() {
            if (quickMathBackgroundSequence && typeof quickMathBackgroundSequence.stop === 'function') {
                quickMathBackgroundSequence.stop();
            }
        }

        function showQuickMathGameMessage(message) {
            if (!quickMathGameMessage) {
                console.warn("Quick Math game message element not found.");
                return;
            }
            quickMathGameMessage.textContent = message;
            quickMathGameMessage.classList.remove('hidden');
            quickMathGameMessage.style.animation = 'none';
            void quickMathGameMessage.offsetWidth;
            quickMathGameMessage.style.animation = 'fadeOut 2s forwards';
            setTimeout(() => {
                quickMathGameMessage.classList.add('hidden');
            }, 2000);
        }

        function startGameQuickMathLogic() {
            if (quickMathGameRunning) return;
            if (startScreenQuickMath) startScreenQuickMath.classList.add('hidden');
            if (gameUIQuickMath) gameUIQuickMath.classList.remove('hidden');
            if (gameUIQuickMath) gameUIQuickMath.style.display = 'flex';
            resetGameQuickMath();
            quickMathGameRunning = true;

            stopMainMenuMusic();
            stopBackgroundMusicAsteroids();
            stopBackgroundMusicMemorama();
            stopLowTimeMusicMemorama();
            stopBackgroundMusicSpelling();

            startBackgroundMusicQuickMath();

            if (currentQuickMathMode === 'freeMode') {
                quickMathTimeElapsed = GAME_DURATION_QUICKMATH_FREEMODE;
                quickMathTimerInterval = setInterval(updateTimerQuickMath, 1000);
                if (livesDisplayQuickMath && livesDisplayQuickMath.parentElement) livesDisplayQuickMath.parentElement.classList.add('hidden');
                if (timeDisplayQuickMath && timeDisplayQuickMath.parentElement) timeDisplayQuickMath.parentElement.classList.remove('hidden');
            } else if (currentQuickMathMode === 'combinedLevel') {
                quickMathLives = INITIAL_LIVES_QUICKMATH;
                quickMathTimeElapsed = 0;
                quickMathTimerInterval = setInterval(updateTimerQuickMath, 1000);
                if (livesDisplayQuickMath && livesDisplayQuickMath.parentElement) livesDisplayQuickMath.parentElement.classList.remove('hidden');
                if (timeDisplayQuickMath && timeDisplayQuickMath.parentElement) timeDisplayQuickMath.parentElement.classList.remove('hidden');
                currentQuickMathLevel = 1;
                quickMathProblemsAnswered = 0;
            }
            generateAndDisplayProblem();
            if (answerInput) answerInput.focus();
        }

        function resetGameQuickMath() {
            clearInterval(quickMathTimerInterval);
            quickMathScore = 0;
            quickMathTimeElapsed = 0;
            quickMathLives = INITIAL_LIVES_QUICKMATH;
            quickMathProblemsAnswered = 0;
            currentQuickMathProblem = null;
            updateUIQuickMath();
            if (answerInput) answerInput.value = '';
            if (submitAnswerButton) submitAnswerButton.disabled = false;
            if (answerInput) answerInput.disabled = false;
        }

        function generateProblem() {
            let num1, num2;
            let currentConfig;

            if (currentQuickMathMode === 'combinedLevel') {
                if (quickMathProblemsAnswered < DIFFICULTY_CONFIG_QUICKMATH.level1.count) {
                    currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level1;
                } else if (quickMathProblemsAnswered < (DIFFICULTY_CONFIG_QUICKMATH.level1.count + DIFFICULTY_CONFIG_QUICKMATH.level2.count)) {
                    currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level2;
                    if (currentQuickMathLevel === 1) {
                        showQuickMathGameMessage("¡Pasaste al Nivel Medio!");
                        currentQuickMathLevel = 2;
                    }
                } else {
                    currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level3;
                    if (currentQuickMathLevel === 2) {
                        showQuickMathGameMessage("¡Pasaste a Infierno!");
                        currentQuickMathLevel = 3;
                    }
                }
            } else {
                const rand = Math.random();
                if (rand < 0.4) currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level1;
                else if (rand < 0.8) currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level2;
                else currentConfig = DIFFICULTY_CONFIG_QUICKMATH.level3;
            }

            num1 = Math.floor(Math.random() * (currentConfig.max1 - currentConfig.min1 + 1)) + currentConfig.min1;
            num2 = Math.floor(Math.random() * (currentConfig.max2 - currentConfig.min2 + 1)) + currentConfig.min2;

            const problem = `${num1} + ${num2}`;
            const answer = num1 + num2;

            currentQuickMathProblem = { problem, answer };
        }

        function generateAndDisplayProblem() {
            generateProblem();
            if (problemDisplay) problemDisplay.textContent = `${currentQuickMathProblem.problem} = ?`;
            if (answerInput) answerInput.value = '';
            if (answerInput) answerInput.focus();
        }

        function checkAnswerQuickMath() {
            if (!quickMathGameRunning) return;
            if (!answerInput || !problemDisplay) {
                console.warn("Answer input or problem display not found.");
                return;
            }

            const userAnswer = parseInt(answerInput.value, 10);
            if (isNaN(userAnswer)) {
                return;
            }

            if (userAnswer === currentQuickMathProblem.answer) {
                quickMathScore++;
                if (quickMathCorrectSynth) quickMathCorrectSynth.triggerAttackRelease("G4", "8n");
            } else {
                if (currentQuickMathMode === 'combinedLevel') {
                    quickMathLives--;
                    if (quickMathIncorrectSynth) quickMathIncorrectSynth.triggerAttackRelease("C#3", "8n");
                    if (quickMathLives <= 0) {
                        endGameQuickMath();
                        return;
                    }
                } else {
                    if (quickMathIncorrectSynth) quickMathIncorrectSynth.triggerAttackRelease("C#3", "8n");
                }
            }
            quickMathProblemsAnswered++;
            updateUIQuickMath();
            generateAndDisplayProblem();
        }

        function updateTimerQuickMath() {
            if (currentQuickMathMode === 'freeMode') {
                quickMathTimeElapsed--;
                if (timeDisplayQuickMath) timeDisplayQuickMath.textContent = `${Math.max(0, quickMathTimeElapsed)}s`;
                if (quickMathTimeElapsed <= 0) {
                    endGameQuickMath();
                }
            } else if (currentQuickMathMode === 'combinedLevel') {
                quickMathTimeElapsed++;
                if (timeDisplayQuickMath) timeDisplayQuickMath.textContent = `${quickMathTimeElapsed}s`;
                if (quickMathProblemsAnswered >= TOTAL_PROBLEMS_QUICKMATH_COMBINED && quickMathLives > 0) {
                    endGameQuickMath();
                }
            }
        }

        function updateUIQuickMath() {
            if (scoreDisplayQuickMath) scoreDisplayQuickMath.textContent = quickMathScore;
            if (currentQuickMathMode === 'combinedLevel') {
                if (livesDisplayQuickMath) livesDisplayQuickMath.textContent = quickMathLives;
            }
        }

        async function endGameQuickMath() {
            quickMathGameRunning = false;
            clearInterval(quickMathTimerInterval);
            stopBackgroundMusicQuickMath();
            if (gameUIQuickMath) gameUIQuickMath.style.display = 'none';

            let title, message;
            if (currentQuickMathMode === 'freeMode') {
                title = "¡Tiempo Agotado!";
                message = `Tu puntuación final en Modo Libre es: ${quickMathScore} sumas correctas.`;
                if (quickMathScore > quickMathScores['freeMode']) {
                    quickMathScores['freeMode'] = quickMathScore;
                    localStorage.setItem('quickMathScores', JSON.stringify(quickMathScores));
                    message += " ¡Nuevo récord local!";
                    showLeaderboardSubmissionModal('quickMathFreeMode', quickMathScore, quickMathTimeElapsed);
                    return; // Don't show generic modal, show submission modal
                }
            } else if (currentQuickMathMode === 'combinedLevel') {
                const totalProblemsPresented = quickMathProblemsAnswered;
                if (quickMathLives > 0) {
                    title = "¡Nivel Combinado Completado!";
                    message = `¡Has acertado ${quickMathScore} de ${totalProblemsPresented} sumas con ${quickMathLives} vidas restantes en ${quickMathTimeElapsed} segundos!`;
                    const currentCombinedScore = { score: quickMathScore, total: totalProblemsPresented, time: quickMathTimeElapsed };
                    const previousBest = quickMathScores['combinedLevel'];

                    if (currentCombinedScore.score > previousBest.score ||
                        (currentCombinedScore.score === previousBest.score && currentCombinedScore.total < previousBest.total) ||
                        (currentCombinedScore.score === previousBest.score && currentCombinedScore.total === previousBest.total && currentCombinedScore.time < previousBest.time) ||
                        (previousBest.score === 0 && previousBest.total === 0)) {
                        quickMathScores['combinedLevel'] = currentCombinedScore;
                        localStorage.setItem('quickMathScores', JSON.stringify(quickMathScores));
                        message += " ¡Nuevo récord!";
                    }
                } else {
                    title = "¡Juego Terminado!";
                    message = `Te quedaste sin vidas. Acertaste ${quickMathScore} de ${totalProblemsPresented} sumas. ¡Sigue practicando!`;
                }
            }
            showModal(title, message, "Volver al Menú Principal", showMainMenu);
        }

        async function updateQuickMathResultsTable() {
            if (quickMathResultsTableLocalBody) quickMathResultsTableLocalBody.innerHTML = '';
            if (quickMathResultsTableGlobalBody) quickMathResultsTableGlobalBody.innerHTML = '';

            // Local Scores
            if (quickMathResultsTableLocalBody) {
                const localRow = quickMathResultsTableLocalBody.insertRow();
                const localModeCell = localRow.insertCell();
                const localScoreCell = localRow.insertCell();
                localModeCell.textContent = MODE_NAMES['combinedLevel'];
                const combinedScore = quickMathScores['combinedLevel'];
                if (combinedScore.score === 0 && combinedScore.total === 0) {
                    localScoreCell.textContent = 'N/A';
                } else {
                    localScoreCell.textContent = `${combinedScore.score}/${combinedScore.total} en ${combinedScore.time}s`;
                }

                const localFreeModeRow = quickMathResultsTableLocalBody.insertRow();
                const localFreeModeCell = localFreeModeRow.insertCell();
                const localFreeModeScoreCell = localFreeModeRow.insertCell();
                localFreeModeCell.textContent = MODE_NAMES['freeMode'];
                localFreeModeScoreCell.textContent = quickMathScores['freeMode'] === 0 ? 'N/A' : `${quickMathScores['freeMode']} correctas`;
            }

            // Global Leaderboard (Free Mode)
            if (db && quickMathResultsTableGlobalBody) {
                try {
                    // Fetch top 10 by score, then sort by timestamp in JS if scores are tied
                    // NOTE: This query requires appropriate Firebase Security Rules to allow read access.
                    // Example rule for public data:
                    // match /artifacts/{appId}/public/data/{collectionId}/{document=**} {
                    //   allow read: if request.auth != null;
                    //   allow write: if request.auth != null;
                    // }
                    const q = query(collection(db, `artifacts/${appId}/public/data/quickMathFreeModeLeaderboard`), orderBy("score", "desc"), limit(10));
                    const querySnapshot = await getDocs(q);
                    let globalScores = [];
                    querySnapshot.forEach((doc) => {
                        globalScores.push(doc.data());
                    });

                    // Sort by score (desc) then timestamp (asc) in JavaScript
                    globalScores.sort((a, b) => {
                        if (a.score !== b.score) {
                            return b.score - a.score;
                        }
                        return a.timestamp - b.timestamp;
                    });

                    if (globalScores.length === 0) {
                        const row = quickMathResultsTableGlobalBody.insertRow();
                        const cell = row.insertCell();
                        cell.colSpan = 3;
                        cell.textContent = 'No hay puntuaciones globales aún.';
                        cell.classList.add('text-center', 'text-gray-400');
                    } else {
                        globalScores.forEach((data) => {
                            const row = quickMathResultsTableGlobalBody.insertRow();
                            row.insertCell().textContent = data.name;
                            row.insertCell().textContent = `${data.score} correctas`;
                            row.insertCell().textContent = new Date(data.timestamp).toLocaleDateString();
                        });
                    }
                } catch (e) {
                    console.error("Error fetching global quick math leaderboard:", e);
                    const row = quickMathResultsTableGlobalBody.insertRow();
                    const cell = row.insertCell();
                    cell.colSpan = 3;
                    cell.textContent = 'Error al cargar el ranking global.';
                    cell.classList.add('text-center', 'text-red-400');
                }
            } else if (quickMathResultsTableGlobalBody) {
                const row = quickMathResultsTableGlobalBody.insertRow();
                const cell = row.insertCell();
                cell.colSpan = 3;
                cell.textContent = 'Firebase no configurado para ranking global.';
                cell.classList.add('text-center', 'text-gray-400');
            }

            if (quickMathResultsContainer) quickMathResultsContainer.classList.remove('hidden');
        }

        if (combinedLevelButtonQuickMath) {
            combinedLevelButtonQuickMath.addEventListener('click', () => {
                currentQuickMathMode = 'combinedLevel';
                startGameQuickMathLogic();
            });
        } else {
            console.warn("Element with ID 'combinedLevelButtonQuickMath' not found.");
        }
        if (freeModeButtonQuickMath) {
            freeModeButtonQuickMath.addEventListener('click', () => {
                currentQuickMathMode = 'freeMode';
                startGameQuickMathLogic();
            });
        } else {
            console.warn("Element with ID 'freeModeButtonQuickMath' not found.");
        }

        if (submitAnswerButton) {
            submitAnswerButton.addEventListener('click', checkAnswerQuickMath);
        } else {
            console.warn("Element with ID 'submitAnswerButton' not found.");
        }
        if (answerInput) {
            answerInput.addEventListener('keydown', (event) => {
                if (event.key === 'Enter') {
                    checkAnswerQuickMath();
                }
            });
        } else {
            console.warn("Element with ID 'answerInput' not found for keydown listener.");
        }
        if (backToMenuButtonQuickMath) {
            backToMenuButtonQuickMath.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonQuickMath' not found.");
        }
        if (backToMenuButtonQuickMathStartScreen) {
            backToMenuButtonQuickMathStartScreen.addEventListener('click', showMainMenu);
        } else {
            console.warn("Element with ID 'backToMenuButtonQuickMathStartScreen' not found.");
        }


        // --- Lógica de Inicio y Cambio de Juegos ---
        if (startGameAsteroidsButton) {
            startGameAsteroidsButton.addEventListener('click', () => {
                console.log("Asteroids button clicked!"); // Debug log
                hideMainMenu();
                hideAllGames();
                if (asteroidGameContainer) asteroidGameContainer.classList.remove('hidden');
                resetGameAsteroids();
                setupAudioAsteroids();
            });
        } else {
            console.warn("Element with ID 'startGameAsteroidsButton' not found.");
        }

        if (startGameMemoramaButton) {
            startGameMemoramaButton.addEventListener('click', () => {
                console.log("Memorama button clicked!"); // Debug log
                hideMainMenu();
                hideAllGames();
                if (memoramaGameContainer) memoramaGameContainer.classList.remove('hidden');
                resetGameMemorama();
                setupAudioMemorama();
            });
        } else {
            console.warn("Element with ID 'startGameMemoramaButton' not found.");
        }

        if (startGameSpellingButton) {
            startGameSpellingButton.addEventListener('click', async () => {
                console.log("Spelling button clicked!"); // Debug log
                hideMainMenu();
                hideAllGames();
                if (spellingGameContainer) spellingGameContainer.classList.remove('hidden');
                setupAudioSpelling();
                updateSpellingResultsTable();
            });
        } else {
            console.warn("Element with ID 'startGameSpellingButton' not found.");
        }

        if (startGameQuickMathButton) {
            startGameQuickMathButton.addEventListener('click', async () => {
                console.log("Quick Math button clicked!"); // Debug log
                hideMainMenu();
                hideAllGames();
                if (quickMathGameContainer) quickMathGameContainer.classList.remove('hidden');
                setupAudioQuickMath();
                updateQuickMathResultsTable();
            });
        } else {
            console.warn("Element with ID 'startGameQuickMathButton' not found.");
        }


        // Configuración inicial al cargar la página principal
        document.addEventListener('DOMContentLoaded', () => {
            console.log("DOM Content Loaded. Initializing app...");

            setupMainMenuAudio();
            setupAudioAsteroids();
            setupAudioMemorama();
            setupAudioSpelling();
            setupAudioQuickMath();

            document.body.addEventListener('click', function firstInteractionHandler() {
                if (Tone.context.state !== 'running') {
                    Tone.start().then(() => {
                        console.log("Tone.js audio context started.");
                        Tone.Transport.start();
                        console.log("Tone.js Transport started.");
                        startMainMenuMusic();
                    }).catch(e => console.error("Error starting Tone.js audio context:", e));
                }
                document.body.removeEventListener('click', firstInteractionHandler);
            }, { once: true });

            showMainMenu();
            hideAllGames(); // Ensure all games are hidden initially
        });
    </script>
</body>
</html>
