## haikyuu

> This will be a volleyball game inspired by the anime Haikyuu. The game will be 3D and playable in the browser.

This will be a volleyball game inspired by the anime Haikyuu. The game will be 3D and playable in the browser.
The game is inspired by the game called Soccer Physics, which is a 2 player game where players can jump and try to hit the ball into each other's net.
It's pretty simple, I want to build a similar game in volleyball.

It will be 3D and multiplayer, where a player can run around the court and jump. The ball will always shoot up and over the net towards the opponent's side if it makes contact with a player's hand.
The player can use WASD controls to move around the court, space bar to jump, and if the ball is ever within animation reach of the hand then the player will trigger a hit animation and hit the ball over the net.

There is no spiking, no setting, and no blocking for now. Just a simple game where if it's close to you you will hit the ball up in the air over the net.
You will be targeting the side based on the angle you are coming at the ball. So if you are coming from it diagonal from the bottom left, then you will hit it towards the top right. 

The game will be implemented in three.js and typescript, with vite as the build tool on the client side. There will also be a socketio integration on the client. 
On the backend side, I will use nodejs with fastify as the framework and socket.io to handle websockets. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/klxu03) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
