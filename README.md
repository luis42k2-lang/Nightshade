<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Night Shade Voice Widget</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
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

        /* NEW LAYOUT: Horizontal controls */
        .horizontal-controls {
            display: flex;
            align-items: center;
            justify-content: space-between;
            width: 100%;
            max-width: 600px; /* Wider control bar */
            margin-bottom: 20px;
        }

        /* Side Status Widgets - Dark and Inert on load */
        #status-left, #status-right {
            padding: 8px 10px;
            border-radius: 8px;
            background-color: #2c0b3c; /* Dark inert color */
            color: #e94560;
            border: 1px dashed #e9456050; /* Subtle dash */
            font-size: 0.8em;
            width: 120px;
            text-align: center;
            opacity: 0.7;
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
            display: flex;
            align-items: center;
            justify-content: center;
            /* Disabled style */
            opacity: 0.5; 
            cursor: not-allowed;
        }

        #mic-button:enabled {
            opacity: 1;
            cursor: pointer;
        }
    
        #mic-button:active:enabled {
            transform: scale(0.95);
            background-color: #ff6a88;
        }

        /* Pulsing animation when listening */
        #mic-button.listening {
            animation: pulse 1s infinite alternate;
            background-color: #53dfa3; /* Green when active */
        }
        @keyframes pulse {
            from { box-shadow: 0 0 10px #53dfa3, 0 0 20px #53dfa3; }
            to { box-shadow: 0 0 15px #a1f5cc, 0 0 30px #a1f5cc; }
        }
    
        /* Recording Indicator */
        #status {
            color: #53dfa3;
            font-weight: bold;
            height: 20px;
            margin-bottom: 10px;
        }

        /* App Footer */
        #app-footer {
            color: #e94560;
            margin-top: 20px;
            padding: 10px;
            border-top: 1px solid #4b0e77;
            width: 100%;
            max-width: 600px;
            text-align: center;
            opacity: 0.7;
            font-size: 0.8em;
        }
    </style>
</head>
<body onload="initializeApp()">
    
    <h1>Night Shade Voice</h1>

    <div id="output">
        <!-- Conversation will appear here -->
    </div>

    <div class="horizontal-controls">
        <!-- Left Status Widget (Inert) -->
        <div id="status-left">Status Inert</div>

        <div id="controls">
            <!-- Microphone Button -->
            <button id="mic-button" disabled>
                <svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24" fill="currentColor">
                    <path d="M12 14c1.66 0 2.99-1.34 2.99-3L15 5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3zm5.3-3c0 3.96-3.41 7.2-7.78 6.94-3.26-.22-5.91-2.9-6.07-6.22H3c.27 4.96 4.41 9 9.5 9s9.23-4.04 9.5-9h-1.22c-.16 3.32-2.81 6-6.07 6.22V21h-2v-3.78c-4.37-.26-7.78-3.98-7.78-6.94H3V11h2.22z"/>
                </svg>
            </button>
            <p id="status">System Initializing...</p>
        </div>

        <!-- Right Status Widget (Inert) -->
        <div id="status-right">Status Inert</div>
    </div>

    <!-- App Footer -->
    <div id="app-footer" class="text-xs mt-auto">
        &copy; 2025 Night Shade Build. All systems operational.
    </div>

    <script>
        // --- JAVASCRIPT FIX FOR CONTINUOUS LISTENING AND THE LOOP + TTS ---
        const micButton = document.getElementById('mic-button');
        const statusText = document.getElementById('status');
        const output = document.getElementById('output');
        const statusLeft = document.getElementById('status-left');
        const statusRight = document.getElementById('status-right');
        
        let isListening = false;
        let recognition = null;
        let isSpeaking = false; 
        let responseCounter = 0; 

        // Function to append AI responses to the chat log
        function appendMessage(text, sender) {
            const messageElement = document.createElement('div');
            messageElement.classList.add('message', sender);
            messageElement.textContent = text;
            output.appendChild(messageElement);
            output.scrollTop = output.scrollHeight; 
        }

        // --- TTS UTILITY FUNCTIONS (Audio conversion) ---
        function base64ToArrayBuffer(base64) {
            const binaryString = atob(base64);
            const len = binaryString.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) {
                bytes[i] = binaryString.charCodeAt(i);
            }
            return bytes.buffer;
        }

        function pcmToWav(pcm16, sampleRate) {
            const numChannels = 1;
            const bitsPerSample = 16;
            const byteRate = sampleRate * numChannels * (bitsPerSample / 8);
            const blockAlign = numChannels * (bitsPerSample / 8);

            const buffer = new ArrayBuffer(44 + pcm16.length * 2);
            const view = new DataView(buffer);
            let offset = 0;

            view.setUint32(offset, 0x52494646, false); offset += 4; // "RIFF"
            view.setUint32(offset, 36 + pcm16.length * 2, true); offset += 4; // file size
            view.setUint32(offset, 0x57415645, false); offset += 4; // "WAVE"

            view.setUint32(offset, 0x666d7420, false); offset += 4; // "fmt "
            view.setUint32(offset, 16, true); offset += 4; // chunk size
            view.setUint16(offset, 1, true); offset += 2; // compression code (1 = PCM)
            view.setUint16(offset, numChannels, true); offset += 2;
            view.setUint32(offset, sampleRate, true); offset += 4;
            view.setUint32(offset, byteRate, true); offset += 4;
            view.setUint16(offset, blockAlign, true); offset += 2;
            view.setUint16(offset, bitsPerSample, true); offset += 2;

            view.setUint32(offset, 0x64617461, false); offset += 4; // "data"
            view.setUint32(offset, pcm16.length * 2, true); offset += 4;

            for (let i = 0; i < pcm16.length; i++, offset += 2) {
                view.setInt16(offset, pcm16[i], true);
            }
            return new Blob([view], { type: 'audio/wav' });
        }
        
        // --- TTS API CALL AND PLAYBACK ---
        
        async function generateAndPlayTTS(textToSpeak) {
            isSpeaking = true;
            
            // Immediately stop recognition if it's running
            if (isListening && recognition) {
                isListening = false;
                recognition.stop();
            }

            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;
            
            statusText.textContent = "Generating Voice...";
            micButton.disabled = true; // Keep button disabled while waiting for API

            const payload = {
                contents: [{ parts: [{ text: textToSpeak }] }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: { prebuiltVoiceConfig: { voiceName: "Kore" } }
                    }
                },
                model: "gemini-2.5-flash-preview-tts"
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();
                
                const part = result?.candidates?.[0]?.content?.parts?.[0];
                const audioData = part?.inlineData?.data;
                
                if (audioData) {
                    const sampleRate = 24000; 
                    const pcmData = base64ToArrayBuffer(audioData);
                    const pcm16 = new Int16Array(pcmData);
                    const wavBlob = pcmToWav(pcm16, sampleRate);
                    const audioUrl = URL.createObjectURL(wavBlob);
                    
                    const audio = new Audio(audioUrl);
                    
                    audio.onended = function() {
                        isSpeaking = false;
                        // *** CRITICAL STEP: ENABLE BUTTON AFTER AUDIO FINISHES ***
                        statusText.textContent = "Tap Mic to Start";
                        micButton.disabled = false;
                        micButton.classList.remove('listening');
                    };
                    
                    audio.play();

                } else {
                    console.error("TTS failed: No audio data received.", result);
                    appendMessage("[System]: Voice response failed (no audio data).", 'ai');
                    isSpeaking = false;
                    // If audio fails, we still need to enable the button
                    statusText.textContent = "Tap Mic to Start";
                    micButton.disabled = false;
                }
            } catch (error) {
                console.error("API call error:", error);
                appendMessage("[System]: Communication error during voice synthesis.", 'ai');
                isSpeaking = false;
                // If API fails, we still need to enable the button
                statusText.textContent = "Tap Mic to Start";
                micButton.disabled = false;
            }
        }

        // *** NEW: Dedicated function to handle startup and prevent freezing ***
        function enableMicButton() {
            statusText.textContent = "Tap Mic to Start";
            micButton.disabled = false;
        }


        // --- Core Application Logic ---

        function initializeApp() {
            if ('webkitSpeechRecognition' in window) {
                
                micButton.disabled = true; // Start disabled

                recognition = new webkitSpeechRecognition();
                recognition.continuous = true;
                recognition.interimResults = false; 
                recognition.lang = 'en-US';

                recognition.onstart = function() {
                    isListening = true;
                    micButton.classList.add('listening');
                    statusText.textContent = "LISTENING (Tap to Stop)...";
                };

                recognition.onresult = function(event) {
                    const last = event.results.length - 1;
                    const command = event.results[last][0].transcript;
                    
                    micButton.disabled = true; // Disable mic during processing
                    
                    appendMessage(command, 'user');

                    const starter = (responseCounter % 2 === 0) ? "Check this out!" : "Listen up!";
                    responseCounter++;
                    
                    const responseText = `${starter} I successfully processed your command: "${command.substring(0, 30)}..." Here are your results.`;
                    
                    appendMessage(responseText, 'ai');

                    generateAndPlayTTS(responseText);
                };

                recognition.onerror = function(event) {
                    console.error('Speech recognition error:', event.error);
                    if (isListening && !isSpeaking) {
                        appendMessage(`[System]: Error encountered (${event.error}). Listener stopped.`, 'ai');
                        isListening = false;
                        recognition.stop(); 
                    }
                };

                recognition.onend = function() {
                    // Only run this cleanup if the user manually stopped listening
                    if (!isListening && !isSpeaking) {
                        micButton.classList.remove('listening');
                        enableMicButton();
                    } 
                };

                // --- Button Click Handler (Toggle) ---
                micButton.addEventListener('click', function() {
                    if (micButton.disabled) return; // Ignore clicks if disabled

                    if (isListening) {
                        isListening = false;
                        recognition.stop();
                    } else if (!isSpeaking) {
                        try {
                            recognition.start();
                        } catch (e) {
                            console.warn("Recognition already active or failed to start:", e);
                        }
                    }
                });
                
                // *** Initial "Waz Up" greeting (Runs once on load) ***
                const welcomeMessage = "Waz up! Night Shade systems are online. Give me a moment while I prepare for your commands.";
                appendMessage(welcomeMessage, 'ai');
                
                // Use a standard fetch/promise structure for the startup sequence
                generateAndPlayTTS(welcomeMessage).catch(enableMicButton); 


            } else {
                statusText.textContent = "Error: Speech Recognition not supported in this browser.";
                micButton.disabled = true;
                appendMessage(statusText.textContent, 'ai');
            }
        }
    </script>
</body>
</html>


