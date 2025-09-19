<!DOCTYPE html>
<html>
<head>
    <title>Scaledrone Chat</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: sans-serif;
            background-color: #f5f5f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        .chat-container {
            width: 100%;
            max-width: 600px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            height: 100%;
            overflow: hidden;
        }
        .messages {
            flex-grow: 1;
            padding: 20px;
            overflow-y: auto;
            border-bottom: 1px solid #eee;
        }
        .message {
            margin-bottom: 15px;
            display: flex;
            align-items: flex-start;
        }
        .message .avatar {
            width: 30px;
            height: 30px;
            background-color: #4CAF50;
            color: white;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            flex-shrink: 0;
            margin-right: 10px;
        }
        .message .content {
            background: #e9e9eb;
            padding: 10px 15px;
            border-radius: 20px;
            max-width: 80%;
        }
        .message.own .avatar {
            order: 2;
            margin-right: 0;
            margin-left: 10px;
            background-color: #2196F3;
        }
        .message.own .content {
            background: #2196F3;
            color: white;
            order: 1;
        }
        .message-form {
            display: flex;
            padding: 15px;
            border-top: 1px solid #eee;
        }
        .message-form input[type="text"] {
            flex-grow: 1;
            border: 1px solid #ddd;
            border-radius: 20px;
            padding: 10px 15px;
            margin-right: 10px;
        }
        .message-form button {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 20px;
            cursor: pointer;
        }
        .message-form button:hover {
            background: #45a049;
        }
        .username-prompt {
            padding: 20px;
            text-align: center;
        }
        .username-prompt input {
            padding: 8px;
            border-radius: 5px;
            border: 1px solid #ccc;
        }
        .username-prompt button {
            padding: 8px 12px;
            border-radius: 5px;
            border: none;
            background-color: #007bff;
            color: white;
            cursor: pointer;
        }
        .username-prompt button:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="username-prompt" id="username-prompt">
            <h2>Enter your custom username</h2>
            <input type="text" id="username-input" placeholder="Your username">
            <button onclick="setUsername()">Start Chatting</button>
        </div>
        <div class="messages" id="messages"></div>
        <form class="message-form" id="message-form">
            <input type="text" id="message-input" placeholder="Type a message..." disabled>
            <button type="submit" disabled>Send</button>
        </form>
    </div>

    <script src="https://cdn.scaledrone.com/scaledrone.min.js"></script>
    <script>
        const CHANNEL_ID = '43VnIkeRto0nANvC'; // Replace with your Channel ID
        const ROOM_NAME = 'observable-custom-chat';

        let drone;
        let username;

        function setUsername() {
            username = document.getElementById('username-input').value;
            if (!username) {
                alert('Please enter a username.');
                return;
            }

            document.getElementById('username-prompt').style.display = 'none';
            document.getElementById('message-input').disabled = false;
            document.querySelector('.message-form button').disabled = false;

            drone = new ScaleDrone(CHANNEL_ID, {
                data: {
                    name: username
                }
            });

            drone.on('open', error => {
                if (error) {
                    console.error(error);
                }
                const room = drone.subscribe(ROOM_NAME);
                room.on('data', (data, member) => {
                    displayMessage(data, member);
                });
            });

            document.getElementById('message-form').addEventListener('submit', (event) => {
                event.preventDefault();
                const input = document.getElementById('message-input');
                const message = input.value;
                if (message.trim()) {
                    drone.publish({
                        room: ROOM_NAME,
                        message: message
                    });
                    input.value = '';
                }
            });
        }

        function displayMessage(data, member) {
            const messagesEl = document.getElementById('messages');
            const isOwnMessage = member.clientData.name === username;

            const messageEl = document.createElement('div');
            messageEl.className = `message ${isOwnMessage ? 'own' : ''}`;

            const avatarEl = document.createElement('div');
            avatarEl.className = 'avatar';
            avatarEl.textContent = member.clientData.name.charAt(0);

            const contentEl = document.createElement('div');
            contentEl.className = 'content';
            contentEl.textContent = `${member.clientData.name}: ${data}`;

            messageEl.appendChild(avatarEl);
            messageEl.appendChild(contentEl);

            messagesEl.appendChild(messageEl);
            messagesEl.scrollTop = messagesEl.scrollHeight;
        }
    </script>
</body>
</html>
