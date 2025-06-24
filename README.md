RealmWeaver – Modular AI-Driven Fantasy Adventure
Welcome to RealmWeaver, an experimental game engine that merges dynamic AI narration with classic sprite-based visuals to create unique fantasy adventures. This project is built with open-source components, offering a flexible and expandable framework for your creative visions.
Features
 * AI-Powered Narration: Experience dynamic storytelling driven by Hugging Face models (Mistral/Zephyr), generating real-time narratives as you play.
 * Procedural Quest Generation: Embark on quests created on the fly by a simple, randomized quest generator, inspired by open-source Storyteller concepts.
 * Sprite-Based Visuals: Watch the world come to life with visuals mapped from narrative tags directly to your custom sprite assets via spriteMap.json.
 * Phaser Map Rendering: Explore expansive worlds thanks to a Phaser-powered map engine that seamlessly loads your Tiled JSON maps.
 * Modular Architecture: Easily expand and customize the game with stub modules for battle logic, tile triggers, and NPC dialogue, ready for your implementation.
Getting Started
Follow these simple steps to get your RealmWeaver adventure up and running.
1. Project Setup
First, get the project files onto your local machine. You can do this by cloning the repository or simply copying the project contents.
2. Install Dependencies
Navigate to the project's root directory in your terminal and install all necessary packages:
npm install

3. Configure Your Hugging Face API Token
Create a file named .env in the root of your project and add your Hugging Face API token to it. This token is crucial for the AI narration features.
HF_API_TOKEN=your_huggingface_api_token_here

Remember to replace your_huggingface_api_token_here with your actual token.
4. Add Your Game Assets
Bring your game world to life by placing your visual assets in the correct directories:
 * Sprites: Add your character sprites, enemy models, item icons, and environmental tiles to client/assets/sprites/. Ensure the file paths match the entries in server/spriteMap.json for proper mapping with the AI narrative tags.
 * Maps: Place your Tiled maps (exported as JSON files) into client/assets/maps/. You can also use a separate maps/ folder if you prefer to keep them outside the client directory.
5. Run the Server
Once your dependencies are installed and assets are in place, start the game server from your project root:
npm start

6. Play the Game
Open your web browser and navigate to http://localhost:3000. You're now ready to embark on your AI-driven fantasy adventure!
Expanding RealmWeaver
RealmWeaver is designed to be modular. Feel free to dive into the server/modules/ directory to expand on the battle logic, tile triggers, or NPC dialogue. Whether you want to add complex combat systems, intricate environmental puzzles, or branching dialogue trees, the framework is ready for your enhancements.
Happy adventuring!


To get RealmWeaver up and running, you'll need the following prerequisites and dependencies:
Prerequisites
These are the foundational tools you'll need installed on your system before you start:
 * Node.js: RealmWeaver's server-side logic and build processes rely on Node.js. It comes bundled with npm (Node Package Manager).
   * How to check: Open your terminal or command prompt and type node -v and npm -v. If you see version numbers, you're good.
   * How to install: Download the recommended version from the official Node.js website.
 * A Hugging Face API Token: This is essential for the AI narration feature to work. You'll use this token in your .env file.
   * How to get one: Sign up on the Hugging Face website and then generate an API token from your profile settings.
Dependencies
These are the software libraries and packages that RealmWeaver uses. They are automatically installed via npm once you have Node.js set up.
 * express: A fast, unopinionated, minimalist web framework for Node.js, used to build the server and API routes.
 * body-parser: A Node.js middleware for handling JSON, raw, text, and URL-encoded form data. It's used here to parse incoming request bodies.
 * dotenv: A zero-dependency module that loads environment variables from a .env file into process.env. This is how your Hugging Face API token is securely loaded.
 * node-fetch: A light-weight module that brings the window.fetch API to Node.js, used for making HTTP requests to the Hugging Face API.
 * Phaser: A fast, free, and fun open-source HTML5 game framework. It's loaded via CDN in client/index.html and used by client/mapEngine.js for map rendering.
Once you have Node.js installed, you can automatically handle the dependencies by running npm install in your project's root directory. After that, make sure your .env file has your Hugging Face API token, and you'll be ready to launch the game!

    