# D&D Web Adventure Application
You're building an impressive AI-driven fantasy adventure game! Here's a well-structured repository setup, ready for direct copy-pasting, based on the file structure and contents you provided.
RealmWeaver – Modular AI-Driven Fantasy Adventure
RealmWeaver is an experimental game engine that combines:
 * AI Narration: Uses Hugging Face models (Mistral/Zephyr) for dynamic storytelling.
 * Quest Generation: A simple, randomized quest generator inspired by open-source Storyteller ideas.
 * Sprite-Based Visuals: Maps narrative tags to your sprite assets via spriteMap.json.
 * Map Rendering: A Phaser‑powered map engine that loads Tiled JSON maps.
 * Modular Components: Stub modules for battle logic, tile triggers, and NPC dialogue.
Setup Instructions
 * Clone or copy the repository.
 * Install Dependencies:
   npm install

 * Create a .env File:
   * Place your Hugging Face API token in the .env file as shown below.
 * Add Your Assets:
   * Place sprite images in client/assets/sprites/ (ensure paths match those in server/spriteMap.json).
   * Place your Tiled maps (exported as JSON) in client/assets/maps/ (or in the maps/ folder).
 * Run the Server:
   npm start

 * Open the Game:
   * Navigate to http://localhost:3000 in your browser.
Expand or swap out modules as needed to fit your creative vision!
Happy adventuring!
Folder Structure
realmweaver/
├── client/
│   ├── index.html                // Main HTML page with game UI, Phaser canvas, and sprite preview panel
│   ├── style.css                 // Styling for HUD, story log, sprites, etc.
│   ├── script.js                 // Frontend logic: communicating with server, updating UI
│   ├── mapEngine.js              // Phaser-based map loader (loads Tiled JSON maps)
│   ├── effectLayer.js            // (Stub) For visual effects (spell animations, combat flashes)
│   └── assets/
│       ├── sprites/              // Your sprite images (tiles, enemies, items, etc.)
│       └── maps/                 // Exported Tiled map files (JSON format)
│
├── server/
│   ├── index.js                  // Express server and API routes
│   ├── aiEngine.js               // AI integration (calls Hugging Face and returns narrative with tags)
│   ├── gameManager.js            // Session management, command processing, logging, quest integration
│   ├── fileStorage.js            // Saves game logs to disk
│   ├── questGenerator.js         // Simple quest generator (inspired by Storyteller)
│   ├── spriteMap.json            // Maps AI tags (like "forest", "goblin") to sprite file paths
│   └── modules/                  
│       ├── battleLogic.js        // (Stub) Battle outcome calculations, XP/loot logic
│       ├── tileTriggers.js       // (Stub) Map event triggers (like traps)
│       └── npcDialogue.js        // (Stub) NPC dialogue generator
│
├── maps/                         // (Optional) External Tiled map files if you want to keep them separate
│
├── logs/                         // Game session logs will be saved here
│
├── .env                          // Environment variables (include your HF API token here)
├── package.json                  // NPM dependencies and start script
└── README.md                     // Project overview and instructions

File Contents
1. .env
(Place this file in the project root.)
HF_API_TOKEN=your_huggingface_api_token_here

Replace the value with your actual Hugging Face API token.
2. package.json
{
  "name": "realmweaver",
  "version": "1.0.0",
  "description": "A modular, AI-driven fantasy adventure game built with open source components.",
  "main": "server/index.js",
  "scripts": {
    "start": "node server/index.js"
  },
  "dependencies": {
    "body-parser": "^1.20.2",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "node-fetch": "^2.6.7"
  }
}

3. README.md
(This content is already provided above as the primary README.)
Server Code
4. server/index.js
require('dotenv').config();
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const { createGame, handleCommand } = require('./gameManager');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, '../client')));

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, '../client/index.html'));
});

app.post('/api/createGame', async (req, res) => {
  const { worldDescription, playerClass } = req.body;
  try {
    const data = await createGame(worldDescription, playerClass);
    res.json(data);
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Failed to create game' });
  }
});

app.post('/api/sendCommand', async (req, res) => {
  const { gameId, command } = req.body;
  try {
    const data = await handleCommand(gameId, command);
    res.json(data);
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Command failed' });
  }
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

5. server/aiEngine.js
require('dotenv').config();
const fetch = require('node-fetch');
const fs = require('fs');
const path = require('path');

const MISTRAL_URL = 'https://api-inference.huggingface.co/models/mistralai/Mistral-7B-Instruct-v0.1';
const ZEPHYR_URL = 'https://api-inference.huggingface.co/models/HuggingFaceH4/zephyr-7b-beta';
const HF_API_TOKEN = process.env.HF_API_TOKEN;

// Load the sprite map from file
const spriteMapPath = path.join(__dirname, 'spriteMap.json');
const spriteMap = JSON.parse(fs.readFileSync(spriteMapPath, 'utf8'));

async function queryHuggingFace(modelUrl, prompt) {
  const response = await fetch(modelUrl, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${HF_API_TOKEN}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      inputs: prompt,
      parameters: {
        max_new_tokens: 300,
        temperature: 0.9,
        return_full_text: false
      }
    })
  });
  const data = await response.json();
  const rawOutput = data?.[0]?.generated_text || '';
  try {
    return JSON.parse(rawOutput.trim());
  } catch (err) {
    const fallback = rawOutput.match(/\{[\s\S]+?\}/);
    if (fallback) {
      try {
        return JSON.parse(fallback[0]);
      } catch (e) {
        return { scene: rawOutput.trim() };
      }
    }
    return { scene: rawOutput.trim() };
  }
}

async function queryAI(prompt) {
  try {
    return await queryHuggingFace(MISTRAL_URL, prompt);
  } catch (e) {
    console.error('Mistral failed, trying Zephyr.', e);
    return await queryHuggingFace(ZEPHYR_URL, prompt);
  }
}

function matchSprites(tags) {
  const fallback = {
    background: "assets/sprites/tiles/unknown.png",
    enemy: "assets/sprites/enemies/unknown_enemy.png",
    loot: "assets/sprites/items/unknown_item.png"
  };
  
  return {
    background: spriteMap[tags.biome] || fallback.background,
    enemy: spriteMap[tags.enemy] || fallback.enemy,
    loot: spriteMap[tags.loot] || fallback.loot
  };
}

async function generateIntro(world, playerClass) {
  const introText = `You, a brave ${playerClass}, awaken in ${world}. Darkness looms and adventure calls.`;
  return {
    scene: introText,
    biome: 'unknown',
    enemy: 'none',
    loot: 'none'
  };
}

async function continueStory(session, playerCommand) {
  const prompt = `
You are an AI Dungeon Master.
Player Command: ${playerCommand}
Player Level: ${session.playerStats.level}
Player Class: ${session.playerClass}
World Description: ${session.worldDescription}
Inventory: ${session.playerStats.inventory.join(', ') || 'None'}
Quest Log: ${session.questLog.join('; ') || 'None'}

Return a JSON with the next scene narrative and structured tags as follows:
{
  "scene": "Narrative description of the scene...",
  "biome": "<biome tag>",
  "enemy": "<enemy tag>",
  "loot": "<loot tag>"
}
`;
  const result = await queryAI(prompt);
  const sceneText = result.scene || "The adventure continues...";
  const tags = {
    biome: result.biome || '',
    enemy: result.enemy || '',
    loot: result.loot || ''
  };
  const spriteTags = matchSprites(tags);
  
  return { text: sceneText, spriteTags };
}

module.exports = { generateIntro, continueStory };

6. server/gameManager.js
const path = require('path');
const { generateIntro, continueStory } = require('./aiEngine');
const { saveLog, updateLog } = require('./fileStorage');
const { generateQuest } = require('./questGenerator');

const sessions = {};

function generateUniqueId() {
  return '_' + Math.random().toString(36).substr(2, 9);
}

async function createGame(worldDescription, playerClass) {
  const gameId = generateUniqueId();
  const intro = await generateIntro(worldDescription, playerClass);
  
  // Optionally, generate an opening quest using the quest generator
  const initialQuest = generateQuest();
  
  sessions[gameId] = {
    worldDescription,
    playerClass,
    story: intro.scene + "\nQuest: " + initialQuest,
    playerStats: { level: 1, xp: 0, health: 100, inventory: [] },
    questLog: [initialQuest],
    logFile: path.join(__dirname, '../logs', `${gameId}.json`)
  };
  
  saveLog(sessions[gameId].logFile, sessions[gameId].story);
  return {
    gameId,
    story: sessions[gameId].story,
    playerStats: sessions[gameId].playerStats,
    questLog: sessions[gameId].questLog,
    sceneSprites: { 
      background: "assets/sprites/tiles/unknown.png",
      enemy: "assets/sprites/enemies/unknown_enemy.png",
      loot: "assets/sprites/items/unknown_item.png" 
    }
  };
}

async function handleCommand(gameId, command) {
  const session = sessions[gameId];
  if (!session) throw new Error('Game session not found');

  const updateData = await continueStory(session, command);
  session.story += "\n" + updateData.text;
  updateLog(session.logFile, updateData.text);
  
  // Optionally, add logic around updating quests or generating new quest events here.
  
  return {
    update: updateData.text,
    playerStats: session.playerStats,
    questLog: session.questLog,
    sceneSprites: updateData.spriteTags
  };
}

module.exports = { createGame, handleCommand };

7. server/fileStorage.js
const fs = require('fs');
const path = require('path');

function saveLog(filePath, initialLog) {
  const logData = { entries: [initialLog] };
  const dir = path.dirname(filePath);
  if (!fs.existsSync(dir)) {
    fs.mkdirSync(dir, { recursive: true });
  }
  fs.writeFileSync(filePath, JSON.stringify(logData, null, 2));
}

function updateLog(filePath, newEntry) {
  let logData = { entries: [] };
  if (fs.existsSync(filePath)) {
    const rawData = fs.readFileSync(filePath);
    logData = JSON.parse(rawData);
  }
  logData.entries.push(newEntry);
  fs.writeFileSync(filePath, JSON.stringify(logData, null, 2));
}

module.exports = { saveLog, updateLog };

8. server/questGenerator.js
function generateQuest() {
  const quests = [
    "Retrieve the ancient sword from the dungeon depths.",
    "Rescue the captured villager from the bandit camp.",
    "Collect 5 healing herbs from the eerie swamp.",
    "Defeat the goblin chief terrorizing the forest."
  ];
  return quests[Math.floor(Math.random() * quests.length)];
}

module.exports = { generateQuest };

9. server/spriteMap.json
{
  "forest": "assets/sprites/tiles/forest.png",
  "swamp": "assets/sprites/tiles/swamp.png",
  "dungeon": "assets/sprites/tiles/dungeon.png",
  "goblin": "assets/sprites/enemies/goblin.png",
  "skeleton": "assets/sprites/enemies/skeleton.png",
  "ogre": "assets/sprites/enemies/ogre.png",
  "wyrm": "assets/sprites/enemies/wyrm.png",
  "shadow_wolf": "assets/sprites/enemies/shadow_wolf.png",
  "healing_potion": "assets/sprites/items/potion.png",
  "scroll_of_light": "assets/sprites/items/scroll.png",
  "gold_coin": "assets/sprites/items/gold.png",
  "mystic_relic": "assets/sprites/items/relic.png"
}

10. server/modules/battleLogic.js
function calculateBattleOutcome(player, enemy) {
  // Simple logic: if the player's level is equal to or higher than the enemy's, the player wins.
  if (player.level >= enemy.level) {
    return { victory: true, xpGained: enemy.level * 10 };
  }
  return { victory: false, damage: enemy.level * 5 };
}

module.exports = { calculateBattleOutcome };

11. server/modules/tileTriggers.js
function checkTileEvent(tileType) {
  if (tileType === 'trap') {
    return "You've stepped on a trap! Lose 10 HP.";
  }
  return null;
}

module.exports = { checkTileEvent };

12. server/modules/npcDialogue.js
function getNPCDialogue(npcName) {
  const dialogues = {
    "old_wise": "The forest holds many secrets. Tread carefully.",
    "merchant": "I trade rare items for brave adventurers.",
    "guard": "Halt! What brings you to these parts?"
  };
  return dialogues[npcName] || "Hello there.";
}

module.exports = { getNPCDialogue };

Client Code
13. client/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>RealmWeaver Adventure</title>
  <link rel="stylesheet" href="style.css" />
  <script src="https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js"></script>
</head>
<body>
  <div id="lobby">
    <h1>RealmWeaver Lobby</h1>
    <form id="create-game">
      <input type="text" id="worldDescription" placeholder="Describe your world" required />
      <input type="text" id="playerClass" placeholder="Your class" required />
      <button type="submit">Start Game</button>
    </form>
  </div>

  <div id="game" style="display: none;">
    <h2>RealmWeaver Adventure</h2>
    
    <div id="hud">
      <div class="stat"><span class="icon">❤️</span><span id="hpDisplay">HP: 100</span></div>
      <div class="stat"><span class="icon">⭐</span><span id="xpDisplay">XP: 0 / 50</span></div>
      <div class="stat"><span class="icon">⬆️</span><span id="lvlDisplay">Level: 1</span></div>
    </div>

    <div id="storyWindow"></div>

    <div id="controls">
      <p><strong>Mobile:</strong> Tap/hold + swipe to move<br>
         <strong>Desktop:</strong> Use W A S D keys</p>
      <form id="command-form">
        <input type="text" id="userCommand" placeholder="Enter command" required />
        <button type="submit">Send Command</button>
      </form>
    </div>

    <div id="sceneVisuals">
      <div id="phaser-container" style="width: 600px; height: 400px;"></div>
    </div>

    <div id="spritePreview" style="margin-top: 1rem;">
      <h3>Sprite Preview</h3>
      <div style="display: flex; gap: 1rem;">
        <div>
          <strong>Background:</strong><br />
          <img id="bgSprite" width="96" height="96" />
        </div>
        <div>
          <strong>Enemy:</strong><br />
          <img id="enemyPreview" width="96" height="96" />
        </div>
        <div>
          <strong>Loot:</strong><br />
          <img id="lootSprite" width="96" height="96" />
        </div>
      </div>
    </div>
  </div>

  <script src="script.js"></script>
  <script src="mapEngine.js"></script>
  <script src="effectLayer.js"></script>
</body>
</html>

14. client/style.css
body {
  font-family: Arial, sans-serif;
  margin: 2rem;
  background-color: #f9f9f9;
}

#lobby, #game {
  max-width: 800px;
  margin: auto;
}

#hud {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 1rem;
  font-size: 1.2rem;
}

.stat .icon {
  font-size: 1.4rem;
  margin-right: 0.3rem;
}

#storyWindow {
  border: 1px solid #ccc;
  padding: 1rem;
  height: 300px;
  overflow-y: auto;
  background: #fff;
  margin-bottom: 1rem;
}

#controls {
  margin-top: 1rem;
}

#sceneVisuals {
  margin-top: 1rem;
}

#sceneVisuals img {
  margin: 0.5rem;
  border: 1px solid #ccc;
}

15. client/script.js
let gameId = null;

document.getElementById('create-game').addEventListener('submit', async (e) => {
  e.preventDefault();
  const worldDescription = document.getElementById('worldDescription').value;
  const playerClass = document.getElementById('playerClass').value;
  
  const res = await fetch('/api/createGame', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ worldDescription, playerClass })
  });
  const data = await res.json();
  gameId = data.gameId;
  document.getElementById('lobby').style.display = 'none';
  document.getElementById('game').style.display = 'block';
  updateStory(data.story, data.playerStats, data.questLog);
  updateSpritePreview({
    background: data.sceneSprites ? data.sceneSprites.background : "assets/sprites/tiles/unknown.png",
    enemy: data.sceneSprites ? data.sceneSprites.enemy : "assets/sprites/enemies/unknown_enemy.png",
    loot: data.sceneSprites ? data.sceneSprites.loot : "assets/sprites/items/unknown_item.png"
  });
});

document.getElementById('command-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const command = document.getElementById('userCommand').value;
  const res = await fetch('/api/sendCommand', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ gameId, command })
  });
  const data = await res.json();
  updateStory(data.update, data.playerStats, data.questLog);
  updateSpritePreview(data.sceneSprites);
});

function updateStory(text, stats, questLog) {
  const windowDiv = document.getElementById('storyWindow');
  windowDiv.innerHTML += `<p>${text}</p>`;
  windowDiv.scrollTop = windowDiv.scrollHeight;
  
  // Update HUD
  document.getElementById('hpDisplay').textContent = `HP: ${stats.health}`;
  document.getElementById('xpDisplay').textContent = `XP: ${stats.xp} / ${stats.level * 50}`;
  document.getElementById('lvlDisplay').textContent = `Level: ${stats.level}`;
}

function updateSpritePreview(sprites) {
  if (sprites.background) {
    document.getElementById('bgSprite').src = sprites.background;
  }
  if (sprites.enemy) {
    document.getElementById('enemyPreview').src = sprites.enemy;
  }
  if (sprites.loot) {
    document.getElementById('lootSprite').src = sprites.loot;
  }
}

16. client/mapEngine.js
// This file uses Phaser 3 to load a Tiled JSON map. Adjust the asset paths and key names to match your exported Tiled map.

window.addEventListener('load', () => {
  const config = {
    type: Phaser.AUTO,
    width: 600,
    height: 400,
    parent: 'phaser-container',
    scene: {
      preload: preload,
      create: create,
      update: update
    }
  };

  const game = new Phaser.Game(config);
  let map, tileset, layer;

  function preload() {
    // Load the Tiled JSON map. Place the JSON file in client/assets/maps/
    this.load.tilemapTiledJSON('map', 'assets/maps/forest_start.json');
    // Load the tileset image. Adjust the key and path according to your asset.
    this.load.image('tiles', 'assets/sprites/tiles/forest.png');
  }

  function create() {
    map = this.make.tilemap({ key: 'map' });
    // Replace 'YourTilesetName' with the name used in Tiled
    tileset = map.addTilesetImage('YourTilesetName', 'tiles');
    layer = map.createLayer('Tile Layer 1', tileset, 0, 0);
  }

  function update() {
    // Optional: Add logic for player movement, collisions, etc.
  }
});

17. client/effectLayer.js
// This file is a stub for adding visual effects (such as spell animations, combat flashes, or particle effects).

// Stub: Implement visual effects (spell animations, combat flashes, particles) here.
console.log("effectLayer loaded. Implement visual effects as needed.");

Final Steps
 * Place Your Assets:
   * Add your sprite images in client/assets/sprites/ so that they match the paths defined in server/spriteMap.json.
   * Place your Tiled map JSON files in client/assets/maps/ (or in the separate maps/ folder as desired).
 * Install Dependencies & Run:
   In your project root, run:
   npm install
npm start

 * Open Your Game:
   * Navigate to http://localhost:3000 in your browser to test your RealmWeaver adventure!

This project is licensed under the MIT License.
