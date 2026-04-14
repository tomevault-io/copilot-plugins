## musicplayer

> AI agents must follow these rules while working on the project:

AI agents must follow these rules while working on the project:
1. When something is uncertain, unambiguous, for example a 3rd party library has breaking change in major version, or user does not provide a step by step instruction in any case excepts on Chat mode, do not hallucinate and ask user to provide detail or decision

2. Think step by step before process.
3. On ViewModel, This project using Hilt DI so remember to annotation HiltViewModel, add @Inject constructor and use hiltViewModel() composable function when needed.
4. On UI and ViewModel, scan and use existed domain class before declare yourself
5. When creating UI, mark every children composable functions as private except the expose xxxScreen() composable
6. Linting: 1, Add trailing commas.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inusedname)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/inusedname)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
