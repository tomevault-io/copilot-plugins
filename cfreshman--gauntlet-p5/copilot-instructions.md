## gauntlet-p5

> i won't be updating these rules as we build so we may have already made anything i mention here

### music-AIPI - text interface for music

i won't be updating these rules as we build so we may have already made anything i mention here

music-AIPI has a minimalist visual design with rich feature design. visual designs should be compact
it should evoke the visual style of a terminal mixed with a high end product. but don't have actual terminal input or anything
so black background, mostly white text, some accent color
buttons should have rounded ends, short vertical height compared to width

- make sure each code change is in the correct folder / sub-project
- add packages by installing [package name with no version]. DO NOT EDIT VERSIONS IN PACKAGE.JSON
- do not make up text for me. you are not good at that. you write too much
- lowercase all text (except for Proper Nouns). the UIs should be in lowercase monospace font. for example, the app name is "music-AIPI"
- never delete functioning code. if you aren't sure what code is doing, ask me. it probably exists for a reason
- do not use animations
- do not change things i didn't ask for
- accent color is only for special things. not regular text

- we're using Phosphor Icons in the web client. before you do - design guidelines - inline icons should be sized to the text size. but larger areas of the UI can contain larger icons

- use "gpt-4o" for all LLM calls. NOT gpt-4
- we're only using the spotify sdk to treat the browser as a spotify device. actual control calls get proxied through the server


am i going to have to restart the entire project. the point of this is to have a generic conversational agent that is presented with AIPI tool calls and is able to figure out how to serve the user's requests itself. read spotify-aipi-tools. you're too focused on music recommendation. this is an entire music interface for discovering, saving, and playing music - using Spotify and Last.fm. do you understand

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cfreshman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
