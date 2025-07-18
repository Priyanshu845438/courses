
# 🚀 Advanced HTML

## Table of Contents
- [HTML5 APIs](#html5-apis)
- [Advanced Forms](#advanced-forms)
- [Multimedia Integration](#multimedia-integration)
- [Performance Optimization](#performance-optimization)
- [Progressive Web Apps](#progressive-web-apps)
- [Accessibility Best Practices](#accessibility-best-practices)
- [SEO Optimization](#seo-optimization)
- [Modern HTML Features](#modern-html-features)

---

## 🔧 HTML5 APIs

### Canvas API

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Canvas API Demo</title>
    <style>
        canvas {
            border: 1px solid #ccc;
            margin: 20px;
        }
        .controls {
            margin: 20px;
        }
        button {
            margin: 5px;
            padding: 10px 15px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Canvas Drawing App</h1>
    
    <div class="controls">
        <button onclick="drawShapes()">Draw Shapes</button>
        <button onclick="drawText()">Draw Text</button>
        <button onclick="drawImage()">Draw Image</button>
        <button onclick="clearCanvas()">Clear Canvas</button>
        <button onclick="saveCanvas()">Save Canvas</button>
    </div>
    
    <canvas id="myCanvas" width="800" height="600"></canvas>
    
    <script>
        const canvas = document.getElementById('myCanvas');
        const ctx = canvas.getContext('2d');
        
        // Draw shapes
        function drawShapes() {
            // Rectangle
            ctx.fillStyle = '#ff6b6b';
            ctx.fillRect(50, 50, 100, 80);
            
            // Circle
            ctx.beginPath();
            ctx.arc(250, 90, 40, 0, 2 * Math.PI);
            ctx.fillStyle = '#4ecdc4';
            ctx.fill();
            
            // Triangle
            ctx.beginPath();
            ctx.moveTo(400, 50);
            ctx.lineTo(350, 130);
            ctx.lineTo(450, 90);
            ctx.fillStyle = '#ffe66d';
            ctx.fill();
            
            // Line
            ctx.beginPath();
            ctx.moveTo(500, 50);
            ctx.lineTo(600, 150);
            ctx.strokeStyle = '#a8e6cf';
            ctx.lineWidth = 5;
            ctx.stroke();
        }
        
        // Draw text
        function drawText() {
            ctx.font = '30px Arial';
            ctx.fillStyle = '#333';
            ctx.textAlign = 'center';
            ctx.fillText('Canvas Drawing Demo', 400, 200);
            
            ctx.font = '16px Arial';
            ctx.fillStyle = '#666';
            ctx.fillText('Interactive graphics with HTML5 Canvas', 400, 230);
        }
        
        // Draw image
        function drawImage() {
            const img = new Image();
            img.onload = function() {
                ctx.drawImage(img, 300, 250, 200, 150);
            };
            img.src = 'data:image/svg+xml;base64,' + btoa(`
                <svg xmlns="http://www.w3.org/2000/svg" width="200" height="150">
                    <rect width="200" height="150" fill="#e8f5e8"/>
                    <circle cx="100" cy="75" r="50" fill="#4CAF50"/>
                    <text x="100" y="80" text-anchor="middle" fill="white" font-size="16">Sample</text>
                </svg>
            `);
        }
        
        // Clear canvas
        function clearCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }
        
        // Save canvas as image
        function saveCanvas() {
            const link = document.createElement('a');
            link.download = 'canvas-drawing.png';
            link.href = canvas.toDataURL();
            link.click();
        }
        
        // Interactive drawing
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;
        
        canvas.addEventListener('mousedown', (e) => {
            isDrawing = true;
            [lastX, lastY] = [e.offsetX, e.offsetY];
        });
        
        canvas.addEventListener('mousemove', (e) => {
            if (!isDrawing) return;
            
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(e.offsetX, e.offsetY);
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 2;
            ctx.lineCap = 'round';
            ctx.stroke();
            
            [lastX, lastY] = [e.offsetX, e.offsetY];
        });
        
        canvas.addEventListener('mouseup', () => {
            isDrawing = false;
        });
        
        canvas.addEventListener('mouseout', () => {
            isDrawing = false;
        });
    </script>
</body>
</html>
```

---

## 🎵 Web Audio API

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web Audio API Demo</title>
    <style>
        .audio-controls {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            margin: 20px;
            padding: 20px;
            background: #f5f5f5;
            border-radius: 8px;
        }
        
        .control-group {
            display: flex;
            flex-direction: column;
            align-items: center;
            min-width: 150px;
        }
        
        .control-group label {
            margin-bottom: 8px;
            font-weight: bold;
        }
        
        .control-group input, .control-group button {
            padding: 8px 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .visualizer {
            width: 100%;
            height: 200px;
            background: #000;
            margin: 20px 0;
        }
        
        .recorder-controls {
            margin: 20px;
            padding: 20px;
            background: #e8f5e8;
            border-radius: 8px;
        }
        
        .audio-player {
            margin: 20px;
            padding: 20px;
            background: #fff3cd;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <h1>Advanced Web Audio API Demo</h1>
    
    <!-- Audio Synthesis Controls -->
    <div class="audio-controls">
        <div class="control-group">
            <label for="frequency">Frequency (Hz)</label>
            <input type="range" id="frequency" min="100" max="1000" value="440">
            <span id="freq-display">440 Hz</span>
        </div>
        
        <div class="control-group">
            <label for="waveform">Waveform</label>
            <select id="waveform">
                <option value="sine">Sine</option>
                <option value="square">Square</option>
                <option value="sawtooth">Sawtooth</option>
                <option value="triangle">Triangle</option>
            </select>
        </div>
        
        <div class="control-group">
            <label for="volume">Volume</label>
            <input type="range" id="volume" min="0" max="100" value="50">
            <span id="vol-display">50%</span>
        </div>
        
        <div class="control-group">
            <button id="play-tone">Play Tone</button>
            <button id="stop-tone">Stop Tone</button>
        </div>
    </div>
    
    <!-- Audio Visualizer -->
    <canvas id="visualizer" class="visualizer" width="800" height="200"></canvas>
    
    <!-- Audio Recorder -->
    <div class="recorder-controls">
        <h3>Audio Recorder</h3>
        <button id="start-record">Start Recording</button>
        <button id="stop-record" disabled>Stop Recording</button>
        <button id="play-recording" disabled>Play Recording</button>
        <div id="recording-status"></div>
    </div>
    
    <!-- Custom Audio Player -->
    <div class="audio-player">
        <h3>Custom Audio Player</h3>
        <input type="file" id="audio-file" accept="audio/*">
        <br><br>
        <div id="player-controls" style="display: none;">
            <button id="play-pause">Play</button>
            <input type="range" id="seek" min="0" max="100" value="0">
            <span id="time-display">0:00 / 0:00</span>
            <br><br>
            <label for="playback-rate">Playback Rate:</label>
            <input type="range" id="playback-rate" min="0.5" max="2" step="0.1" value="1">
            <span id="rate-display">1x</span>
        </div>
    </div>
    
    <script>
        class AudioController {
            constructor() {
                this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
                this.oscillator = null;
                this.gainNode = null;
                this.analyser = null;
                this.mediaRecorder = null;
                this.recordedChunks = [];
                this.currentAudioBuffer = null;
                this.audioSource = null;
                
                this.setupControls();
                this.setupVisualizer();
                this.setupRecorder();
                this.setupPlayer();
            }
            
            setupControls() {
                const frequencySlider = document.getElementById('frequency');
                const waveformSelect = document.getElementById('waveform');
                const volumeSlider = document.getElementById('volume');
                const playButton = document.getElementById('play-tone');
                const stopButton = document.getElementById('stop-tone');
                
                frequencySlider.addEventListener('input', (e) => {
                    document.getElementById('freq-display').textContent = e.target.value + ' Hz';
                    if (this.oscillator) {
                        this.oscillator.frequency.setValueAtTime(e.target.value, this.audioContext.currentTime);
                    }
                });
                
                volumeSlider.addEventListener('input', (e) => {
                    const volume = e.target.value / 100;
                    document.getElementById('vol-display').textContent = e.target.value + '%';
                    if (this.gainNode) {
                        this.gainNode.gain.setValueAtTime(volume, this.audioContext.currentTime);
                    }
                });
                
                waveformSelect.addEventListener('change', (e) => {
                    if (this.oscillator) {
                        this.oscillator.type = e.target.value;
                    }
                });
                
                playButton.addEventListener('click', () => this.playTone());
                stopButton.addEventListener('click', () => this.stopTone());
            }
            
            playTone() {
                if (this.oscillator) return;
                
                this.oscillator = this.audioContext.createOscillator();
                this.gainNode = this.audioContext.createGain();
                this.analyser = this.audioContext.createAnalyser();
                
                const frequency = document.getElementById('frequency').value;
                const waveform = document.getElementById('waveform').value;
                const volume = document.getElementById('volume').value / 100;
                
                this.oscillator.frequency.setValueAtTime(frequency, this.audioContext.currentTime);
                this.oscillator.type = waveform;
                this.gainNode.gain.setValueAtTime(volume, this.audioContext.currentTime);
                
                this.oscillator.connect(this.gainNode);
                this.gainNode.connect(this.analyser);
                this.analyser.connect(this.audioContext.destination);
                
                this.oscillator.start();
                this.startVisualization();
            }
            
            stopTone() {
                if (this.oscillator) {
                    this.oscillator.stop();
                    this.oscillator = null;
                    this.gainNode = null;
                }
            }
            
            setupVisualizer() {
                this.canvas = document.getElementById('visualizer');
                this.canvasCtx = this.canvas.getContext('2d');
                this.canvas.width = 800;
                this.canvas.height = 200;
            }
            
            startVisualization() {
                if (!this.analyser) return;
                
                this.analyser.fftSize = 256;
                const bufferLength = this.analyser.frequencyBinCount;
                const dataArray = new Uint8Array(bufferLength);
                
                const draw = () => {
                    if (!this.analyser) return;
                    
                    requestAnimationFrame(draw);
                    
                    this.analyser.getByteFrequencyData(dataArray);
                    
                    this.canvasCtx.fillStyle = 'rgb(0, 0, 0)';
                    this.canvasCtx.fillRect(0, 0, this.canvas.width, this.canvas.height);
                    
                    const barWidth = (this.canvas.width / bufferLength) * 2.5;
                    let barHeight;
                    let x = 0;
                    
                    for (let i = 0; i < bufferLength; i++) {
                        barHeight = dataArray[i] / 2;
                        
                        const r = barHeight + 25 * (i / bufferLength);
                        const g = 250 * (i / bufferLength);
                        const b = 50;
                        
                        this.canvasCtx.fillStyle = `rgb(${r},${g},${b})`;
                        this.canvasCtx.fillRect(x, this.canvas.height - barHeight, barWidth, barHeight);
                        
                        x += barWidth + 1;
                    }
                };
                
                draw();
            }
            
            async setupRecorder() {
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                    this.mediaRecorder = new MediaRecorder(stream);
                    
                    this.mediaRecorder.ondataavailable = (event) => {
                        if (event.data.size > 0) {
                            this.recordedChunks.push(event.data);
                        }
                    };
                    
                    this.mediaRecorder.onstop = () => {
                        const blob = new Blob(this.recordedChunks, { type: 'audio/wav' });
                        this.recordedChunks = [];
                        
                        const url = URL.createObjectURL(blob);
                        document.getElementById('play-recording').onclick = () => {
                            const audio = new Audio(url);
                            audio.play();
                        };
                        document.getElementById('play-recording').disabled = false;
                    };
                    
                    document.getElementById('start-record').addEventListener('click', () => {
                        this.recordedChunks = [];
                        this.mediaRecorder.start();
                        document.getElementById('start-record').disabled = true;
                        document.getElementById('stop-record').disabled = false;
                        document.getElementById('recording-status').textContent = 'Recording...';
                    });
                    
                    document.getElementById('stop-record').addEventListener('click', () => {
                        this.mediaRecorder.stop();
                        document.getElementById('start-record').disabled = false;
                        document.getElementById('stop-record').disabled = true;
                        document.getElementById('recording-status').textContent = 'Recording stopped';
                    });
                    
                } catch (error) {
                    console.error('Error accessing microphone:', error);
                    document.getElementById('recording-status').textContent = 'Microphone access denied';
                }
            }
            
            setupPlayer() {
                const fileInput = document.getElementById('audio-file');
                const playerControls = document.getElementById('player-controls');
                const playPauseBtn = document.getElementById('play-pause');
                const seekSlider = document.getElementById('seek');
                const timeDisplay = document.getElementById('time-display');
                const playbackRateSlider = document.getElementById('playback-rate');
                const rateDisplay = document.getElementById('rate-display');
                
                fileInput.addEventListener('change', async (e) => {
                    const file = e.target.files[0];
                    if (!file) return;
                    
                    const arrayBuffer = await file.arrayBuffer();
                    this.currentAudioBuffer = await this.audioContext.decodeAudioData(arrayBuffer);
                    playerControls.style.display = 'block';
                });
                
                playPauseBtn.addEventListener('click', () => {
                    if (this.audioSource) {
                        this.audioSource.stop();
                        this.audioSource = null;
                        playPauseBtn.textContent = 'Play';
                    } else {
                        this.playAudioBuffer();
                        playPauseBtn.textContent = 'Pause';
                    }
                });
                
                playbackRateSlider.addEventListener('input', (e) => {
                    const rate = e.target.value;
                    rateDisplay.textContent = rate + 'x';
                    if (this.audioSource) {
                        this.audioSource.playbackRate.setValueAtTime(rate, this.audioContext.currentTime);
                    }
                });
            }
            
            playAudioBuffer() {
                if (!this.currentAudioBuffer) return;
                
                this.audioSource = this.audioContext.createBufferSource();
                this.audioSource.buffer = this.currentAudioBuffer;
                this.audioSource.playbackRate.value = document.getElementById('playback-rate').value;
                this.audioSource.connect(this.audioContext.destination);
                
                this.audioSource.onended = () => {
                    this.audioSource = null;
                    document.getElementById('play-pause').textContent = 'Play';
                };
                
                this.audioSource.start();
            }
        }
        
        // Initialize the audio controller
        document.addEventListener('DOMContentLoaded', () => {
            // Resume audio context on user interaction (required by browsers)
            document.addEventListener('click', () => {
                if (window.audioController && window.audioController.audioContext.state === 'suspended') {
                    window.audioController.audioContext.resume();
                }
            }, { once: true });
            
            window.audioController = new AudioController(); 130);
            ctx.lineTo(450, 130);
            ctx.closePath();
            ctx.fillStyle = '#45b7d1';
            ctx.fill();
            
            // Line
            ctx.beginPath();
            ctx.moveTo(500, 50);
            ctx.lineTo(600, 130);
            ctx.strokeStyle = '#f39c12';
            ctx.lineWidth = 5;
            ctx.stroke();
        }
        
        // Draw text
        function drawText() {
            ctx.font = '30px Arial';
            ctx.fillStyle = '#2c3e50';
            ctx.fillText('Canvas Text', 50, 200);
            
            ctx.font = 'bold 24px Georgia';
            ctx.strokeStyle = '#e74c3c';
            ctx.lineWidth = 2;
            ctx.strokeText('Outlined Text', 50, 250);
        }
        
        // Draw image
        function drawImage() {
            const img = new Image();
            img.onload = function() {
                ctx.drawImage(img, 50, 300, 200, 150);
            };
            img.src = 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAwIiBoZWlnaHQ9IjE1MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICA8cmVjdCB3aWR0aD0iMjAwIiBoZWlnaHQ9IjE1MCIgZmlsbD0iIzNhNzRhMyIvPgogIDx0ZXh0IHg9IjEwMCIgeT0iNzUiIGZvbnQtZmFtaWx5PSJBcmlhbCIgZm9udC1zaXplPSIxNiIgZmlsbD0id2hpdGUiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGR5PSIuM2VtIj5TYW1wbGUgSW1hZ2U8L3RleHQ+Cjwvc3ZnPgo=';
        }
        
        // Clear canvas
        function clearCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }
        
        // Save canvas
        function saveCanvas() {
            const link = document.createElement('a');
            link.download = 'canvas-drawing.png';
            link.href = canvas.toDataURL();
            link.click();
        }
        
        // Interactive drawing
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;
        
        canvas.addEventListener('mousedown', (e) => {
            isDrawing = true;
            [lastX, lastY] = [e.offsetX, e.offsetY];
        });
        
        canvas.addEventListener('mousemove', (e) => {
            if (!isDrawing) return;
            
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(e.offsetX, e.offsetY);
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 2;
            ctx.lineCap = 'round';
            ctx.stroke();
            
            [lastX, lastY] = [e.offsetX, e.offsetY];
        });
        
        canvas.addEventListener('mouseup', () => {
            isDrawing = false;
        });
        
        canvas.addEventListener('mouseout', () => {
            isDrawing = false;
        });
    </script>
</body>
</html>
```

### Web Storage API

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web Storage Demo</title>
    <style>
        .storage-demo {
            max-width: 600px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        .form-group {
            margin: 15px 0;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="text"], textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            padding: 10px 15px;
            margin: 5px;
            cursor: pointer;
            border: none;
            border-radius: 4px;
            background-color: #007bff;
            color: white;
        }
        button:hover {
            background-color: #0056b3;
        }
        .output {
            margin: 15px 0;
            padding: 10px;
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 4px;
            white-space: pre-wrap;
        }
        .storage-list {
            max-height: 200px;
            overflow-y: auto;
            border: 1px solid #ddd;
            padding: 10px;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <h1>Web Storage API Demo</h1>
    
    <div class="storage-demo">
        <h2>Local Storage</h2>
        <div class="form-group">
            <label for="ls-key">Key:</label>
            <input type="text" id="ls-key" placeholder="Enter key">
        </div>
        <div class="form-group">
            <label for="ls-value">Value:</label>
            <textarea id="ls-value" placeholder="Enter value" rows="3"></textarea>
        </div>
        <button onclick="setLocalStorage()">Save to Local Storage</button>
        <button onclick="getLocalStorage()">Get from Local Storage</button>
        <button onclick="removeLocalStorage()">Remove from Local Storage</button>
        <button onclick="clearLocalStorage()">Clear All Local Storage</button>
        <div id="ls-output" class="output"></div>
    </div>
    
    <div class="storage-demo">
        <h2>Session Storage</h2>
        <div class="form-group">
            <label for="ss-key">Key:</label>
            <input type="text" id="ss-key" placeholder="Enter key">
        </div>
        <div class="form-group">
            <label for="ss-value">Value:</label>
            <textarea id="ss-value" placeholder="Enter value" rows="3"></textarea>
        </div>
        <button onclick="setSessionStorage()">Save to Session Storage</button>
        <button onclick="getSessionStorage()">Get from Session Storage</button>
        <button onclick="removeSessionStorage()">Remove from Session Storage</button>
        <button onclick="clearSessionStorage()">Clear All Session Storage</button>
        <div id="ss-output" class="output"></div>
    </div>
    
    <div class="storage-demo">
        <h2>Storage Contents</h2>
        <button onclick="showAllStorage()">Show All Storage</button>
        <div id="storage-list" class="storage-list"></div>
    </div>
    
    <script>
        // Local Storage functions
        function setLocalStorage() {
            const key = document.getElementById('ls-key').value;
            const value = document.getElementById('ls-value').value;
            
            if (key && value) {
                try {
                    localStorage.setItem(key, value);
                    document.getElementById('ls-output').textContent = 
                        `✅ Saved to Local Storage: ${key} = ${value}`;
                } catch (error) {
                    document.getElementById('ls-output').textContent = 
                        `❌ Error: ${error.message}`;
                }
            } else {
                document.getElementById('ls-output').textContent = 
                    '❌ Please enter both key and value';
            }
        }
        
        function getLocalStorage() {
            const key = document.getElementById('ls-key').value;
            
            if (key) {
                const value = localStorage.getItem(key);
                if (value !== null) {
                    document.getElementById('ls-output').textContent = 
                        `✅ Retrieved from Local Storage: ${key} = ${value}`;
                    document.getElementById('ls-value').value = value;
                } else {
                    document.getElementById('ls-output').textContent = 
                        `❌ Key '${key}' not found in Local Storage`;
                }
            } else {
                document.getElementById('ls-output').textContent = 
                    '❌ Please enter a key';
            }
        }
        
        function removeLocalStorage() {
            const key = document.getElementById('ls-key').value;
            
            if (key) {
                localStorage.removeItem(key);
                document.getElementById('ls-output').textContent = 
                    `✅ Removed from Local Storage: ${key}`;
            } else {
                document.getElementById('ls-output').textContent = 
                    '❌ Please enter a key';
            }
        }
        
        function clearLocalStorage() {
            localStorage.clear();
            document.getElementById('ls-output').textContent = 
                '✅ All Local Storage cleared';
        }
        
        // Session Storage functions
        function setSessionStorage() {
            const key = document.getElementById('ss-key').value;
            const value = document.getElementById('ss-value').value;
            
            if (key && value) {
                try {
                    sessionStorage.setItem(key, value);
                    document.getElementById('ss-output').textContent = 
                        `✅ Saved to Session Storage: ${key} = ${value}`;
                } catch (error) {
                    document.getElementById('ss-output').textContent = 
                        `❌ Error: ${error.message}`;
                }
            } else {
                document.getElementById('ss-output').textContent = 
                    '❌ Please enter both key and value';
            }
        }
        
        function getSessionStorage() {
            const key = document.getElementById('ss-key').value;
            
            if (key) {
                const value = sessionStorage.getItem(key);
                if (value !== null) {
                    document.getElementById('ss-output').textContent = 
                        `✅ Retrieved from Session Storage: ${key} = ${value}`;
                    document.getElementById('ss-value').value = value;
                } else {
                    document.getElementById('ss-output').textContent = 
                        `❌ Key '${key}' not found in Session Storage`;
                }
            } else {
                document.getElementById('ss-output').textContent = 
                    '❌ Please enter a key';
            }
        }
        
        function removeSessionStorage() {
            const key = document.getElementById('ss-key').value;
            
            if (key) {
                sessionStorage.removeItem(key);
                document.getElementById('ss-output').textContent = 
                    `✅ Removed from Session Storage: ${key}`;
            } else {
                document.getElementById('ss-output').textContent = 
                    '❌ Please enter a key';
            }
        }
        
        function clearSessionStorage() {
            sessionStorage.clear();
            document.getElementById('ss-output').textContent = 
                '✅ All Session Storage cleared';
        }
        
        // Show all storage
        function showAllStorage() {
            const storageList = document.getElementById('storage-list');
            let content = '';
            
            content += '📦 Local Storage:\n';
            if (localStorage.length === 0) {
                content += '  (empty)\n';
            } else {
                for (let i = 0; i < localStorage.length; i++) {
                    const key = localStorage.key(i);
                    const value = localStorage.getItem(key);
                    content += `  ${key}: ${value}\n`;
                }
            }
            
            content += '\n📦 Session Storage:\n';
            if (sessionStorage.length === 0) {
                content += '  (empty)\n';
            } else {
                for (let i = 0; i < sessionStorage.length; i++) {
                    const key = sessionStorage.key(i);
                    const value = sessionStorage.getItem(key);
                    content += `  ${key}: ${value}\n`;
                }
            }
            
            storageList.textContent = content;
        }
        
        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            showAllStorage();
        });
    </script>
</body>
</html>
```

### Geolocation API

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Geolocation API Demo</title>
    <style>
        .geo-demo {
            max-width: 600px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        button {
            padding: 10px 15px;
            margin: 5px;
            cursor: pointer;
            border: none;
            border-radius: 4px;
            background-color: #28a745;
            color: white;
        }
        button:hover {
            background-color: #218838;
        }
        .location-info {
            margin: 15px 0;
            padding: 15px;
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 4px;
        }
        .map-container {
            margin: 15px 0;
            height: 300px;
            border: 1px solid #ddd;
            border-radius: 4px;
            background-color: #f0f0f0;
            display: flex;
            align-items: center;
            justify-content: center;
        }
    </style>
</head>
<body>
    <h1>Geolocation API Demo</h1>
    
    <div class="geo-demo">
        <h2>Get Your Location</h2>
        <button onclick="getCurrentLocation()">Get Current Location</button>
        <button onclick="watchLocation()">Watch Location Changes</button>
        <button onclick="stopWatching()">Stop Watching</button>
        
        <div id="location-info" class="location-info">
            Click "Get Current Location" to see your coordinates
        </div>
        
        <div id="map-container" class="map-container">
            <p>Map will appear here when location is available</p>
        </div>
    </div>
    
    <script>
        let watchId = null;
        
        // Check if geolocation is supported
        if (!navigator.geolocation) {
            document.getElementById('location-info').innerHTML = 
                '❌ Geolocation is not supported by this browser';
        }
        
        // Get current location
        function getCurrentLocation() {
            if (!navigator.geolocation) {
                showError('Geolocation is not supported');
                return;
            }
            
            document.getElementById('location-info').innerHTML = 
                '📍 Getting your location...';
            
            const options = {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 0
            };
            
            navigator.geolocation.getCurrentPosition(
                showPosition,
                showError,
                options
            );
        }
        
        // Watch location changes
        function watchLocation() {
            if (!navigator.geolocation) {
                showError('Geolocation is not supported');
                return;
            }
            
            if (watchId !== null) {
                navigator.geolocation.clearWatch(watchId);
            }
            
            document.getElementById('location-info').innerHTML = 
                '📍 Watching for location changes...';
            
            const options = {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 0
            };
            
            watchId = navigator.geolocation.watchPosition(
                showPosition,
                showError,
                options
            );
        }
        
        // Stop watching location
        function stopWatching() {
            if (watchId !== null) {
                navigator.geolocation.clearWatch(watchId);
                watchId = null;
                document.getElementById('location-info').innerHTML += 
                    '<br>📍 Stopped watching location changes';
            }
        }
        
        // Show position
        function showPosition(position) {
            const lat = position.coords.latitude;
            const lon = position.coords.longitude;
            const accuracy = position.coords.accuracy;
            const timestamp = new Date(position.timestamp);
            
            const locationInfo = `
                <h3>📍 Your Location</h3>
                <p><strong>Latitude:</strong> ${lat.toFixed(6)}</p>
                <p><strong>Longitude:</strong> ${lon.toFixed(6)}</p>
                <p><strong>Accuracy:</strong> ${accuracy} meters</p>
                <p><strong>Timestamp:</strong> ${timestamp.toLocaleString()}</p>
                ${position.coords.altitude ? `<p><strong>Altitude:</strong> ${position.coords.altitude} meters</p>` : ''}
                ${position.coords.speed ? `<p><strong>Speed:</strong> ${position.coords.speed} m/s</p>` : ''}
                ${position.coords.heading ? `<p><strong>Heading:</strong> ${position.coords.heading}°</p>` : ''}
            `;
            
            document.getElementById('location-info').innerHTML = locationInfo;
            showMap(lat, lon);
        }
        
        // Show error
        function showError(error) {
            let errorMessage = '';
            
            switch(error.code) {
                case error.PERMISSION_DENIED:
                    errorMessage = '❌ Location access denied by user';
                    break;
                case error.POSITION_UNAVAILABLE:
                    errorMessage = '❌ Location information unavailable';
                    break;
                case error.TIMEOUT:
                    errorMessage = '❌ Location request timed out';
                    break;
                default:
                    errorMessage = '❌ An unknown error occurred';
                    break;
            }
            
            document.getElementById('location-info').innerHTML = errorMessage;
        }
        
        // Show map (using OpenStreetMap)
        function showMap(lat, lon) {
            const mapContainer = document.getElementById('map-container');
            
            // Create a simple map using OpenStreetMap tiles
            mapContainer.innerHTML = `
                <iframe
                    width="100%"
                    height="100%"
                    frameborder="0"
                    scrolling="no"
                    marginheight="0"
                    marginwidth="0"
                    src="https://www.openstreetmap.org/export/embed.html?bbox=${lon-0.01}%2C${lat-0.01}%2C${lon+0.01}%2C${lat+0.01}&layer=mapnik&marker=${lat}%2C${lon}"
                    style="border: 1px solid black; border-radius: 4px;">
                </iframe>
            `;
        }
        
        // Distance calculation
        function calculateDistance(lat1, lon1, lat2, lon2) {
            const R = 6371; // Earth's radius in kilometers
            const dLat = (lat2 - lat1) * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                    Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                    Math.sin(dLon/2) * Math.sin(dLon/2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
            return R * c;
        }
        
        // Clean up on page unload
        window.addEventListener('beforeunload', function() {
            if (watchId !== null) {
                navigator.geolocation.clearWatch(watchId);
            }
        });
    </script>
</body>
</html>
```

---

## 📋 Advanced Forms

### Custom Form Validation

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Form Validation</title>
    <style>
        .form-container {
            max-width: 600px;
            margin: 20px auto;
            padding: 30px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background-color: #f9f9f9;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #333;
        }
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007bff;
        }
        input.valid {
            border-color: #28a745;
        }
        input.invalid {
            border-color: #dc3545;
        }
        .error-message {
            color: #dc3545;
            font-size: 14px;
            margin-top: 5px;
            display: none;
        }
        .success-message {
            color: #28a745;
            font-size: 14px;
            margin-top: 5px;
            display: none;
        }
        button {
            background-color: #007bff;
            color: white;
            padding: 12px 24px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin-right: 10px;
        }
        button:hover {
            background-color: #0056b3;
        }
        button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
        .form-row {
            display: flex;
            gap: 20px;
        }
        .form-row .form-group {
            flex: 1;
        }
        .password-strength {
            margin-top: 5px;
            height: 5px;
            background-color: #e9ecef;
            border-radius: 3px;
            overflow: hidden;
        }
        .password-strength-bar {
            height: 100%;
            transition: width 0.3s, background-color 0.3s;
        }
        .strength-weak { background-color: #dc3545; }
        .strength-fair { background-color: #ffc107; }
        .strength-good { background-color: #17a2b8; }
        .strength-strong { background-color: #28a745; }
    </style>
</head>
<body>
    <h1>Advanced Form Validation</h1>
    
    <div class="form-container">
        <form id="registrationForm" novalidate>
            <div class="form-group">
                <label for="fullName">Full Name *</label>
                <input type="text" id="fullName" name="fullName" required>
                <div class="error-message" id="fullNameError"></div>
                <div class="success-message" id="fullNameSuccess"></div>
            </div>
            
            <div class="form-group">
                <label for="email">Email Address *</label>
                <input type="email" id="email" name="email" required>
                <div class="error-message" id="emailError"></div>
                <div class="success-message" id="emailSuccess"></div>
            </div>
            
            <div class="form-row">
                <div class="form-group">
                    <label for="phone">Phone Number</label>
                    <input type="tel" id="phone" name="phone" pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}" placeholder="123-456-7890">
                    <div class="error-message" id="phoneError"></div>
                    <div class="success-message" id="phoneSuccess"></div>
                </div>
                
                <div class="form-group">
                    <label for="birthdate">Date of Birth *</label>
                    <input type="date" id="birthdate" name="birthdate" required>
                    <div class="error-message" id="birthdateError"></div>
                    <div class="success-message" id="birthdateSuccess"></div>
                </div>
            </div>
            
            <div class="form-group">
                <label for="password">Password *</label>
                <input type="password" id="password" name="password" required minlength="8">
                <div class="password-strength">
                    <div class="password-strength-bar" id="passwordStrengthBar"></div>
                </div>
                <div class="error-message" id="passwordError"></div>
                <div class="success-message" id="passwordSuccess"></div>
            </div>
            
            <div class="form-group">
                <label for="confirmPassword">Confirm Password *</label>
                <input type="password" id="confirmPassword" name="confirmPassword" required>
                <div class="error-message" id="confirmPasswordError"></div>
                <div class="success-message" id="confirmPasswordSuccess"></div>
            </div>
            
            <div class="form-group">
                <label for="website">Website URL</label>
                <input type="url" id="website" name="website" placeholder="https://example.com">
                <div class="error-message" id="websiteError"></div>
                <div class="success-message" id="websiteSuccess"></div>
            </div>
            
            <div class="form-group">
                <label for="bio">Bio</label>
                <textarea id="bio" name="bio" rows="4" maxlength="500" placeholder="Tell us about yourself..."></textarea>
                <div class="error-message" id="bioError"></div>
                <div class="success-message" id="bioSuccess"></div>
            </div>
            
            <button type="submit">Register</button>
            <button type="button" onclick="resetForm()">Reset</button>
        </form>
    </div>
    
    <script>
        // Form validation rules
        const validationRules = {
            fullName: {
                required: true,
                minLength: 2,
                pattern: /^[a-zA-Z\s]+$/,
                message: 'Full name must contain only letters and spaces, minimum 2 characters'
            },
            email: {
                required: true,
                pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
                message: 'Please enter a valid email address'
            },
            phone: {
                required: false,
                pattern: /^\d{3}-\d{3}-\d{4}$/,
                message: 'Phone number must be in format: 123-456-7890'
            },
            birthdate: {
                required: true,
                custom: (value) => {
                    const today = new Date();
                    const birthDate = new Date(value);
                    const age = today.getFullYear() - birthDate.getFullYear();
                    return age >= 13 && age <= 120;
                },
                message: 'You must be between 13 and 120 years old'
            },
            password: {
                required: true,
                minLength: 8,
                pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
                message: 'Password must contain at least 8 characters, including uppercase, lowercase, number, and special character'
            },
            confirmPassword: {
                required: true,
                custom: (value) => {
                    return value === document.getElementById('password').value;
                },
                message: 'Passwords do not match'
            },
            website: {
                required: false,
                pattern: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
                message: 'Please enter a valid URL (including http:// or https://)'
            },
            bio: {
                required: false,
                maxLength: 500,
                message: 'Bio must be less than 500 characters'
            }
        };
        
        // Validate individual field
        function validateField(fieldName, value) {
            const rules = validationRules[fieldName];
            if (!rules) return { valid: true };
            
            // Check required
            if (rules.required && (!value || value.trim() === '')) {
                return { valid: false, message: `${fieldName} is required` };
            }
            
            // Skip other validations if field is empty and not required
            if (!rules.required && (!value || value.trim() === '')) {
                return { valid: true };
            }
            
            // Check minimum length
            if (rules.minLength && value.length < rules.minLength) {
                return { valid: false, message: rules.message };
            }
            
            // Check maximum length
            if (rules.maxLength && value.length > rules.maxLength) {
                return { valid: false, message: rules.message };
            }
            
            // Check pattern
            if (rules.pattern && !rules.pattern.test(value)) {
                return { valid: false, message: rules.message };
            }
            
            // Check custom validation
            if (rules.custom && !rules.custom(value)) {
                return { valid: false, message: rules.message };
            }
            
            return { valid: true };
        }
        
        // Update field UI
        function updateFieldUI(fieldName, validation) {
            const field = document.getElementById(fieldName);
            const errorDiv = document.getElementById(fieldName + 'Error');
            const successDiv = document.getElementById(fieldName + 'Success');
            
            if (validation.valid) {
                field.classList.remove('invalid');
                field.classList.add('valid');
                errorDiv.style.display = 'none';
                successDiv.style.display = 'block';
                successDiv.textContent = '✓ Valid';
            } else {
                field.classList.remove('valid');
                field.classList.add('invalid');
                errorDiv.style.display = 'block';
                errorDiv.textContent = validation.message;
                successDiv.style.display = 'none';
            }
        }
        
        // Password strength checker
        function checkPasswordStrength(password) {
            let strength = 0;
            let feedback = '';
            
            if (password.length >= 8) strength++;
            if (password.match(/[a-z]/)) strength++;
            if (password.match(/[A-Z]/)) strength++;
            if (password.match(/[0-9]/)) strength++;
            if (password.match(/[@$!%*?&]/)) strength++;
            
            const strengthBar = document.getElementById('passwordStrengthBar');
            
            switch (strength) {
                case 0:
                case 1:
                    strengthBar.style.width = '25%';
                    strengthBar.className = 'password-strength-bar strength-weak';
                    feedback = 'Weak';
                    break;
                case 2:
                    strengthBar.style.width = '50%';
                    strengthBar.className = 'password-strength-bar strength-fair';
                    feedback = 'Fair';
                    break;
                case 3:
                case 4:
                    strengthBar.style.width = '75%';
                    strengthBar.className = 'password-strength-bar strength-good';
                    feedback = 'Good';
                    break;
                case 5:
                    strengthBar.style.width = '100%';
                    strengthBar.className = 'password-strength-bar strength-strong';
                    feedback = 'Strong';
                    break;
            }
            
            return { strength, feedback };
        }
        
        // Add event listeners
        document.addEventListener('DOMContentLoaded', function() {
            const form = document.getElementById('registrationForm');
            const fields = Object.keys(validationRules);
            
            // Add real-time validation
            fields.forEach(fieldName => {
                const field = document.getElementById(fieldName);
                
                field.addEventListener('blur', function() {
                    const validation = validateField(fieldName, this.value);
                    updateFieldUI(fieldName, validation);
                });
                
                field.addEventListener('input', function() {
                    // Real-time validation for password
                    if (fieldName === 'password') {
                        const strengthInfo = checkPasswordStrength(this.value);
                        
                        // Also revalidate confirm password
                        const confirmPassword = document.getElementById('confirmPassword');
                        if (confirmPassword.value) {
                            const confirmValidation = validateField('confirmPassword', confirmPassword.value);
                            updateFieldUI('confirmPassword', confirmValidation);
                        }
                    }
                    
                    // Real-time validation for confirm password
                    if (fieldName === 'confirmPassword') {
                        const validation = validateField(fieldName, this.value);
                        updateFieldUI(fieldName, validation);
                    }
                });
            });
            
            // Form submission
            form.addEventListener('submit', function(e) {
                e.preventDefault();
                
                let isFormValid = true;
                const formData = new FormData(form);
                
                // Validate all fields
                fields.forEach(fieldName => {
                    const value = formData.get(fieldName) || '';
                    const validation = validateField(fieldName, value);
                    updateFieldUI(fieldName, validation);
                    
                    if (!validation.valid) {
                        isFormValid = false;
                    }
                });
                
                if (isFormValid) {
                    alert('Form submitted successfully!');
                    // Here you would typically send the data to a server
                    console.log('Form data:', Object.fromEntries(formData));
                } else {
                    alert('Please fix the errors in the form');
                }
            });
        });
        
        // Reset form
        function resetForm() {
            const form = document.getElementById('registrationForm');
            form.reset();
            
            // Reset all field states
            Object.keys(validationRules).forEach(fieldName => {
                const field = document.getElementById(fieldName);
                const errorDiv = document.getElementById(fieldName + 'Error');
                const successDiv = document.getElementById(fieldName + 'Success');
                
                field.classList.remove('valid', 'invalid');
                errorDiv.style.display = 'none';
                successDiv.style.display = 'none';
            });
            
            // Reset password strength bar
            const strengthBar = document.getElementById('passwordStrengthBar');
            strengthBar.style.width = '0%';
            strengthBar.className = 'password-strength-bar';
        }
    </script>
</body>
</html>
```

---

## 🎵 Multimedia Integration

### Audio and Video Controls

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Multimedia</title>
    <style>
        .media-container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        .custom-controls {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 10px;
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 4px;
        }
        .control-btn {
            padding: 8px 12px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
        }
        .control-btn:hover {
            background-color: #0056b3;
        }
        .control-btn:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
        .progress-container {
            flex: 1;
            margin: 0 10px;
        }
        .progress-bar {
            width: 100%;
            height: 6px;
            background-color: #e9ecef;
            border-radius: 3px;
            cursor: pointer;
        }
        .progress-fill {
            height: 100%;
            background-color: #007bff;
            border-radius: 3px;
            transition: width 0.1s;
        }
        .volume-container {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        .volume-slider {
            width: 80px;
        }
        .time-display {
            font-family: monospace;
            font-size: 14px;
            color: #666;
        }
        .media-info {
            margin-top: 10px;
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 4px;
            font-size: 14px;
        }
        .playlist {
            margin-top: 20px;
        }
        .playlist-item {
            padding: 10px;
            margin: 5px 0;
            background-color: #ffffff;
            border: 1px solid #ddd;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .playlist-item:hover {
            background-color: #f8f9fa;
        }
        .playlist-item.active {
            background-color: #007bff;
            color: white;
        }
        .recorder-controls {
            margin-top: 20px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 4px;
        }
        .recorded-audio {
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <h1>Advanced Multimedia Controls</h1>
    
    <div class="media-container">
        <h2>Custom Video Player</h2>
        <video id="customVideo" width="100%" height="300" poster="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iODAwIiBoZWlnaHQ9IjMwMCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICA8cmVjdCB3aWR0aD0iODAwIiBoZWlnaHQ9IjMwMCIgZmlsbD0iIzMzMzMzMyIvPgogIDx0ZXh0IHg9IjQwMCIgeT0iMTUwIiBmb250LWZhbWlseT0iQXJpYWwiIGZvbnQtc2l6ZT0iMjAiIGZpbGw9IndoaXRlIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBkeT0iLjNlbSI+VmlkZW8gUGxhY2Vob2xkZXI8L3RleHQ+Cjwvc3ZnPgo=">
            <source src="https://www.w3schools.com/html/mov_bbb.mp4" type="video/mp4">
            Your browser does not support the video tag.
        </video>
        
        <div class="custom-controls">
            <button class="control-btn" id="playPauseBtn">▶️</button>
            <button class="control-btn" id="stopBtn">⏹️</button>
            <button class="control-btn" id="muteBtn">🔊</button>
            
            <div class="progress-container">
                <div class="progress-bar" id="progressBar">
                    <div class="progress-fill" id="progressFill"></div>
                </div>
            </div>
            
            <div class="time-display" id="timeDisplay">0:00 / 0:00</div>
            
            <div class="volume-container">
                <input type="range" class="volume-slider" id="volumeSlider" min="0" max="1" step="0.1" value="1">
                <span id="volumeDisplay">100%</span>
            </div>
            
            <button class="control-btn" id="fullscreenBtn">⛶</button>
        </div>
        
        <div class="media-info" id="videoInfo">
            Video information will appear here
        </div>
    </div>
    
    <div class="media-container">
        <h2>Custom Audio Player with Playlist</h2>
        <audio id="customAudio">
            <source src="https://www.w3schools.com/html/horse.mp3" type="audio/mpeg">
            Your browser does not support the audio tag.
        </audio>
        
        <div class="custom-controls">
            <button class="control-btn" id="audioPlayPauseBtn">▶️</button>
            <button class="control-btn" id="audioStopBtn">⏹️</button>
            <button class="control-btn" id="audioMuteBtn">🔊</button>
            
            <div class="progress-container">
                <div class="progress-bar" id="audioProgressBar">
                    <div class="progress-fill" id="audioProgressFill"></div>
                </div>
            </div>
            
            <div class="time-display" id="audioTimeDisplay">0:00 / 0:00</div>
            
            <div class="volume-container">
                <input type="range" class="volume-slider" id="audioVolumeSlider" min="0" max="1" step="0.1" value="1">
                <span id="audioVolumeDisplay">100%</span>
            </div>
        </div>
        
        <div class="playlist" id="audioPlaylist">
            <h3>Playlist</h3>
            <div class="playlist-item active" data-src="https://www.w3schools.com/html/horse.mp3">
                🎵 Horse Sound
            </div>
            <div class="playlist-item" data-src="https://www.soundjay.com/misc/sounds/bell-ringing-05.wav">
                🔔 Bell Sound
            </div>
        </div>
    </div>
    
    <div class="media-container">
        <h2>Audio Recorder</h2>
        <div class="recorder-controls">
            <button class="control-btn" id="startRecordBtn">🎙️ Start Recording</button>
            <button class="control-btn" id="stopRecordBtn" disabled>⏹️ Stop Recording</button>
            <button class="control-btn" id="clearRecordBtn">🗑️ Clear</button>
            
            <div id="recordingStatus" style="margin-top: 10px;"></div>
            <div id="recordedAudios" class="recorded-audio"></div>
        </div>
    </div>
    
    <script>
        // Custom Video Player
        class CustomVideoPlayer {
            constructor(videoId) {
                this.video = document.getElementById(videoId);
                this.playPauseBtn = document.getElementById('playPauseBtn');
                this.stopBtn = document.getElementById('stopBtn');
                this.muteBtn = document.getElementById('muteBtn');
                this.progressBar = document.getElementById('progressBar');
                this.progressFill = document.getElementById('progressFill');
                this.timeDisplay = document.getElementById('timeDisplay');
                this.volumeSlider = document.getElementById('volumeSlider');
                this.volumeDisplay = document.getElementById('volumeDisplay');
                this.fullscreenBtn = document.getElementById('fullscreenBtn');
                this.videoInfo = document.getElementById('videoInfo');
                
                this.init();
            }
            
            init() {
                // Play/Pause button
                this.playPauseBtn.addEventListener('click', () => {
                    if (this.video.paused) {
                        this.video.play();
                    } else {
                        this.video.pause();
                    }
                });
                
                // Stop button
                this.stopBtn.addEventListener('click', () => {
                    this.video.pause();
                    this.video.currentTime = 0;
                });
                
                // Mute button
                this.muteBtn.addEventListener('click', () => {
                    this.video.muted = !this.video.muted;
                    this.muteBtn.textContent = this.video.muted ? '🔇' : '🔊';
                });
                
                // Progress bar
                this.progressBar.addEventListener('click', (e) => {
                    const rect = this.progressBar.getBoundingClientRect();
                    const percentage = (e.clientX - rect.left) / rect.width;
                    this.video.currentTime = percentage * this.video.duration;
                });
                
                // Volume slider
                this.volumeSlider.addEventListener('input', (e) => {
                    this.video.volume = e.target.value;
                    this.volumeDisplay.textContent = Math.round(e.target.value * 100) + '%';
                });
                
                // Fullscreen button
                this.fullscreenBtn.addEventListener('click', () => {
                    if (this.video.requestFullscreen) {
                        this.video.requestFullscreen();
                    }
                });
                
                // Video events
                this.video.addEventListener('play', () => {
                    this.playPauseBtn.textContent = '⏸️';
                });
                
                this.video.addEventListener('pause', () => {
                    this.playPauseBtn.textContent = '▶️';
                });
                
                this.video.addEventListener('timeupdate', () => {
                    this.updateProgress();
                });
                
                this.video.addEventListener('loadedmetadata', () => {
                    this.updateVideoInfo();
                });
                
                this.video.addEventListener('ended', () => {
                    this.playPauseBtn.textContent = '▶️';
                    this.video.currentTime = 0;
                });
            }
            
            updateProgress() {
                const percentage = (this.video.currentTime / this.video.duration) * 100;
                this.progressFill.style.width = percentage + '%';
                
                const current = this.formatTime(this.video.currentTime);
                const duration = this.formatTime(this.video.duration);
                this.timeDisplay.textContent = `${current} / ${duration}`;
            }
            
            updateVideoInfo() {
                const info = `
                    <strong>Duration:</strong> ${this.formatTime(this.video.duration)} | 
                    <strong>Resolution:</strong> ${this.video.videoWidth}x${this.video.videoHeight} | 
                    <strong>Ready State:</strong> ${this.video.readyState}
                `;
                this.videoInfo.innerHTML = info;
            }
            
            formatTime(seconds) {
                const minutes = Math.floor(seconds / 60);
                const secs = Math.floor(seconds % 60);
                return `${minutes}:${secs.toString().padStart(2, '0')}`;
            }
        }
        
        // Custom Audio Player
        class CustomAudioPlayer {
            constructor(audioId) {
                this.audio = document.getElementById(audioId);
                this.playPauseBtn = document.getElementById('audioPlayPauseBtn');
                this.stopBtn = document.getElementById('audioStopBtn');
                this.muteBtn = document.getElementById('audioMuteBtn');
                this.progressBar = document.getElementById('audioProgressBar');
                this.progressFill = document.getElementById('audioProgressFill');
                this.timeDisplay = document.getElementById('audioTimeDisplay');
                this.volumeSlider = document.getElementById('audioVolumeSlider');
                this.volumeDisplay = document.getElementById('audioVolumeDisplay');
                this.playlist = document.getElementById('audioPlaylist');
                
                this.init();
            }
            
            init() {
                // Play/Pause button
                this.playPauseBtn.addEventListener('click', () => {
                    if (this.audio.paused) {
                        this.audio.play();
                    } else {
                        this.audio.pause();
                    }
                });
                
                // Stop button
                this.stopBtn.addEventListener('click', () => {
                    this.audio.pause();
                    this.audio.currentTime = 0;
                });
                
                // Mute button
                this.muteBtn.addEventListener('click', () => {
                    this.audio.muted = !this.audio.muted;
                    this.muteBtn.textContent = this.audio.muted ? '🔇' : '🔊';
                });
                
                // Progress bar
                this.progressBar.addEventListener('click', (e) => {
                    const rect = this.progressBar.getBoundingClientRect();
                    const percentage = (e.clientX - rect.left) / rect.width;
                    this.audio.currentTime = percentage * this.audio.duration;
                });
                
                // Volume slider
                this.volumeSlider.addEventListener('input', (e) => {
                    this.audio.volume = e.target.value;
                    this.volumeDisplay.textContent = Math.round(e.target.value * 100) + '%';
                });
                
                // Playlist
                this.playlist.addEventListener('click', (e) => {
                    if (e.target.classList.contains('playlist-item')) {
                        const src = e.target.dataset.src;
                        if (src) {
                            this.loadTrack(src);
                            this.setActivePlaylistItem(e.target);
                        }
                    }
                });
                
                // Audio events
                this.audio.addEventListener('play', () => {
                    this.playPauseBtn.textContent = '⏸️';
                });
                
                this.audio.addEventListener('pause', () => {
                    this.playPauseBtn.textContent = '▶️';
                });
                
                this.audio.addEventListener('timeupdate', () => {
                    this.updateProgress();
                });
                
                this.audio.addEventListener('ended', () => {
                    this.playPauseBtn.textContent = '▶️';
                    this.audio.currentTime = 0;
                });
            }
            
            loadTrack(src) {
                this.audio.src = src;
                this.audio.load();
            }
            
            setActivePlaylistItem(item) {
                // Remove active class from all items
                const items = this.playlist.querySelectorAll('.playlist-item');
                items.forEach(i => i.classList.remove('active'));
                
                // Add active class to current item
                item.classList.add('active');
            }
            
            updateProgress() {
                const percentage = (this.audio.currentTime / this.audio.duration) * 100;
                this.progressFill.style.width = percentage + '%';
                
                const current = this.formatTime(this.audio.currentTime);
                const duration = this.formatTime(this.audio.duration);
                this.timeDisplay.textContent = `${current} / ${duration}`;
            }
            
            formatTime(seconds) {
                const minutes = Math.floor(seconds / 60);
                const secs = Math.floor(seconds % 60);
                return `${minutes}:${secs.toString().padStart(2, '0')}`;
            }
        }
        
        // Audio Recorder
        class AudioRecorder {
            constructor() {
                this.mediaRecorder = null;
                this.recordedChunks = [];
                this.startBtn = document.getElementById('startRecordBtn');
                this.stopBtn = document.getElementById('stopRecordBtn');
                this.clearBtn = document.getElementById('clearRecordBtn');
                this.status = document.getElementById('recordingStatus');
                this.container = document.getElementById('recordedAudios');
                this.recordingCounter = 0;
                
                this.init();
            }
            
            init() {
                this.startBtn.addEventListener('click', () => this.startRecording());
                this.stopBtn.addEventListener('click', () => this.stopRecording());
                this.clearBtn.addEventListener('click', () => this.clearRecordings());
            }
            
            async startRecording() {
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                    this.mediaRecorder = new MediaRecorder(stream);
                    this.recordedChunks = [];
                    
                    this.mediaRecorder.ondataavailable = (event) => {
                        if (event.data.size > 0) {
                            this.recordedChunks.push(event.data);
                        }
                    };
                    
                    this.mediaRecorder.onstop = () => {
                        this.createAudioPlayer();
                        stream.getTracks().forEach(track => track.stop());
                    };
                    
                    this.mediaRecorder.start();
                    this.startBtn.disabled = true;
                    this.stopBtn.disabled = false;
                    this.status.textContent = '🔴 Recording...';
                    
                } catch (error) {
                    console.error('Error accessing microphone:', error);
                    this.status.textContent = '❌ Error: Could not access microphone';
                }
            }
            
            stopRecording() {
                if (this.mediaRecorder && this.mediaRecorder.state !== 'inactive') {
                    this.mediaRecorder.stop();
                    this.startBtn.disabled = false;
                    this.stopBtn.disabled = true;
                    this.status.textContent = '✅ Recording stopped';
                }
            }
            
            createAudioPlayer() {
                const blob = new Blob(this.recordedChunks, { type: 'audio/wav' });
                const url = URL.createObjectURL(blob);
                
                this.recordingCounter++;
                
                const audioContainer = document.createElement('div');
                audioContainer.style.margin = '10px 0';
                audioContainer.style.padding = '10px';
                audioContainer.style.border = '1px solid #ddd';
                audioContainer.style.borderRadius = '4px';
                
                audioContainer.innerHTML = `
                    <h4>Recording ${this.recordingCounter}</h4>
                    <audio controls>
                        <source src="${url}" type="audio/wav">
                        Your browser does not support the audio element.
                    </audio>
                    <br>
                    <button onclick="this.parentElement.remove()">Delete</button>
                    <a href="${url}" download="recording-${this.recordingCounter}.wav">
                        <button>Download</button>
                    </a>
                `;
                
                this.container.appendChild(audioContainer);
            }
            
            clearRecordings() {
                this.container.innerHTML = '';
                this.recordingCounter = 0;
                this.status.textContent = '';
            }
        }
        
        // Initialize players
        document.addEventListener('DOMContentLoaded', function() {
            const videoPlayer = new CustomVideoPlayer('customVideo');
            const audioPlayer = new CustomAudioPlayer('customAudio');
            const recorder = new AudioRecorder();
        });
    </script>
</body>
</html>
```

This comprehensive guide demonstrates advanced HTML5 features including Canvas API for graphics, Web Audio API for sound manipulation, and modern web technologies. These APIs enable creating rich, interactive web applications with multimedia capabilities.

---

## 📚 Quick Reference

### HTML5 API Summary

| API | Purpose | Key Features |
|-----|---------|--------------|
| Canvas | 2D/3D Graphics | Drawing, animation, image manipulation |
| Web Audio | Sound processing | Synthesis, effects, visualization |
| Geolocation | Location services | GPS coordinates, position tracking |
| WebRTC | Real-time communication | Video calls, screen sharing |
| Web Storage | Client-side storage | localStorage, sessionStorage |
| Service Workers | Offline functionality | Caching, background sync |
| WebGL | 3D graphics | Hardware-accelerated rendering |

### Best Practices:
1. Always check browser compatibility
2. Provide fallbacks for unsupported features
3. Optimize performance for complex operations
4. Handle errors gracefully
5. Follow accessibility guidelines

### Next Steps:
1. Explore WebGL for 3D graphics
2. Learn Progressive Web App development
3. Master Service Workers for offline functionality
4. Study WebRTC for real-time applications
5. Practice with modern JavaScript frameworks

This advanced HTML guide provides the foundation for building sophisticated web applications with cutting-edge browser APIs and modern web standards. comprehensive Advanced HTML guide demonstrates sophisticated HTML5 features and modern web technologies. Practice these examples to master advanced HTML development!
