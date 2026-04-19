## xeocast-video-service

> - This API service provides functionality to generate and publish a video file taking as input a background image file and an audio file, or publish a video file from a pre-existing video file. It operates asynchronously using a task-based system. When a generate process is initiated, a task ID is returned immediately. The final results (or any errors) are delivered via a POST request to a user-provided callback URL once the task completes.

About this project:

- This API service provides functionality to generate and publish a video file taking as input a background image file and an audio file, or publish a video file from a pre-existing video file. It operates asynchronously using a task-based system. When a generate process is initiated, a task ID is returned immediately. The final results (or any errors) are delivered via a POST request to a user-provided callback URL once the task completes.
- The project architecture follows the Model-View-Controller (MVC) pattern.
- The tasks storage is just in memory.
- The generated videos are shared in a public static web server protected by a signature.

About technologies in general:

- The service is written in Python using FastAPI.
- The video generation is done using a custom script that uses the `moviepy` library.
- Always use the latest version of all required Python packages

About the API:

- Don't use '/v1/' or any other version number in endpoints
- Always follows the Model-View-Controller (MVC) pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xeocast) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
