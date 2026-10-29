<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub DJ Deck Game</title>
    <style>
        body {
            background-color: #121212;
            color: #ffffff;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 { margin-bottom: 30px; color: #00ffcc; text-shadow: 0 0 10px #00ffcc; }
        .dj-console {
            background-color: #1e1e1e;
            border: 4px solid #333;
            border-radius: 15px;
            padding: 30px;
            display: flex;
            gap: 40px;
            align-items: center;
            box-shadow: 0 20px 40px rgba(0,0,0,0.5);
        }
        .deck {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }
        .vinyl {
            width: 150px;
            height: 150px;
            background: radial-gradient(circle, #333 40%, #111 41%, #000 100%);
            border-radius: 50%;
            border: 2px solid #555;
            position: relative;
            transition: transform 0.1s linear;
        }
        .vinyl.spinning {
            animation: spin 2s linear infinite;
        }
        .vinyl::after {
            content: '';
            position: absolute;
            top: 50%; left: 50%;
            width: 20px; height: 20px;
            background-color: #00ffcc;
            border-radius: 50%;
            transform: translate(-50%, -50%);
        }
        @keyframes spin {
            100% { transform: rotate(360deg); }
        }
        .controls { display: flex; gap: 10px; }
        button {
            background-color: #333;
            color: #fff;
            border: 1px solid #00ffcc;
            padding: 8px 15px;
            cursor: pointer;
            border-radius: 5px;
            font-weight: bold;
        }
        button:hover { background-color: #00ffcc; color: #000; }
        .mixer {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }
        input[type="range"] {
            width: 150px;
            accent-color: #00ffcc;
        }
    </style>
</head>
<body>

    <h1>🎧 GITHUB DJ MIXER 🎧</h1>

    <div class="dj-console">
        <!-- DECK A -->
        <div class="deck" id="deck-a">
            <h3>DECK A</h3>
            <div class="vinyl" id="vinyl-a"></div>
            <input type="file" id="audio-file-a" accept="audio/*">
            <div class="controls">
                <button onclick="togglePlay('a')">PLAY/PAUSE</button>
            </div>
        </div>

        <!-- MIXER / CROSSFADER -->
        <div class="mixer">
            <label for="crossfader">CROSSFADER</label>
            <input type="type" type="range" id="crossfader" min="-1" max="1" step="0.01" value="0">
            <span id="fade-status">Center</span>
        </div>

        <!-- DECK B -->
        <div class="deck" id="deck-b">
            <h3>DECK B</h3>
            <div class="vinyl" id="vinyl-b"></div>
            <input type="file" id="audio-file-b" accept="audio/*">
            <div class="controls">
                <button onclick="togglePlay('b')">PLAY/PAUSE</button>
            </div>
        </div>
    </div>

    <script>
        // Web Audio Context setup
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        const audioCtx = new AudioContext();

        // State trackers
        const state = {
            a: { audio: null, source: null, gainNode: null, playing: false },
            b: { audio: null, source: null, gainNode: null, playing: false }
        };

        // Initialize Audio Nodes for Decks
        function setupAudioElement(deckId, fileElement) {
            const file = fileElement.files[0];
            if (!file) return;

            if (state[deckId].audio) {
                state[deckId].audio.pause();
            }

            const url = URL.createObjectURL(file);
            const audio = new Audio(url);
            audio.loop = true;

            const source = audioCtx.createMediaElementSource(audio);
            const gainNode = audioCtx.createGain();

            source.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            state[deckId].audio = audio;
            state[deckId].source = source;
            state[deckId].gainNode = gainNode;
            
            // Trigger crossfader recalculation to apply correct volumes
            updateVolumes();
        }

        document.getElementById('audio-file-a').addEventListener('change', (e) => setupAudioElement('a', e.target));
        document.getElementById('audio-file-b').addEventListener('change', (e) => setupAudioElement('b', e.target));

        function togglePlay(deckId) {
            // Resume context if suspended (browser security feature)
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }

            const deck = state[deckId];
            const vinyl = document.getElementById(`vinyl-${deckId}`);

            if (!deck.audio) {
                alert(`Please upload an audio file to Deck ${deckId.toUpperCase()} first!`);
                return;
            }

            if (deck.playing) {
                deck.audio.pause();
                deck.playing = false;
                vinyl.classList.remove('spinning');
            } else {
                deck.audio.play();
                deck.playing = true;
                vinyl.classList.add('spinning');
            }
        }

        // Crossfader Logic
        const crossfader = document.getElementById('crossfader');
        crossfader.addEventListener('input', updateVolumes);

        function updateVolumes() {
            const val = parseFloat(crossfader.value); // -1 (All Left) to 1 (All Right)
            
            // Equal power crossfade calculation
            const gainA = Math.cos((val + 1) * Math.PI / 4);
            const gainB = Math.sin((val + 1) * Math.PI / 4);

            if (state.a.gainNode) state.a.gainNode.gain.value = gainA;
            if (state.b.gainNode) state.b.gainNode.gain.value = gainB;

            // Status indicator text
            const status = document.getElementById('fade-status');
            if (val < -0.1) status.innerText = "◄ Deck A";
            else if (val > 0.1) status.innerText = "Deck B ►";
            else status.innerText = "Center";
        }
    </script>
</body>
</html>

