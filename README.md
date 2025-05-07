# chat-app
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Chat</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f4f4f4;
        }
        h1, h2, h3 {
            color: #333;
        }
        .section {
            margin: 20px 0;
            padding: 15px;
            background: white;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        label {
            display: block;
            margin: 10px 0 5px;
            font-weight: bold;
        }
        input, button {
            width: 100%;
            padding: 8px;
            margin: 5px 0;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #218838;
        }
        #messages, #public-messages {
            height: 150px;
            overflow-y: auto;
            border: 1px solid #ccc;
            padding: 10px;
            background: #f9f9f9;
            margin-top: 10px;
        }
        .error {
            color: red;
            font-size: 0.9em;
        }
        #login-section {
            display: block;
        }
        #chat-section, #private-chat-section {
            display: none;
        }
    </style>
</head>
<body>
    <h1>Simple Chat</h1>

    <div id="login-section" class="section">
        <h2>Login</h2>
        <label for="username">Username:</label>
        <input type="text" id="username" placeholder="Enter username">
        <button onclick="login()">Login</button>
        <p id="login-error" class="error"></p>
    </div>

    <div id="chat-section" class="section">
        <h2>Chat Homepage</h2>
        <div class="section">
            <h3>Public Chat</h3>
            <label for="public-message">Message:</label>
            <input type="text" id="public-message" placeholder="Enter your message">
            <button onclick="sendPublicMessage()">Send Message</button>
            <div id="public-messages"></div>
        </div>

        <div class="section">
            <h3>Create Private Chat</h3>
            <label for="create-room">Room ID:</label>
            <input type="text" id="create-room" placeholder="Enter room ID (e.g., room123)">
            <label for="create-password">Password:</label>
            <input type="password" id="create-password" placeholder="Enter chat password">
            <button onclick="createChat()">Create Chat</button>
        </div>

        <div class="section">
            <h3>Join Private Chat</h3>
            <label for="join-room">Room ID:</label>
            <input type="text" id="join-room" placeholder="Enter room ID (e.g., room123)">
            <label for="join-password">Password:</label>
            <input type="password" id="join-password" placeholder="Enter chat password">
            <button onclick="joinChat()">Join Chat</button>
        </div>
    </div>

    <div id="private-chat-section" class="section">
        <h2>Private Chat: <span id="current-room"></span></h2>
        <div class="section">
            <h3>Send Message</h3>
            <label for="message">Message:</label>
            <input type="text" id="message" placeholder="Enter your message">
            <button onclick="sendPrivateMessage()">Send Message</button>
            <div id="messages"></div>
        </div>
        <button onclick="backToHome()">Back to Home</button>
    </div>

    <script>
        let currentRoom = '';
        let currentPassword = '';

        // Login
        function login() {
            const username = document.getElementById('username').value;
            if (!username) {
                document.getElementById('login-error').textContent = 'Username required';
                return;
            }
            localStorage.setItem('username', username);
            document.getElementById('login-section').style.display = 'none';
            document.getElementById('chat-section').style.display = 'block';
            loadPublicMessages();
            logPublicMessage(`Logged in as ${username}`);
            startPolling();
        }

        // Auto-login
        if (localStorage.getItem('username')) {
            document.getElementById('login-section').style.display = 'none';
            document.getElementById('chat-section').style.display = 'block';
            loadPublicMessages();
            startPolling();
            logPublicMessage(`Logged in as ${localStorage.getItem('username')}`);
        }

        // Listen for storage changes
        window.addEventListener('storage', (event) => {
            if (event.key === 'public-messages') {
                loadPublicMessages();
            } else if (event.key === `room-${currentRoom}-messages`) {
                loadPrivateMessages(currentRoom, currentPassword);
            }
        });

        // Poll for updates
        let lastPublicHash = '';
        let lastPrivateHash = '';
        function startPolling() {
            setInterval(() => {
                // Check public messages
                const publicMessages = localStorage.getItem('public-messages') || '[]';
                const publicHash = btoa(publicMessages);
                if (publicHash !== lastPublicHash) {
                    loadPublicMessages();
                    lastPublicHash = publicHash;
                }
                // Check private messages
                if (currentRoom) {
                    const privateMessages = localStorage.getItem(`room-${currentRoom}-messages`) || '[]';
                    const privateHash = btoa(privateMessages);
                    if (privateHash !== lastPrivateHash) {
                        loadPrivateMessages(currentRoom, currentPassword);
                        lastPrivateHash = privateHash;
                    }
                }
            }, 1000);
        }

        // Load public messages
        function loadPublicMessages() {
            const messages = JSON.parse(localStorage.getItem('public-messages') || '[]');
            const messagesDiv = document.getElementById('public-messages');
            messagesDiv.innerHTML = '';
            messages.forEach(msg => {
                const p = document.createElement('p');
                p.textContent = `${msg.username}: ${msg.text}`;
                messagesDiv.appendChild(p);
            });
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        // Send public message
        function sendPublicMessage() {
            const message = document.getElementById('public-message').value;
            if (!message) {
                logPublicMessage('Error: Message required', 'error');
                return;
            }
            const username = localStorage.getItem('username');
            const messages = JSON.parse(localStorage.getItem('public-messages') || '[]');
            messages.push({ username, text: message });
            localStorage.setItem('public-messages', JSON.stringify(messages));
            loadPublicMessages();
            document.getElementById('public-message').value = '';
        }

        // Log public messages
        function logPublicMessage(message, className = '') {
            const messagesDiv = document.getElementById('public-messages');
            const p = document.createElement('p');
            p.textContent = message;
            if (className) p.className = className;
            messagesDiv.appendChild(p);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        // Create chat
        function createChat() {
            const room = document.getElementById('create-room').value;
            const password = document.getElementById('create-password').value;
            if (!room || !password) {
                logPublicMessage('Error: Room ID and password required', 'error');
                return;
            }
            localStorage.setItem(`room-${room}-password`, password);
            localStorage.setItem(`room-${room}-messages`, JSON.stringify([]));
            currentRoom = room;
            currentPassword = password;
            document.getElementById('current-room').textContent = room;
            document.getElementById('chat-section').style.display = 'none';
            document.getElementById('private-chat-section').style.display = 'block';
            loadPrivateMessages(room, password);
            logPrivateMessage('Private chat created.');
        }

        // Join chat
        function joinChat() {
            const room = document.getElementById('join-room').value;
            const password = document.getElementById('join-password').value;
            if (!room || !password) {
                logPublicMessage('Error: Room ID and password required', 'error');
                return;
            }
            const storedPassword = localStorage.getItem(`room-${room}-password`);
            if (password !== storedPassword) {
                logPublicMessage('Error: Incorrect password', 'error');
                return;
            }
            currentRoom = room;
            currentPassword = password;
            document.getElementById('current-room').textContent = room;
            document.getElementById('chat-section').style.display = 'none';
            document.getElementById('private-chat-section').style.display = 'block';
            loadPrivateMessages(room, password);
            logPrivateMessage('Joined private chat.');
        }

        // Load private messages
        function loadPrivateMessages(room, password) {
            const storedPassword = localStorage.getItem(`room-${room}-password`);
            const messagesDiv = document.getElementById('messages');
            messagesDiv.innerHTML = '';
            if (password !== storedPassword) {
                logPrivateMessage('Error: Incorrect password', 'error');
                return;
            }
            const messages = JSON.parse(localStorage.getItem(`room-${room}-messages`) || '[]');
            messages.forEach(msg => {
                const p = document.createElement('p');
                p.textContent = `${msg.username}: ${msg.text}`;
                messagesDiv.appendChild(p);
            });
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        // Send private message
        function sendPrivateMessage() {
            const message = document.getElementById('message').value;
            if (!message) {
                logPrivateMessage('Error: Message required', 'error');
                return;
            }
            const username = localStorage.getItem('username');
            const messages = JSON.parse(localStorage.getItem(`room-${currentRoom}-messages`) || '[]');
            messages.push({ username, text: message });
            localStorage.setItem(`room-${currentRoom}-messages`, JSON.stringify(messages));
            loadPrivateMessages(currentRoom, currentPassword);
            document.getElementById('message').value = '';
        }

        Castello: "Send private message"
        // Log private messages
        function logPrivateMessage(message, className = '') {
            const messagesDiv = document.getElementById('messages');
            const p = document.createElement('p');
            p.textContent = message;
            if (className) p.className = className;
            messagesDiv.appendChild(p);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        // Back to home
        function backToHome() {
            document.getElementById('private-chat-section').style.display = 'none';
            document.getElementById('chat-section').style.display = 'block';
            document.getElementById('create-room').value = '';
            document.getElementById('create-password').value = '';
            document.getElementById('join-room').value = '';
            document.getElementById('join-password').value = '';
            currentRoom = '';
            currentPassword = '';
            loadPublicMessages();
        }
    </script>
</body>
</html>
