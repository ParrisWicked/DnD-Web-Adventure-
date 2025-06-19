# D&D Web Adventure Application

This repository contains a D&D web adventure application that integrates AI storytelling. Follow the steps below to set up and run the application on your local machine.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
  - [Step 1: Clone the Repository](#step-1-clone-the-repository)
  - [Step 2: Install Dependencies](#step-2-install-dependencies)
  - [Step 3: Set Up the Project](#step-3-set-up-the-project)
  - [Step 4: Run the Application](#step-4-run-the-application)
- [Project Structure](#project-structure-1)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

- Node.js (v14 or later)
- npm (v6 or later)

## Project Structure

```
dnd-web-adventure/
│
├─ server/
│  ├─ index.js
│
├─ client/
│  ├─ index.html
│  ├─ script.js
│  ├─ style.css
│
├─ package.json
├─ package-lock.json
```

## Setup Instructions

### Step 1: Clone the Repository

Clone this repository to your local machine:

```sh
git clone https://github.com/yourusername/dnd-web-adventure.git
cd dnd-web-adventure
```

### Step 2: Install Dependencies

Install the necessary npm dependencies:

```sh
npm install
```

### Step 3: Set Up the Project

Create the necessary directories and files as outlined in the project structure above. Add the following code to the respective files:

#### `server/index.js`

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const axios = require('axios');

const app = express();
const PORT = process.env.PORT || 3000;

const gameStates = {};

// Middleware
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, '../client')));

// Basic route
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, '../client/index.html'));
});

// API routes
app.post('/api/createGame', async (req, res) => {
  const { worldDescription, playerClass } = req.body;
  const gameId = generateUniqueId();
  try {
    const aiResponse = await axios.post('https://api.bolt.new/generateStory', {
      worldDescription,
      playerClass
    });
    gameStates[gameId] = { worldDescription, playerClass, story: aiResponse.data.story };
    res.json({ gameId, story: aiResponse.data.story });
  } catch (error) {
    res.status(500).json({ error: 'Failed to generate story' });
  }
});

app.post('/api/sendCommand', async (req, res) => {
  const { gameId, command } = req.body;
  if (gameStates[gameId]) {
    try {
      const aiResponse = await axios.post('https://api.bolt.new/generateUpdate', {
        command,
        context: gameStates[gameId].story
      });
      gameStates[gameId].story += `\n${aiResponse.data.update}`;
      res.json({ update: aiResponse.data.update });
    } catch (error) {
      res.status(500).json({ error: 'Failed to generate update' });
    }
  } else {
    res.status(404).json({ error: 'Game session not found' });
  }
});

function generateUniqueId() {
  return '_' + Math.random().toString(36).substr(2, 9);
}

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### `client/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>D&D Web Adventure</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="lobby">
    <h1>Welcome to the D&D Web Adventure</h1>
    <form id="gameForm">
      <label for="worldInput">Describe Your World:</label>
      <textarea id="worldInput" placeholder="E.g., a dark fantasy realm or a steampunk city"></textarea>

      <label for="classSelect">Choose Your Class:</label>
      <select id="classSelect">
        <option value="fighter">Fighter</option>
        <option value="wizard">Wizard</option>
        <option value="rogue">Rogue</option>
        <option value="cleric">Cleric</option>
      </select>

      <button type="submit">Start Game</button>
    </form>
  </div>

  <div id="gameScreen" style="display:none;">
    <h2>Your Adventure Awaits!</h2>
    <input type="hidden" id="gameId">
    <div id="storyBoard"></div>
    <input type="text" id="playerCommand" placeholder="Enter your action...">
    <button id="sendCommand">Send</button>
  </div>

  <script src="script.js"></script>
</body>
</html>
```

#### `client/script.js`

```javascript
document.getElementById('gameForm').addEventListener('submit', async (e) => {
  e.preventDefault();

  const worldDescription = document.getElementById('worldInput').value;
  const playerClass = document.getElementById('classSelect').value;

  try {
    const response = await fetch('/api/createGame', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ worldDescription, playerClass })
    });

    const data = await response.json();
    document.getElementById('lobby').style.display = 'none';
    document.getElementById('gameScreen').style.display = 'block';
    document.getElementById('storyBoard').innerText = data.story;
    document.getElementById('gameId').value = data.gameId; // Store gameId for future commands
  } catch (error) {
    console.error('Error starting game:', error);
    alert('Failed to start game. Please try again.');
  }
});

// Handling player commands
document.getElementById('sendCommand').addEventListener('click', async () => {
  const command = document.getElementById('playerCommand').value;
  const gameId = document.getElementById('gameId').value;

  if (!command.trim()) return;

  try {
    const response = await fetch('/api/sendCommand', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ gameId, command })
    });

    const data = await response.json();
    const storyBoard = document.getElementById('storyBoard');
    storyBoard.innerText += '\n\n' + data.update;
    document.getElementById('playerCommand').value = '';
  } catch (error) {
    console.error('Error sending command:', error);
    alert('Failed to send command. Please try again.');
  }
});

// Allow Enter key to send commands
document.getElementById('playerCommand').addEventListener('keypress', (e) => {
  if (e.key === 'Enter') {
    document.getElementById('sendCommand').click();
  }
});
```

#### `client/style.css`

```css
body {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
  background-image: url('path/to/background-image.jpg');
  background-size: cover;
  background-position: center;
}

#lobby {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
  color: #333;
  text-align: center;
  margin-bottom: 30px;
}

form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

label {
  font-weight: bold;
  color: #555;
}

textarea, select, input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
}

textarea {
  min-height: 100px;
  resize: vertical;
}

button {
  background-color: #007bff;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}

#gameScreen {
  background: rgba(255, 255, 255, 0.9);
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  border: 2px solid #007bff;
}

#storyBoard {
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 5px;
  margin: 20px 0;
  min-height: 200px;
  border-left: 6px solid #007bff;
  white-space: pre-wrap;
}

#playerCommand {
  width: calc(100% - 100px);
  display: inline-block;
  margin-right: 10px;
}

#sendCommand {
  width: 80px;
  display: inline-block;
}
```

### Step 4: Run the Application

Start the server by running:

```sh
node server/index.js
```

Open your browser and navigate to `http://localhost:3000`. You should see the D&D web adventure application interface. You can describe your world, choose your class, and start your adventure. The application will generate a story based on your inputs and allow you to send commands to progress the narrative.

## Project Structure

```
dnd-web-adventure/
│
├─ server/
│  ├─ index.js
│
├─ client/
│  ├─ index.html
│  ├─ script.js
│  ├─ style.css
│
├─ package.json
├─ package-lock.json
```

## Usage

1. **Start the Server**: Run `node server/index.js` to start the Express server.
2. **Open the Application**: Navigate to `http://localhost:3000` in your web browser.
3. **Describe Your World**: Enter a description of the world you want to adventure in.
4. **Choose Your Class**: Select your character class from the dropdown menu.
5. **Start the Game**: Click the "Start Game" button to begin your adventure.
6. **Send Commands**: Enter commands in the input field and click "Send" or press Enter to progress the story.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License.
