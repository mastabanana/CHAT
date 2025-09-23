<!DOCTYPE html>
<html>
<head>
    <title>Terminal Chatroom</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { 
            font-family: 'Source Code Pro', monospace; 
            margin: 0; 
            padding: 10px; 
            background-color: #000; 
            color: #0f0; 
            overflow: hidden;
            position: absolute;
            top: 0;
            left: 0;
            font-size: 10px;
        }
        .container { 
            max-width: 900px; 
            margin: 0; 
            padding: 0; 
            background: transparent; 
            box-shadow: none;
        }
        .messages { 
            list-style-type: none; 
            padding: 0; 
            margin-bottom: 5px; 
            max-height: 90vh;
            overflow-y: auto; 
            border: none;
        }
        .message { 
            padding: 2px 0; 
            margin-bottom: 2px; 
            border-radius: 0; 
            border: none;
        }
        .own-message, .other-message { 
            background-color: transparent; 
            text-align: left; 
        }
        .username { 
            font-weight: bold; 
        }
        .message-content { 
            word-wrap: break-word; 
        }
        .prompt-container {
            display: flex;
            align-items: center;
            position: fixed;
            bottom: 10px;
            left: 10px;
            width: calc(100% - 20px);
        }
        .prompt-container input { 
            flex-grow: 1; 
            border: none; 
            padding: 5px 0; 
            border-radius: 0; 
            background-color: transparent;
            color: #0f0;
            font-family: 'Source Code Pro', monospace;
            font-size: 10px;
            outline: none;
            caret-color: transparent; /* Hide native cursor */
        }
        .prompt-container span.blinking-cursor {
            color: #0f0;
            animation: blink 1s step-end infinite;
        }
        @keyframes blink {
            from, to { opacity: 0; }
            50% { opacity: 1; }
        }
        .welcome-text {
            color: #0f0;
            padding: 2px 0;
            margin-bottom: 2px;
        }
    </style>
</head>
<body>
    <div class="container">
        <ul id="messages" class="messages"></ul>
        <div class="prompt-container">
            <div id="username-prompt" style="display: block;">
                <span id="prompt-text">user@local:&nbsp;</span>
                <input id="username-input" type="text" autofocus>
                <span id="username-cursor" class="blinking-cursor">█</span>
            </div>
            <div id="chat-prompt" style="display: none;">
                <span id="chat-prompt-text">$&nbsp;</span>
                <input id="message-input" type="text">
                <span id="chat-cursor" class="blinking-cursor">█</span>
            </div>
        </div>
    </div>

    <!-- Scaledrone client library -->
    <script src="https://cdn.scaledrone.com/scaledrone.min.js"></script>

    <script>
        const CHANNEL_ID = 't9OHWbkbZdHg7tCb';
        let drone = null;
        let room = null;
        let me = {};

        const messagesList = document.getElementById('messages');
        const usernameInput = document.getElementById('username-input');
        const messageInput = document.getElementById('message-input');
        const usernamePrompt = document.getElementById('username-prompt');
        const chatPrompt = document.getElementById('chat-prompt');

        function joinChat(username) {
            me = {
                id: Math.random().toString(36).substring(2, 15),
                name: username
            };

            usernamePrompt.style.display = 'none';
            chatPrompt.style.display = 'flex'; // Use flex to align prompt and input
            messageInput.focus();

            drone = new ScaleDrone(CHANNEL_ID, { data: me });

            drone.on('open', error => {
                if (error) {
                    return console.error(error);
                }
                
                addMessageToScreen('Connection established. Type your messages below.', 'system');
                
                room = drone.subscribe('general-chat');
                room.on('data', (message, client) => {
                    const isOwnMessage = client && client.id === drone.clientId;
                    addMessageToScreen(message.content, message.sender, isOwnMessage);
                });
            });
        }

        function sendMessage() {
            const messageText = messageInput.value.trim();
            if (messageText) {
                drone.publish({
                    room: 'general-chat',
                    message: {
                        content: messageText,
                        sender: me.name
                    }
                });
                messageInput.value = '';
            }
        }

        function addMessageToScreen(content, sender, isOwnMessage) {
            const messageElement = document.createElement('li');
            messageElement.classList.add('message');
            
            const usernameSpan = document.createElement('span');
            usernameSpan.classList.add('username');
            usernameSpan.textContent = sender + ": ";
            
            const messageContentSpan = document.createElement('span');
            messageContentSpan.classList.add('message-content');
            messageContentSpan.textContent = content;

            if (!isOwnMessage) {
                 messageElement.appendChild(usernameSpan);
            }

            messageElement.appendChild(messageContentSpan);
            messagesList.appendChild(messageElement);
            messagesList.scrollTop = messagesList.scrollHeight;
        }
        
        // Handle initial username submission
        usernameInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                const username = this.value.trim();
                if (username) {
                    addMessageToScreen(username, 'user@local');
                    joinChat(username);
                }
            }
        });

        // Handle sending messages
        messageInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        });

        // Toggle cursor visibility based on input focus
        function handleInputFocus(input, cursor) {
            input.addEventListener('focus', () => cursor.style.display = 'inline');
            input.addEventListener('blur', () => cursor.style.display = 'none');
        }

        handleInputFocus(usernameInput, document.getElementById('username-cursor'));
        handleInputFocus(messageInput, document.getElementById('chat-cursor'));

        // Initial welcome message
        addMessageToScreen('Welcome to the terminal chatroom.', 'system');
        addMessageToScreen('Please enter your username to join.', 'system');
    </script>
</body>
</html>

