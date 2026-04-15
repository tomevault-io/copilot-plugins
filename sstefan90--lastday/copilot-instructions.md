## lastday

> - Use Unity's MonoBehaviour patterns

# Unity C# Project Rules - Last Day

## Code Style
- Use Unity's MonoBehaviour patterns
- Prefer [SerializeField] private over public fields
- Use [Header] attributes for Inspector organization
- Include XML documentation for public methods
- Organize with namespaces: LastDay.Core, LastDay.Player, LastDay.Interaction, etc.

## Unity Specifics
- Always null-check GetComponent results
- Use TryGetComponent when appropriate
- Avoid Find() in Update loops
- Use events (GameEvents static class) over direct references
- Singletons inherit from Singleton<T> base class

## Project Context
- This is a 2D point-and-click narrative game about an elderly man deciding on medical assistance in dying
- Using LLMUnity for AI dialogue (currently in stub mode)
- Pixel art aesthetic (Stardew Valley style)
- Target platform: macOS (M4 Apple Silicon)
- Unity 2022.3 LTS, C# 9.0

## Architecture
- Singleton managers: GameManager, EventManager, AudioManager, FadeManager, GameStateMachine
- Event-based communication via GameEvents static class
- State machine: Loading -> Playing <-> InDialogue/PhoneCall/Decision -> Ending
- Interaction: hover for gaze, click to interact, events published via EventManager

## File Locations
- Scripts: Assets/Scripts/{Core,Player,Pathfinding,NPC,Interaction,Dialogue,UI,Audio,Camera,Data,Utilities}/
- Prefabs: Assets/Prefabs/
- Art: Assets/Art/
- Audio: Assets/Audio/
- Data: Assets/Data/

## When Debugging
- Check GameState.txt export if available
- Check EventManager.triggeredMemories for game progress
- Check GameStateMachine.CurrentState for flow issues
- Common issues: null refs from unassigned Inspector fields, event double-subscription, state transition failures

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sstefan90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
