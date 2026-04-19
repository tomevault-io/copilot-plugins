## astridmart

> Let's build a fun simple arcade retail game for home use for my 4 year old. It's a combination of hardware and software and should result in a game console we can plug into TV or monitor and have a fun experience.

# Arcade Retail Store game

## Description
Let's build a fun simple arcade retail game for home use for my 4 year old. It's a combination of hardware and software and should result in a game console we can plug into TV or monitor and have a fun experience.

Hardware I will use:
- Raspberry Pi. The game should run when the pi gets connected to power
- Arcade buttons and joystick attached to a controller and then to Raspberry Pi
- A 1D and 2D barcode scanner
- Optionally - a mobile app to control the SKUs (products corresponding to the numbers the scanner scans)

## Logic of the game
The game should have two modes:
- Retail mode. This is to simulate the experience of shopping and scanning the products. It adds up the prices, creates a virtual receipt, and lets the kid "pay" for the product.
- Timer mode. Products get shown on the screen and the kid has to find the item and scan them as quickly as possible. The score is how many items they got right and how fast.

## Principles to follow
- The game needs to work offline. Don't rely on any API calls.
- Inserting SKUs needs to be straightfoward - either the parent can do it from the game environment or from their phone via bluetooth. BONUS: if we can take photos of the products and the game generates arcade versions of it (this requires an API call).
- Keep the code as simple as possible. It's for home use so we don't need to worry about fancy database or security protocols.
- Build in testing mode on desktop. All buttons should also have corresponding keys on keyboard so the game can be developed and tested without the arcade setup.
- Design should be a 16-bit arcade game. Think of early 1990s. Add sound effects and visuals for fun. I'll create the images and sounds as needed, for example for staple items like bread, pasta, milk, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lauri245) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
