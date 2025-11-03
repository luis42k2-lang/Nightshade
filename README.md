<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nightshade AI</title>

    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#1a1a2e">
    
    <style>
        /* Basic Reset and Setup */
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a2e;
            color: #e94560;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
            user-select: none; /* Disable text selection for app-like feel */
        }
        
        /* Header/Title Area */
        h1 {
            color: #53dfa3;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        /* Conversation Log */
        #output {
            width: 100%;
            max-width: 600px;
            height: 60vh;
            padding: 15px;
            margin-bottom: 20px;
            background-color: #2c0b3c;
            border-radius: 12px;
            overflow-y: auto;
            border: 2px solid #53dfa3;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
            font-size: 1.1em;
            line-height: 1.6;
        }

        .message {
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            opacity: 0.9;
            transition: opacity 0.3s;
        }

        .user {
            background-color: #e94560;
            color: #1a1a2e;
            text-align: right;
            margin-left: 20%;
        }

        .ai {
            background-color: #4b0e77;
            color: #fff;
            text-align: left;
            margin-right: 20%;
        }

        /* Input and Controls Area */
        #controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 400px;
        }

        /* Microphone Button */
        #mic-button {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: #e94560;
            color: #1a1a2e;
            border: none;
            font-size: 2.5em;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: background-color 0.2s, transform 0.1s;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        #mic-button:active {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* Voice Unlock Feature Display */
        #voice-unlock {
            background-color: #2c0b3c;
            color: #e94560;
            padding: 8px 15px;
            border-radius: 5px;
            font-size: 0.9em;
            border: 1px dashed #e94560;
        }
    </style>
</head>
<body>

    <h1>Nightshade AI</h1>

    <div id="output">
        <div class="message ai">Hello! I am Nightshade. Tap the mic to start talking, or say the Voice Unlock phrase.</div>
    </div>

    <div id="controls">
        <div id="status">Tap and hold to talk</div>
        
        <button id="mic-button" aria-label="Push to Talk">
            ðŸŽ™ï¸
        </button>

        <div id="voice-unlock">Voice Unlock Phrase: **"System check complete"**</div>
    </div>

    <script>
        const micButton = document.getElementById('mic-button');
        const statusDiv = document.getElementById('status');
        const outputDiv = document.getElementById('output');
        const VOICE_UNLOCK_PHRASE = "system check complete";
        
        let recognition = null;
        let isListening = false;
        let isUnlocked = false;

        // --- Core Speech Recognition Setup ---
        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window)) {
                statusDiv.textContent = "Speech recognition not supported in this browser.";
                return null;
            }

            const recognitionInstance = new webkitSpeechRecognition();
            recognitionInstance.continuous = false; // Stop after a single phrase
            recognitionInstance.interimResults = false; // Only final results
            recognitionInstance.lang = 'en-US';

            recognitionInstance.onstart = () => {
                isListening = true;
                statusDiv.textContent = isUnlocked ? "Listening..." : "Waiting for unlock phrase...";
                micButton.style.backgroundColor = '#53dfa3'; // Change color when listening
            };

            recognitionInstance.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase().trim();
                
                if (!isUnlocked) {
                    // --- Voice Unlock Logic ---
                    if (transcript.includes(VOICE_UNLOCK_PHRASE)) {
                        isUnlocked = true;
                        addMessage('user', `(Whisper) ${transcript}`);
                        addMessage('ai', "Access Granted. Welcome, user. How may I assist you?");
                        document.getElementById('voice-unlock').innerHTML = "Status: **Unlocked**";
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    } else {
                        addMessage('user', `(Attempted Unlock) ${transcript}`);
                        addMessage('ai', "Access Denied. Please state the correct unlock phrase.");
                        micButton.style.backgroundColor = '#e94560'; // Reset color
                    }
                } else {
                    // --- Standard Conversation Logic ---
                    addMessage('user', transcript);
                    // Simulate AI Response (replace this with actual API call later)
                    setTimeout(() => {
                        const aiResponse = generateSimpleResponse(transcript);
                        addMessage('ai', aiResponse);
                        speakResponse(aiResponse);
                    }, 1000);
                }
            };

            recognitionInstance.onerror = (event) => {
                console.error('Speech Recognition Error:', event.error);
                statusDiv.textContent = "Error: Try again.";
                isListening = false;
                micButton.style.backgroundColor = '#e94560';
            };

            recognitionInstance.onend = () => {
                isListening = false;
                if(isUnlocked) {
                    statusDiv.textContent = "Ready to talk. Tap and hold.";
                } else {
                    statusDiv.textContent = "Tap and hold to attempt unlock.";
                }
                micButton.style.backgroundColor = '#e94560';
            };

            return recognitionInstance;
        }

        // --- Text-to-Speech (TTS) Function ---
        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'en-US';
                speechSynthesis.speak(utterance);
            }
        }

        // --- Simple Simulated AI Response (Placeholder) ---
        function generateSimpleResponse(userText) {
            if (userText.includes("how are you")) {
                return "I am functioning optimally, thank you for asking.";
            } else if (userText.includes("your name")) {
                return "I am Nightshade, an artificial intelligence assistant.";
            } else if (userText.includes("pwa")) {
                return "A PWA is a Progressive Web App, combining the best of web and mobile apps.";
            } else {
                return `Processing command for: "${userText}". How else can I assist?`;
            }
        }

        // --- Message Display Function ---
        function addMessage(sender, text) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}`;
            msgDiv.textContent = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight; // Auto-scroll to the latest message
        }

        // --- Event Listeners for PTT (Push-to-Talk) ---
        function startListening() {
            if (isListening) return;
            if (!recognition) {
                recognition = setupSpeechRecognition();
                if (!recognition) return;
            }
            // Reset recognition to start fresh
            recognition.start();
        }

        // PTT: Start listening on press, stop on release (or timeout)
        micButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); // Prevent accidental scrolling/other mobile behavior
            startListening();
        });

        micButton.addEventListener('mousedown', startListening); // For desktop testing

        // --- PWA Service Worker Registration (for offline functionality/caching) ---
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/service-worker.js')
                    .then(reg => console.log('Service Worker registered: ', reg.scope))
                    .catch(err => console.error('Service Worker registration failed: ', err));
            });
        }
        
        // Initial setup prompt
        statusDiv.textContent = "Tap and hold to attempt unlock.";
    </script>
</body>
</html>