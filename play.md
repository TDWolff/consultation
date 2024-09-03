---
course: "consultation"
---

# Consultation

<input type="text" id="username" placeholder="Enter your username" />
<input type="text" id="room-code" placeholder="Enter room code" />
<button id="create-button">Create</button>
<button id="join-button">Join</button>
<button id="leave-button" style="display: none;">Leave</button>
<div id="response-container">
    <!-- put the response here -->
</div>

<script>
    let roomCode = null;
    let pollingInterval = null;

    document.getElementById('create-button').addEventListener('click', function() {
        const username = document.getElementById('username').value;
        fetch('http://localhost:8088/consultation/create', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                course: 'consultation',
                action: 'create',
                username: username
            })
        }).then(response => response.json())
            .then(data => {
                const responseContainer = document.getElementById('response-container');
                responseContainer.innerText = JSON.stringify(data, null, 2);
                roomCode = data.room_code;
                document.getElementById('leave-button').style.display = 'block';
                startPolling();
            });
    });

    document.getElementById('join-button').addEventListener('click', function() {
        const username = document.getElementById('username').value;
        roomCode = document.getElementById('room-code').value;
        fetch('http://localhost:8088/consultation/join', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                course: 'consultation',
                action: 'join',
                username: username,
                roomCode: roomCode
            })
        }).then(response => response.json())
            .then(data => {
                const responseContainer = document.getElementById('response-container');
                responseContainer.innerText = JSON.stringify(data, null, 2);
                document.getElementById('leave-button').style.display = 'block';
                startPolling();
            });
    });

    document.getElementById('leave-button').addEventListener('click', function() {
        leaveRoom();
    });

    window.addEventListener('beforeunload', function(event) {
        if (roomCode) {
            leaveRoom();
        }
    });

    function leaveRoom() {
        fetch('http://localhost:8088/consultation/leave', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                course: 'consultation',
                action: 'leave',
                roomCode: roomCode,
                username: document.getElementById('username').value
            })
        }).then(response => response.json())
            .then(data => {
                const responseContainer = document.getElementById('response-container');
                responseContainer.innerText = JSON.stringify(data, null, 2);
                roomCode = null;
                document.getElementById('leave-button').style.display = 'none';
                stopPolling();
            });
    }

    function startPolling() {
        if (pollingInterval) {
            clearInterval(pollingInterval);
        }
        pollingInterval = setInterval(() => {
            fetch(`http://localhost:8088/consultation/users?roomCode=${roomCode}`)
                .then(response => response.json())
                .then(data => {
                    const responseContainer = document.getElementById('response-container');
                    responseContainer.innerText = JSON.stringify(data, null, 2);
                });
        }, 5000); // Poll every 5 seconds
    }

    function stopPolling() {
        if (pollingInterval) {
            clearInterval(pollingInterval);
            pollingInterval = null;
        }
    }
</script>