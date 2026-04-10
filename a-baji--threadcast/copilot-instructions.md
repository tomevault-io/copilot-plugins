## threadcast

> Threadcast is a Flutter mobile app (iOS + Android) that converts Reddit posts from story-driven subreddits (e.g. r/AITAH, r/AmItheAsshole, r/relationship_advice) into multi-speaker, AI-generated podcast audio. The user pastes one or more Reddit URLs, the app scrapes the post and comments, an on-device LLM generates a faithful podcast-style transcript, and an on-device TTS engine synthesizes the audio with distinct voices per speaker.

# Threadcast — CLAUDE.md

## Project Overview

Threadcast is a Flutter mobile app (iOS + Android) that converts Reddit posts from story-driven subreddits (e.g. r/AITAH, r/AmItheAsshole, r/relationship_advice) into multi-speaker, AI-generated podcast audio. The user pastes one or more Reddit URLs, the app scrapes the post and comments, an on-device LLM generates a faithful podcast-style transcript, and an on-device TTS engine synthesizes the audio with distinct voices per speaker.

**Core constraints:**
- Zero paid API dependencies. Free to run indefinitely.
- All ML inference runs entirely on-device. No cloud calls for LLM or TTS.
- Cross-platform: Flutter for shared UI/logic, native platform channels for LLM only.

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI framework | Flutter (Dart) |
| State management | Riverpod |
| Navigation | go_router |
| Android LLM | Gemini Nano via Android AICore (ML Kit / Google AI Edge) |
| iOS LLM | Apple Foundation Models framework (iOS 18+) |
| TTS engine | Kokoro-82M via sherpa_onnx Flutter plugin |
| Reddit API | OAuth 2.0, script-type app, free tier (100 QPM) |
| HTTP client | dio |
| Local database | Drift + drift_flutter (actively maintained SQLite ORM) |
| Audio playback | just_audio |
| Audio export | ffmpeg_kit_flutter_new (community fork of retired ffmpeg_kit_flutter) |
| File sharing | share_plus |
| Secure storage | flutter_secure_storage (OAuth tokens) |
| Platform channels | Standard Flutter MethodChannel (LLM bridge only) |

---

## Minimum OS Requirements (Phase 1)

- **Android 16+** (API level 36) — required for guaranteed AICore / Gemini Nano availability
- **iOS 18+** — required for Foundation Models framework

Devices below these versions see a "coming soon" screen with an explanation. No generation functionality is exposed on unsupported devices. The hybrid fallback (private on-device model for older OS) is a planned Phase 2 feature — do not implement it now, but architect the LLM abstraction layer to accommodate it cleanly.

---

## Project Structure

```
threadcast/
├── android/
│   └── app/src/main/kotlin/com/threadcast/app/
│       ├── MainActivity.kt
│       └── llm/
│           ├── LlmPlugin.kt           # MethodChannel handler
│           └── GeminiNanoService.kt   # AICore integration
├── ios/
│   └── Runner/
│       ├── AppDelegate.swift
│       └── llm/
│           ├── LlmPlugin.swift        # MethodChannel handler
│           └── FoundationModelService.swift
├── lib/
│   ├── main.dart
│   ├── app.dart                       # GoRouter + Riverpod ProviderScope
│   ├── core/
│   │   ├── constants.dart
│   │   ├── errors.dart                # Typed error classes
│   │   └── extensions.dart
│   ├── features/
│   │   ├── onboarding/                # Reddit OAuth + first-run setup
│   │   │   ├── onboarding_screen.dart
│   │   │   └── onboarding_provider.dart
│   │   ├── create/                    # URL input + generation flow
│   │   │   ├── create_screen.dart
│   │   │   ├── create_provider.dart
│   │   │   └── widgets/
│   │   │       ├── url_input_field.dart
│   │   │       ├── url_chip_list.dart      # Multi-URL list
│   │   │       └── generation_progress.dart
│   │   ├── player/                    # Audio playback
│   │   │   ├── player_screen.dart
│   │   │   ├── player_provider.dart
│   │   │   └── widgets/
│   │   │       ├── waveform_display.dart
│   │   │       ├── playback_controls.dart
│   │   │       └── transcript_view.dart   # Scrolling transcript synced to audio
│   │   └── library/                   # Saved episodes
│   │       ├── library_screen.dart
│   │       ├── library_provider.dart
│   │       └── widgets/
│   │           └── episode_tile.dart
│   ├── services/
│   │   ├── reddit/
│   │   │   ├── reddit_auth_service.dart   # OAuth 2.0 flow
│   │   │   ├── reddit_scraper.dart        # Post + comment fetching
│   │   │   └── models/
│   │   │       ├── reddit_post.dart
│   │   │       └── reddit_comment.dart
│   │   ├── llm/
│   │   │   ├── llm_service.dart           # Abstract interface
│   │   │   ├── os_llm_service.dart        # Platform channel impl (Phase 1)
│   │   │   ├── llm_prompt_builder.dart    # Prompt construction
│   │   │   └── models/
│   │   │       ├── transcript.dart
│   │   │       └── speaker.dart
│   │   └── tts/
│   │       ├── tts_service.dart           # Sherpa-ONNX wrapper
│   │       ├── voice_assignment.dart      # Speaker → voice mapping
│   │       └── audio_stitcher.dart        # Merge per-speaker audio segments
│   ├── models/
│   │   └── episode.dart                   # Drift schema
│   └── shared/
│       ├── widgets/
│       │   ├── threadcast_scaffold.dart
│       │   └── unsupported_device_screen.dart
│       └── theme/
│           └── app_theme.dart
├── assets/
│   └── tts_models/                    # Kokoro ONNX model + voicepack files
│       ├── model.onnx
│       ├── voices.bin
│       └── espeak-ng-data/            # G2P data directory
└── pubspec.yaml
```

---

## Reddit Data Acquisition

### Current Status: Approval Pending

Reddit introduced the **Responsible Builder Policy** in late 2025, ending self-service API registration. New OAuth app credentials now require manual review by Reddit before they are issued. The "create app" flow at `reddit.com/prefs/apps` will block new registrations until approval is granted.

**What this means for development:** Use the public `.json` endpoint (see below) during development. Apply for OAuth approval in parallel. Swap in the authenticated client when credentials arrive.

### Phase 1 (Development): Public JSON Endpoint

Append `.json` to any Reddit post URL to get structured data with no credentials:

```
https://www.reddit.com/r/AITAH/comments/{post_id}/{slug}.json?sort=top&limit=100&depth=3&raw_json=1
```

- No authentication required
- Rate limit: 10 requests per minute per IP
- Fully sufficient for development — a single post fetch is 1–2 requests
- Response structure is identical to the OAuth API, so no code changes are needed when switching

### Phase 2 (Production): OAuth 2.0

#### Applying for API Access

Submit a developer support request at:
`https://support.reddithelp.com/hc/en-us/requests/new`

Select **"Developer Support"** as the category. In your application describe Threadcast as:
- A personal, non-commercial mobile app
- Reads only public Reddit post content
- All processing is on-device — no data stored on servers, no resale, no AI training
- Use case: converts posts to audio for personal listening

Reddit's stated response time is approximately seven days.

#### Once Approved: Registering the App

1. Go to `https://www.reddit.com/prefs/apps`
2. Click **"create an app"**
3. Fill in:
   - **Name:** Threadcast
   - **App type:** `installed app`
   - **Redirect URI:** `threadcast://oauth/callback`
4. Copy the **client ID** (string under the app name, above "installed app")

### Storing Credentials

```dart
// lib/core/constants.dart
class AppConstants {
  // Set to empty string during development (triggers .json fallback mode)
  // Set to your approved client ID for production OAuth flow
  static const redditClientId = '';

  static const redditRedirectUri = 'threadcast://oauth/callback';
  static const redditUserAgent = 'ios:com.threadcast.app:v1.0.0 (by /u/YOUR_REDDIT_USERNAME)';
  static const redditScopes = 'read identity';
}
```

**Important:** The User-Agent string must follow Reddit's format exactly or requests will be rejected. Update the username field before going to production.

---

## Reddit OAuth 2.0 Implementation

Threadcast uses the **Authorization Code flow** with PKCE for installed mobile apps. There is no client secret for installed apps.

### Auth Flow

```
1. User taps "Connect Reddit Account" on onboarding screen
2. App generates PKCE code_verifier + code_challenge (S256)
3. App opens in-app WebView to Reddit authorization URL:
   https://www.reddit.com/api/v1/authorize.compact
     ?client_id={CLIENT_ID}
     &response_type=code
     &state={random_state}
     &redirect_uri=threadcast://oauth/callback
     &duration=permanent
     &scope=read identity
4. User approves in WebView
5. Reddit redirects to threadcast://oauth/callback?code={code}&state={state}
6. App intercepts the deep link, validates state param
7. App exchanges code for access_token + refresh_token via POST to:
   https://www.reddit.com/api/v1/access_token
8. Tokens stored in flutter_secure_storage
9. Access token refreshed automatically when expired (1 hour TTL)
```

### Rate Limits

- Free tier: 60 requests per minute per OAuth client ID (tracked over a rolling 10-minute window)
- All users of the app share this limit if using a single client ID
- Implement per-request delay of 100ms minimum between calls
- Respect `X-Ratelimit-Remaining` and `X-Ratelimit-Reset` response headers
- Back off exponentially on HTTP 429 responses

### Key API Endpoints

```
# Get post + comments (sorted by top)
GET https://oauth.reddit.com/r/{subreddit}/comments/{post_id}
  ?sort=top
  &limit=100        # top-level comments to fetch
  &depth=3          # nesting depth
  &raw_json=1       # prevents HTML encoding of special chars

# Get post by full URL (extract post_id from URL first)
# URL format: reddit.com/r/{sub}/comments/{post_id}/{slug}/
```

### RedditPost Model

```dart
class RedditPost {
  final String id;
  final String subreddit;
  final String title;
  final String selftext;       // Full post body
  final String authorName;
  final int score;
  final int numComments;
  final DateTime createdUtc;
  final List<RedditComment> comments;
}

class RedditComment {
  final String id;
  final String authorName;
  final String body;
  final int score;
  final int depth;             // 0 = top-level, 1 = reply, 2 = reply-to-reply
  final String? parentId;      // null if top-level
  final List<RedditComment> replies;
  final bool isOp;             // True if authorName == post author
}
```

### Scraping Logic

The scraper automatically selects the correct endpoint based on whether an OAuth client ID is configured. The response structure is identical between the two — no conditional parsing logic is needed.

```dart
// lib/services/reddit/reddit_scraper.dart

class RedditScraper {
  final Dio _dio;
  final RedditAuthService _auth;

  // True when a client ID is configured and an OAuth token is available
  bool get _isAuthenticated =>
      AppConstants.redditClientId.isNotEmpty && _auth.hasValidToken;

  /// Fetch and parse a single Reddit post.
  /// Automatically uses the OAuth API when credentials are available,
  /// falling back to the public .json endpoint for development.
  Future<RedditPost> fetchPost(String url) async {
    final postId = _extractPostId(url);
    final endpoint = _isAuthenticated
        ? 'https://oauth.reddit.com/comments/$postId'
        : 'https://www.reddit.com/comments/$postId.json';

    final headers = _isAuthenticated
        ? {'Authorization': 'Bearer ${await _auth.getAccessToken()}',
           'User-Agent': AppConstants.redditUserAgent}
        : {'User-Agent': AppConstants.redditUserAgent};

    final response = await _dio.get(
      endpoint,
      options: Options(headers: headers),
      queryParameters: {
        'sort': 'top',
        'limit': 100,
        'depth': 3,
        'raw_json': 1,
      },
    );

    // Both endpoints return a 2-element array: [postData, commentsData]
    return RedditPost.fromJson(response.data[0], response.data[1]);
  }

  /// Extract the post ID from any Reddit URL format.
  /// Handles: reddit.com/r/sub/comments/ID/slug/
  ///          redd.it/ID
  ///          old.reddit.com/r/sub/comments/ID/
  String _extractPostId(String url) {
    final uri = Uri.parse(url);
    final segments = uri.pathSegments;
    final commentsIdx = segments.indexOf('comments');
    if (commentsIdx != -1 && commentsIdx + 1 < segments.length) {
      return segments[commentsIdx + 1];
    }
    // redd.it/ID short links
    if (uri.host == 'redd.it' && segments.isNotEmpty) {
      return segments.first;
    }
    throw FormatException('Could not extract post ID from URL: $url');
  }
}
```

For multi-part episodes (multiple URLs), fetch each post independently and pass them all to the LLM prompt builder in order. The LLM stitches the narrative.

**Development note:** In `.json` fallback mode the onboarding OAuth flow should be skipped entirely. Check `AppConstants.redditClientId.isEmpty` in the router redirect guard and bypass the onboarding screen when no client ID is configured.

---

## LLM Abstraction Layer

The LLM service is abstracted behind an interface so Phase 2 (private model fallback) can be slotted in without touching the rest of the app.

```dart
// lib/services/llm/llm_service.dart
abstract class LlmService {
  /// Returns true if the OS model is available on this device/OS version.
  Future<bool> isAvailable();

  /// Generates a structured transcript from scraped Reddit data.
  /// Streams progress events for UI updates.
  Stream<LlmProgress> generateTranscript({
    required List<RedditPost> posts,   // Ordered list (multi-part support)
    required String episodeId,
  });
}

enum LlmProgress {
  analyzing,      // Personality + tone analysis phase
  writing,        // Transcript generation phase
  complete,
  failed,
}
```

### Platform Channel Contract

Channel name: `com.threadcast.app/llm`

**Methods (Dart → Native):**

| Method | Arguments | Returns |
|---|---|---|
| `isAvailable` | none | `bool` |
| `generateTranscript` | `{ "prompt": String }` | `String` (full transcript JSON) |
| `cancelGeneration` | none | `void` |

**Error codes:**

| Code | Meaning |
|---|---|
| `MODEL_UNAVAILABLE` | OS model not present on device |
| `CONTEXT_TOO_LONG` | Input exceeded model's context window |
| `GENERATION_FAILED` | Model returned error or empty output |
| `CANCELLED` | User cancelled mid-generation |

### Android: Gemini Nano via ML Kit GenAI Prompt API

**Dependency** (add to `android/app/build.gradle`):
```kotlin
implementation("com.google.mlkit:genai-prompt:1.0.0-beta1")
```

The correct API is the **ML Kit GenAI Prompt API** — NOT the Firebase/Vertex AI `GenerativeModel` constructor. Access is via `Generation.getClient()`. There is no `modelName` parameter, no `responseMimeType` parameter, and no static `isAvailable()` method.

**IMPORTANT**: AICore enforces that GenAI API inference is only permitted when the app is the **top foreground application**. Calling the API from a foreground service or background task will return `ErrorCode.BACKGROUND_USE_BLOCKED`. The app must be actively in the foreground during the entire generation pipeline.

```kotlin
// android/app/src/main/kotlin/com/threadcast/app/llm/GeminiNanoService.kt
import com.google.mlkit.genai.prompt.Generation
import com.google.mlkit.genai.prompt.GenerativeModel
import com.google.mlkit.genai.prompt.FeatureStatus
import com.google.mlkit.genai.prompt.DownloadCallback
import com.google.mlkit.genai.prompt.GenAiException
import com.google.mlkit.genai.prompt.generateContentRequest

class GeminiNanoService {

    // Obtain the shared on-device model client — no constructor params
    private val generativeModel: GenerativeModel = Generation.getClient()

    suspend fun checkStatus(): Int = generativeModel.checkStatus()

    suspend fun isAvailable(): Boolean =
        generativeModel.checkStatus() == FeatureStatus.AVAILABLE

    suspend fun downloadIfNeeded(
        onProgress: (Long) -> Unit = {},
        onComplete: () -> Unit = {},
        onFail: (GenAiException) -> Unit = {}
    ) {
        val status = generativeModel.checkStatus()
        if (status == FeatureStatus.DOWNLOADABLE) {
            generativeModel.download(object : DownloadCallback {
                override fun onDownloadCompleted() = onComplete()
                override fun onDownloadFail(e: GenAiException) = onFail(e)
                override fun onDownloadProgress(bytes: Long) = onProgress(bytes)
                override fun onDownloadStarted(bytes: Long) {}
            })
        }
    }

    suspend fun generateTranscript(prompt: String): String {
        // JSON output is enforced via the prompt itself — no responseMimeType param available
        val request = generateContentRequest {
            text(prompt)
            temperature = 0.7f
        }
        val response = generativeModel.generateContent(request)
        return response.text ?: throw GenAiException("MODEL_RETURNED_EMPTY")
    }

    fun release() = generativeModel.close()
}
```

```kotlin
// android/app/src/main/kotlin/com/threadcast/app/llm/LlmPlugin.kt

class LlmPlugin : FlutterPlugin, MethodCallHandler {

    private val service = GeminiNanoService()
    private var activeJob: Job? = null

    override fun onMethodCall(call: MethodCall, result: Result) {
        when (call.method) {
            "isAvailable" -> {
                CoroutineScope(Dispatchers.IO).launch {
                    val available = service.isAvailable()
                    withContext(Dispatchers.Main) { result.success(available) }
                }
            }
            "generateTranscript" -> {
                val prompt = call.argument<String>("prompt")!!
                activeJob = CoroutineScope(Dispatchers.IO).launch {
                    try {
                        val transcript = service.generateTranscript(prompt)
                        withContext(Dispatchers.Main) { result.success(transcript) }
                    } catch (e: Exception) {
                        val code = when {
                            e.message?.contains("BACKGROUND") == true -> "BACKGROUND_BLOCKED"
                            e.message?.contains("QUOTA") == true -> "QUOTA_EXCEEDED"
                            else -> "GENERATION_FAILED"
                        }
                        withContext(Dispatchers.Main) {
                            result.error(code, e.message, null)
                        }
                    }
                }
            }
            "cancelGeneration" -> {
                activeJob?.cancel()
                result.success(null)
            }
            else -> result.notImplemented()
        }
    }

    override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {
        service.release()
    }
}
```

### iOS: Foundation Models Framework

```swift
// ios/Runner/llm/FoundationModelService.swift
import FoundationModels

// NOTE: You must enable the "Foundation Models" entitlement in your Xcode project
// (Signing & Capabilities → + Capability → Foundation Models) or this import
// will compile but all calls will fail at runtime.

@available(iOS 18.0, *)
class FoundationModelService {

    func isAvailable() -> Bool {
        // Correct check: SystemLanguageModel.default.availability
        // LanguageModelSession.isSupported does NOT exist
        if case .available = SystemLanguageModel.default.availability {
            return true
        }
        return false
    }

    func unavailabilityReason() -> String? {
        if case .unavailable(let reason) = SystemLanguageModel.default.availability {
            return reason.localizedDescription
        }
        return nil
    }

    func generateTranscript(prompt: String) async throws -> String {
        guard isAvailable() else { throw LlmError.modelUnavailable }

        // Token limit: 4,096 combined input + output tokens.
        // If the prompt is long, the output will be truncated — ensure prompts
        // are concise and responses are within budget.
        let session = LanguageModelSession(
            instructions: "You are a podcast transcript writer. Always respond with valid JSON only. Do not include markdown code fences."
        )

        // prewarm() reduces first-token latency — call this when the user
        // navigates to the Create screen, before they tap Generate.
        try await session.prewarm()

        let response = try await session.respond(to: prompt)
        guard !response.content.isEmpty else {
            throw LlmError.emptyResponse
        }
        return response.content
    }
}

enum LlmError: Error {
    case emptyResponse
    case modelUnavailable
}
```

```swift
// ios/Runner/llm/LlmPlugin.swift
import Flutter
import FoundationModels

class LlmPlugin: NSObject, FlutterPlugin {

    static func register(with registrar: FlutterPluginRegistrar) {
        let channel = FlutterMethodChannel(
            name: "com.threadcast.app/llm",
            binaryMessenger: registrar.messenger()
        )
        let instance = LlmPlugin()
        registrar.addMethodCallDelegate(instance, channel: channel)
    }

    public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
        guard #available(iOS 18.0, *) else {
            result(FlutterError(code: "MODEL_UNAVAILABLE",
                               message: "iOS 18+ required", details: nil))
            return
        }

        let service = FoundationModelService()

        switch call.method {
        case "isAvailable":
            // Synchronous availability check — no async needed
            let available = service.isAvailable()
            DispatchQueue.main.async { result(available) }

        case "generateTranscript":
            guard let args = call.arguments as? [String: Any],
                  let prompt = args["prompt"] as? String else {
                result(FlutterError(code: "INVALID_ARGS", message: nil, details: nil))
                return
            }
            Task {
                do {
                    let transcript = try await service.generateTranscript(prompt: prompt)
                    DispatchQueue.main.async { result(transcript) }
                } catch {
                    DispatchQueue.main.async {
                        result(FlutterError(code: "GENERATION_FAILED",
                                           message: error.localizedDescription,
                                           details: nil))
                    }
                }
            }

        default:
            result(FlutterMethodNotImplemented)
        }
    }
}
```

---

## LLM Prompt Design

### Phase 1: Analysis Prompt

Send first. Extract tone, personalities, and decide which comments to include before generating the transcript.

```
You are analyzing a Reddit post to prepare it for conversion into a podcast.

## Input Data
<post>
SUBREDDIT: {subreddit}
TITLE: {title}
AUTHOR: {author}
BODY:
{post_body}
</post>

<comments>
{comments_json}
</comments>

## Your Task
Analyze the post and comments and return a JSON object with the following structure:

{
  "tone": "true_crime" | "talk_show" | "documentary" | "comedic" | "heartfelt",
  "tone_reasoning": "Brief explanation of why this tone fits",
  "speakers": [
    {
      "id": "op",
      "reddit_username": "{post_author}",
      "role": "main_speaker",
      "personality_notes": "How they speak — e.g. anxious and self-deprecating, uses run-on sentences, lots of qualifiers",
      "voice_gender": "male" | "female"
    },
    {
      "id": "speaker_2",
      "reddit_username": "commenter_username OR 'composite' if multiple merged",
      "role": "commenter",
      "personality_notes": "...",
      "voice_gender": "male" | "female",
      "merged_usernames": []   // List of usernames merged into this composite speaker, empty if single
    }
    // Include only commenters worth including in the podcast
    // Merge multiple commenters with the same energy/role into one speaker
    // Aim for 2-5 total speakers including OP
  ],
  "selected_comment_ids": ["id1", "id2"],   // Comment IDs selected for inclusion
  "episode_title": "A punchy, engaging episode title (not the Reddit post title verbatim)"
}

## Rules
- Tone must be inferred entirely from content. Do not default to any tone.
- Personality notes must be grounded in evidence from the actual writing — word choice, sentence structure, punctuation habits, emoji usage, etc.
- Only include comments that meaningfully advance the narrative or represent a significant reaction. Exclude low-effort comments, pure jokes with no substance, and comments that repeat a point already made.
- If OP has replied to commenters, prioritize those comment chains — the conversation dynamic between OP and commenters is more interesting than isolated takes.
- You may consolidate multiple commenters into a single composite speaker if they share a clear role (e.g. several people all saying "leave him", several people defending OP). Composite speakers should feel like one coherent voice.
```

### Phase 2: Transcript Generation Prompt

Send after analysis. Uses the output of Phase 1.

```
You are converting a Reddit post and selected comments into a podcast transcript.

## Analysis Results
<analysis>
{analysis_json_from_phase_1}
</analysis>

## Original Content
<post>
{post_body}
</post>

<selected_comments>
{selected_comments_json}
</selected_comments>

## Output Format
Return a JSON array of transcript segments:

[
  {
    "speaker_id": "op",
    "text": "The spoken dialogue text",
    "delivery": {
      "pace": "normal" | "slow" | "fast",
      "emotion": "calm" | "anxious" | "frustrated" | "upset" | "excited" | "sarcastic" | "empathetic",
      "pause_before_ms": 0,      // Milliseconds of silence before this segment
      "overlap_previous": false  // True if this speaker is interrupting/overlapping the previous
    }
  }
]

## Rules — CRITICAL

### Faithfulness
- The OP's post must be adapted into spoken dialogue. You may reorder sentences for clarity and remove Reddit-specific formatting (e.g. "Edit:", "UPDATE:", bullet points, "throwaway account"). You may NOT add facts, invent details, or change what happened.
- Each commenter's core sentiment and factual claims must be preserved. You may adapt their phrasing for natural speech. You may NOT change their position or invent reactions they did not express.
- Do not editorialize. Do not add a narrator. The OP is the main speaker telling their own story in their own voice.

### Structure
- Begin with the OP telling their story. This should feel natural and conversational, not like someone reading an essay.
- Interject commenters at dramatically appropriate moments — not just at the end. A commenter can react mid-story when the OP describes the key moment that their comment addresses.
- OP may respond to commenters (use OP's actual replies if they exist; otherwise do not invent OP responses).
- Use overlap_previous: true sparingly for moments of genuine interruption or strong reaction. When used, keep the overlapping segment short (one sentence max).
- For multi-part posts, the OP's narrative flows continuously across parts. Do not announce "Part 2" — just continue the story.

### Delivery
- Match the tone from analysis: {tone}
- Delivery cues must be grounded in the content of the line, not randomly assigned.
- Pauses (pause_before_ms) should be used at emotional beats, revelations, or natural paragraph breaks. Typical values: 300ms (light pause), 800ms (dramatic pause), 1500ms (scene break).

### Personality
- Each speaker's dialogue style must reflect their personality_notes from the analysis.
- OP's verbal tics, hedging language, and sentence rhythm should feel consistent throughout.
- Commenter voices must feel distinct from each other and from OP.

### Length
- Aim for a transcript that would produce 8-15 minutes of audio at natural speaking pace (~130 words per minute).
- For multi-part posts, scale up proportionally.
```

### Prompt Builder Implementation

```dart
// lib/services/llm/llm_prompt_builder.dart

class LlmPromptBuilder {

  String buildAnalysisPrompt(List<RedditPost> posts) {
    // For multi-part: concatenate posts, label each part
    final postSection = posts.length == 1
        ? _formatPost(posts.first)
        : posts.mapIndexed((i, p) => '## Part ${i + 1}\n${_formatPost(p)}').join('\n\n');

    final allComments = posts.expand((p) => p.comments).toList();
    final commentsJson = jsonEncode(allComments.map((c) => c.toJson()).toList());

    return ANALYSIS_PROMPT_TEMPLATE
        .replaceAll('{subreddit}', posts.first.subreddit)
        .replaceAll('{title}', posts.map((p) => p.title).join(' / '))
        .replaceAll('{author}', posts.first.authorName)
        .replaceAll('{post_body}', postSection)
        .replaceAll('{comments_json}', commentsJson);
  }

  String buildTranscriptPrompt({
    required List<RedditPost> posts,
    required Map<String, dynamic> analysis,
  }) {
    final selectedIds = (analysis['selected_comment_ids'] as List).cast<String>().toSet();
    final selectedComments = posts
        .expand((p) => _flattenComments(p.comments))
        .where((c) => selectedIds.contains(c.id))
        .toList();

    return TRANSCRIPT_PROMPT_TEMPLATE
        .replaceAll('{analysis_json_from_phase_1}', jsonEncode(analysis))
        .replaceAll('{post_body}', posts.map((p) => p.selftext).join('\n\n---\n\n'))
        .replaceAll('{selected_comments_json}', jsonEncode(selectedComments.map((c) => c.toJson()).toList()))
        .replaceAll('{tone}', analysis['tone'] as String);
  }

  List<RedditComment> _flattenComments(List<RedditComment> comments) {
    final result = <RedditComment>[];
    for (final c in comments) {
      result.add(c);
      result.addAll(_flattenComments(c.replies));
    }
    return result;
  }
}
```

---

## Transcript Data Models

```dart
// lib/services/llm/models/transcript.dart

class Transcript {
  final String episodeId;
  final String episodeTitle;
  final String tone;
  final List<Speaker> speakers;
  final List<TranscriptSegment> segments;
}

class Speaker {
  final String id;          // "op", "speaker_2", etc.
  final String? redditUsername;
  final String role;        // "main_speaker" | "commenter"
  final String personalityNotes;
  final String voiceGender; // "male" | "female"
  final String assignedVoice; // Set during voice assignment (Kokoro voice name)
}

class TranscriptSegment {
  final String speakerId;
  final String text;
  final DeliveryInstructions delivery;
  // Set after TTS synthesis:
  final String? audioFilePath;
  final Duration? audioDuration;
  final Duration? audioOffset; // Start time within final stitched audio
}

class DeliveryInstructions {
  final String pace;         // "normal" | "slow" | "fast"
  final String emotion;
  final int pauseBeforeMs;
  final bool overlapPrevious;
}
```

---

## TTS Pipeline

### Sherpa-ONNX + Kokoro Setup

Add to `pubspec.yaml`:
```yaml
dependencies:
  sherpa_onnx: ^1.x.x

flutter:
  assets:
    - assets/tts_models/
```

Model files to bundle (download from https://github.com/k2-fsa/sherpa-onnx/releases):
- `model.onnx` (~82MB)
- `voices.bin` (voice embeddings)
- `espeak-ng-data/` directory (G2P phoneme data — required for correct pronunciation)

**Important:** The espeak-ng-data directory must be extracted to the app's documents directory at first launch — it cannot be read directly from the Flutter assets bundle on all platforms. Implement a first-run extraction step.

### Voice Assignment

Kokoro provides named voices. Assign voices to speakers based on gender and role:

```dart
// lib/services/tts/voice_assignment.dart

class VoiceAssignment {

  // Kokoro built-in voices
  static const _maleVoices = ['am_adam', 'am_michael', 'am_fenrir'];
  static const _femaleVoices = ['af_sarah', 'af_bella', 'af_nicole'];

  static final _roleVoices = {
    'main_speaker_male': 'am_adam',
    'main_speaker_female': 'af_sarah',
  };

  static Map<String, String> assignVoices(List<Speaker> speakers) {
    final assignment = <String, String>{};
    final usedVoices = <String>{};

    for (final speaker in speakers) {
      if (speaker.role == 'main_speaker') {
        final voice = _roleVoices['main_speaker_${speaker.voiceGender}']!;
        assignment[speaker.id] = voice;
        usedVoices.add(voice);
      }
    }

    // Assign distinct voices to commenters
    final commenters = speakers.where((s) => s.role == 'commenter').toList();
    for (final speaker in commenters) {
      final pool = speaker.voiceGender == 'female'
          ? _femaleVoices
          : _maleVoices;

      final available = pool.where((v) => !usedVoices.contains(v)).toList();
      final voice = available.isNotEmpty ? available.first : pool.first;
      assignment[speaker.id] = voice;
      usedVoices.add(voice);
    }

    return assignment;
  }
}
```

### TTS Service

```dart
// lib/services/tts/tts_service.dart

class TtsService {

  late SherpaOnnxTts _tts;
  bool _initialized = false;

  Future<void> initialize() async {
    if (_initialized) return;

    final modelDir = await _getModelDir();
    _tts = SherpaOnnxTts(
      model: KokoroModel(
        model: '$modelDir/model.onnx',
        voices: '$modelDir/voices.bin',
        tokens: '$modelDir/tokens.txt',
        dataDir: '$modelDir/espeak-ng-data',
      ),
    );
    _initialized = true;
  }

  /// Synthesizes a single transcript segment.
  /// Returns path to the generated WAV file.
  Future<String> synthesizeSegment({
    required TranscriptSegment segment,
    required String voice,
    required String episodeId,
    required int segmentIndex,
  }) async {
    await initialize();

    // Map delivery pace to speaking speed
    final speed = switch (segment.delivery.pace) {
      'slow' => 0.85,
      'fast' => 1.15,
      _ => 1.0,
    };

    final outputPath = await _segmentPath(episodeId, segmentIndex);
    await _tts.generate(
      text: segment.text,
      voice: voice,
      speed: speed,
      outputFile: outputPath,
    );

    return outputPath;
  }

  Future<String> _getModelDir() async {
    final docsDir = await getApplicationDocumentsDirectory();
    return '${docsDir.path}/tts_models';
  }

  Future<String> _segmentPath(String episodeId, int index) async {
    final dir = await getTemporaryDirectory();
    return '${dir.path}/$episodeId/segment_$index.wav';
  }
}
```

### Audio Stitching

Merge per-segment WAV files into one final audio file, respecting pause and overlap timing:

```dart
// lib/services/tts/audio_stitcher.dart

class AudioStitcher {

  /// Takes ordered list of synthesized segments and produces a single WAV.
  /// Handles pause_before_ms and overlap_previous timing.
  Future<String> stitch({
    required List<TranscriptSegment> segments,
    required String episodeId,
  }) async {
    // Build ffmpeg filter_complex for precise timing
    // Each segment is placed at its calculated timestamp
    // Overlapping segments are mixed at their overlap point

    final timeline = _buildTimeline(segments);
    final outputPath = await _episodePath(episodeId);

    await FFmpegKit.execute(_buildFfmpegCommand(timeline, outputPath));    return outputPath;
  }

  List<_SegmentTiming> _buildTimeline(List<TranscriptSegment> segments) {
    final timing = <_SegmentTiming>[];
    var currentMs = 0;

    for (int i = 0; i < segments.length; i++) {
      final seg = segments[i];
      currentMs += seg.delivery.pauseBeforeMs;

      if (seg.delivery.overlapPrevious && timing.isNotEmpty) {
        // Start this segment 500ms before the previous one ends
        final prevEnd = timing.last.startMs + timing.last.durationMs;
        currentMs = prevEnd - 500;
      }

      timing.add(_SegmentTiming(
        filePath: seg.audioFilePath!,
        startMs: currentMs,
        durationMs: seg.audioDuration!.inMilliseconds,
      ));

      currentMs += seg.audioDuration!.inMilliseconds;
    }

    return timing;
  }

  String _buildFfmpegCommand(List<_SegmentTiming> timing, String outputPath) {
    final inputs = timing.map((t) => '-i "${t.filePath}"').join(' ');
    final delays = timing.mapIndexed((i, t) =>
        '[$i:a]adelay=${t.startMs}|${t.startMs}[s$i]').join(';');
    final mix = timing.mapIndexed((i, _) => '[s$i]').join('') +
        'amix=inputs=${timing.length}:normalize=0[out]';

    return '$inputs -filter_complex "$delays;$mix" -map "[out]" '
        '-acodec pcm_s16le -ar 24000 "$outputPath"';
  }

  Future<String> _episodePath(String episodeId) async {
    final dir = await getApplicationDocumentsDirectory();
    return '${dir.path}/episodes/$episodeId/audio.wav';
  }
}
```

### MP3 Encoding (for export)

```dart
// Convert final WAV to MP3 for export/sharing
Future<String> encodeToMp3(String wavPath, String episodeId) async {
  final mp3Path = wavPath.replaceAll('.wav', '.mp3');
  await FFmpegKit.execute(
    '-i "$wavPath" -codec:a libmp3lame -qscale:a 2 "$mp3Path"'
  );
  return mp3Path;
}
```

---

## Data Persistence

### Episode Schema (Drift)

```dart
// lib/models/episode.dart
// Full implementation written by migrate-dependencies.ps1.
// Key points for implementing tickets:

// Drift Episode is an IMMUTABLE data class.
// Never use cascade mutation (Episode()..field = value).
// All writes use EpisodesCompanion with Value() wrappers:

await db.upsertEpisode(EpisodesCompanion(
  episodeId:      Value(episodeId),
  title:          Value(analysis['episode_title'] as String),
  subreddit:      Value(posts.first.subreddit),
  sourceUrlsJson: Value(AppDatabase.encodeUrls(urls)),
  tone:           Value(analysis['tone'] as String),
  createdAt:      Value(DateTime.now()),
  durationSeconds: Value(durationSecs),
  audioWavPath:   Value(finalAudioPath),
  status:         Value(AppDatabase.encodeStatus(EpisodeStatus.complete)),
));

// Partial updates (e.g. after MP3 encoding):
await db.updateEpisodeFields(episode.id, audioMp3Path: mp3Path);

enum EpisodeStatus { pending, generating, complete, failed }
```

### Library Provider

```dart
// lib/features/library/library_provider.dart
// Full implementation written by migrate-dependencies.ps1.

class LibraryNotifier extends StateNotifier<AsyncValue<List<Episode>>> {
  final Ref _ref;

  LibraryNotifier(this._ref) : super(const AsyncValue.loading()) {
    reload();
  }

  Future<void> reload() async {
    final episodes = await _ref.read(databaseProvider).completedEpisodes();
    state = AsyncValue.data(episodes);
  }

  Future<void> deleteEpisode(int rowId, {String? wavPath, String? mp3Path}) async {
    if (wavPath != null) await File(wavPath).deleteIfExists();
    if (mp3Path != null) await File(mp3Path).deleteIfExists();
    await _ref.read(databaseProvider).deleteEpisodeById(rowId);
    await reload();
  }
}
```

---

## Audio Playback

```dart
// lib/features/player/player_provider.dart

@riverpod
class PlayerNotifier extends _$PlayerNotifier {

  late AudioPlayer _player;

  @override
  PlayerState build() {
    _player = AudioPlayer();
    ref.onDispose(() => _player.dispose());
    return const PlayerState.idle();
  }

  Future<void> loadEpisode(Episode episode) async {
    await _player.setFilePath(episode.audioWavPath!);
    state = PlayerState.ready(episode: episode, duration: _player.duration!);
  }

  Future<void> play() => _player.play();
  Future<void> pause() => _player.pause();
  Future<void> seekTo(Duration position) => _player.seek(position);
  Future<void> skipForward() =>
      _player.seek(_player.position + const Duration(seconds: 15));
  Future<void> skipBackward() =>
      _player.seek(_player.position - const Duration(seconds: 15));
}
```

---

## Export & Sharing

```dart
// lib/services/export_service.dart

class ExportService {

  /// Export episode as MP3 to user-accessible location.
  Future<String> exportMp3(Episode episode) async {
    if (episode.audioMp3Path != null &&
        await File(episode.audioMp3Path!).exists()) {
      return episode.audioMp3Path!;
    }
    // Encode on demand if not already done
    final mp3Path = await encodeToMp3(episode.audioWavPath!, episode.episodeId);
    // Persist the path using Drift partial update
    // (pass db from the calling context -- see databaseProvider)
    await db.updateEpisodeFields(episode.id, audioMp3Path: mp3Path);
    return mp3Path;
  }

  /// Share via OS share sheet (WhatsApp, iMessage, etc.)
  Future<void> shareEpisode(Episode episode, BuildContext context) async {
    final mp3Path = await exportMp3(episode);
    await Share.shareXFiles(
      [XFile(mp3Path, mimeType: 'audio/mpeg')],
      subject: episode.title,
    );
  }

  /// Save to Apple Podcasts (iOS) / system music library (Android)
  Future<void> saveToPodcastLibrary(Episode episode) async {
    final mp3Path = await exportMp3(episode);
    // iOS: Use AVAssetExportSession to hand off to Podcasts app
    // Android: Save to Music directory with MediaStore entry
    if (Platform.isIOS) {
      await _saveToIosPodcasts(mp3Path, episode.title);
    } else {
      await _saveToAndroidMediaStore(mp3Path, episode.title);
    }
  }
}
```

---

## Generation Pipeline (End-to-End Flow)

```dart
// lib/features/create/create_provider.dart

@riverpod
class CreateNotifier extends _$CreateNotifier {

  @override
  CreateState build() => const CreateState.idle();

  Future<void> generatePodcast(List<String> urls) async {
    final episodeId = const Uuid().v4();

    try {
      // 1. Check device compatibility
      state = const CreateState.checkingCompatibility();
      final llm = ref.read(llmServiceProvider);
      if (!await llm.isAvailable()) {
        state = const CreateState.unsupported();
        return;
      }

      // 2. Scrape Reddit posts
      state = const CreateState.scraping();
      final scraper = ref.read(redditScraperProvider);
      final posts = await Future.wait(urls.map(scraper.fetchPost));

      // 3. Phase 1: Analysis
      state = const CreateState.analyzing();
      final promptBuilder = ref.read(llmPromptBuilderProvider);
      final analysisPrompt = promptBuilder.buildAnalysisPrompt(posts);
      final analysisJson = await llm.generateRaw(analysisPrompt);
      final analysis = jsonDecode(analysisJson) as Map<String, dynamic>;

      // 4. Phase 2: Transcript generation
      state = const CreateState.generatingTranscript();
      final transcriptPrompt = promptBuilder.buildTranscriptPrompt(
        posts: posts,
        analysis: analysis,
      );
      final transcriptJson = await llm.generateRaw(transcriptPrompt);
      final segments = (jsonDecode(transcriptJson) as List)
          .map((s) => TranscriptSegment.fromJson(s as Map<String, dynamic>))
          .toList();

      // 5. Assign voices
      final speakers = (analysis['speakers'] as List)
          .map((s) => Speaker.fromJson(s as Map<String, dynamic>))
          .toList();
      final voiceMap = VoiceAssignment.assignVoices(speakers);

      // 6. TTS synthesis — STREAMING PIPELINE
      // Do NOT wait for the full transcript before starting TTS.
      // Synthesize each segment as soon as it is available and begin
      // stitching incrementally. This reduces perceived latency from
      // "wait for everything" to "audio starts appearing within seconds
      // of the first segment being synthesized."
      final tts = ref.read(ttsServiceProvider);
      final synthesized = <TranscriptSegment>[];
      final stitcher = ref.read(audioStitcherProvider);

      // Stream segments to TTS as they complete, updating state for UI
      for (int i = 0; i < segments.length; i++) {
        state = CreateState.synthesizing(
          progress: i / segments.length,
          currentSpeaker: segments[i].speakerId,
        );

        final seg = segments[i];
        final voice = voiceMap[seg.speakerId] ?? 'am_adam';
        final audioPath = await tts.synthesizeSegment(
          segment: seg,
          voice: voice,
          episodeId: episodeId,
          segmentIndex: i,
        );
        final duration = await _getAudioDuration(audioPath);
        synthesized.add(seg.copyWith(
          audioFilePath: audioPath,
          audioDuration: duration,
        ));

        // Once we have the first few segments synthesized, allow early
        // playback preview — the player can begin playing the partial
        // audio while the rest continues synthesizing in the background.
        // Emit a partial episode record to the player after segment 3.
        if (i == 2) {
          final partialPath = await stitcher.stitch(
            segments: synthesized,
            episodeId: '${episodeId}_partial',
          );
          state = CreateState.synthesizing(
            progress: i / segments.length,
            currentSpeaker: segments[i].speakerId,
            partialAudioPath: partialPath, // UI can offer "listen now" CTA
          );
        }
      }

      // 7. Final stitch of all segments
      state = const CreateState.stitching();
      final finalAudioPath = await stitcher.stitch(
        segments: synthesized,
        episodeId: episodeId,
      );

      // 8. Persist episode
      final episode = Episode()
        ..episodeId = episodeId
        ..title = analysis['episode_title'] as String
        ..subreddit = posts.first.subreddit
        ..sourceUrls = urls
        ..tone = analysis['tone'] as String
        ..createdAt = DateTime.now()
        ..durationSeconds = await _getAudioDurationSeconds(finalAudioPath)
        ..audioWavPath = finalAudioPath
        ..status = EpisodeStatus.complete;

      final db = ref.read(databaseProvider);
      await db.upsertEpisode(EpisodesCompanion(/* ... all fields ... */));

      state = CreateState.complete(episode: episode);

    } catch (e) {
      state = CreateState.failed(error: e.toString());
    }
  }
}
```

---

## Screen Inventory

### 1. Onboarding Screen
- Shown on first launch only
- Brief explanation of what Threadcast does
- "Connect Reddit Account" button → triggers OAuth flow
- Reddit WebView for authorization
- Handles deep link callback (`threadcast://oauth/callback`)
- On success: save tokens → navigate to Create screen

### 2. Create Screen
- URL input field with "Add" button
- URL chip list showing added URLs (ordered, deletable, draggable to reorder)
- "Generate Podcast" CTA button
- Generation progress overlay showing current stage:
  - Checking compatibility
  - Fetching posts (x/y)
  - Analyzing content
  - Writing transcript
  - Generating voices (x%)
  - Stitching audio
- Cancel button during generation
- On unsupported device: `UnsupportedDeviceScreen` widget (not a full page, inline)

### 3. Player Screen
- Episode title and subreddit badge
- Large play/pause button
- Seek bar with current time / total duration
- Skip ±15s buttons
- Speed selector (0.75x, 1x, 1.25x, 1.5x)
- Scrollable transcript view synced to playback (highlights current segment)
- Action row: Export MP3 | Share | Save to Podcasts | Delete

### 4. Library Screen
- Simple list of completed episodes (most recent first)
- Each tile: episode title, subreddit, duration, date
- Swipe-to-delete
- Tap → Player Screen
- Empty state with CTA to Create screen

### 5. Unsupported Device Screen (widget, not full screen)
- Shown inline in Create screen when OS model unavailable
- Message: "Threadcast requires Android 16 / iOS 18 or later to generate podcasts. Support for older devices is coming soon."
- No action button — informational only

---

## Navigation (go_router)

```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (_, __) => const CreateScreen()),
    GoRoute(path: '/onboarding', builder: (_, __) => const OnboardingScreen()),
    GoRoute(path: '/library', builder: (_, __) => const LibraryScreen()),
    GoRoute(
      path: '/player/:episodeId',
      builder: (_, state) => PlayerScreen(
        episodeId: state.pathParameters['episodeId']!,
      ),
    ),
  ],
  redirect: (context, state) async {
    final hasToken = await SecureStorageService.hasRedditToken();
    if (!hasToken && state.matchedLocation != '/onboarding') {
      return '/onboarding';
    }
    return null;
  },
);
```

---

## Background Processing Constraints

### iOS
- The LLM + TTS pipeline will take several minutes. iOS aggressively suspends background tasks.
- Keep the screen on during generation using `WakeLock` package.
- Show a persistent banner: "Keep Threadcast open while generating."
- **For background fallback**, use `BGContinuedProcessingTaskRequest` (iOS 18+), NOT `BGProcessingTask`. The standard `BGProcessingTask` is limited to ~5 minutes, which is insufficient for a full pipeline run. `BGContinuedProcessingTaskRequest` allows up to **one hour** of background runtime but requires an explicit user-initiated trigger — it cannot be scheduled opportunistically. Register it when the user taps "Generate" so the OS treats it as a user-requested long-running task.

```swift
// Register in AppDelegate or when generation begins
import BackgroundTasks

BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.threadcast.app.generate",
    using: nil
) { task in
    guard let task = task as? BGContinuedProcessingTask else { return }
    // Resume or complete pipeline here
    task.expirationHandler = {
        // Save partial state so we can resume
    }
}

// Submit when user taps Generate:
let request = BGContinuedProcessingTaskRequest(
    identifier: "com.threadcast.app.generate"
)
try BGTaskScheduler.shared.submit(request)
```

- Add `BGContinuedProcessingTask` to the `BGTaskSchedulerPermittedIdentifiers` array in Info.plist.
- The iOS simulator does NOT support Foundation Models. Test on a real device (iPhone 15 Pro or newer) from day one.

### Android
- **AICore explicitly blocks background inference.** The ML Kit GenAI Prompt API returns `ErrorCode.BACKGROUND_USE_BLOCKED` if called from any context where the app is not the active foreground application — this includes foreground services. Do NOT implement a ForegroundService for this purpose.
- Keep the app foregrounded during generation. Use `WakeLock` to prevent the screen from sleeping.
- Show a persistent in-app banner: "Keep Threadcast open while generating."
- Remove `FOREGROUND_SERVICE` and `FOREGROUND_SERVICE_DATA_SYNC` permissions from AndroidManifest.xml — they are not needed and imply a pattern that will not work.

---

## Error Handling Strategy

All errors in the generation pipeline should be:
1. Caught at the `CreateNotifier` level
2. Stored in the `Episode` record with `status = failed` and the error message
3. Displayed with a user-friendly message and a "Try again" button

| Error | User-facing message |
|---|---|
| `MODEL_UNAVAILABLE` | "Your device doesn't support on-device AI yet. Stay tuned." |
| `REDDIT_AUTH_EXPIRED` | "Reddit session expired. Please reconnect your account." |
| `REDDIT_RATE_LIMITED` | "Reddit is busy. Please wait a moment and try again." |
| `CONTEXT_TOO_LONG` | "This post is too long for the AI to process. Try a shorter post." |
| `GENERATION_FAILED` | "Something went wrong generating the transcript. Please try again." |
| `TTS_FAILED` | "Audio generation failed. Please try again." |
| `INVALID_URL` | "That doesn't look like a Reddit post URL." |

---

## pubspec.yaml (Key Dependencies)

```yaml
name: threadcast
description: Reddit posts converted to on-device AI podcasts
publish_to: none
version: 1.0.0+1

environment:
  sdk: '>=3.3.0 <4.0.0'
  flutter: '>=3.22.0'

dependencies:
  flutter:
    sdk: flutter

  # State management & navigation
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  go_router: ^14.0.0

  # Networking
  dio: ^5.4.0

  # Reddit OAuth
  flutter_web_auth_2: ^4.0.0    # WebView-based OAuth
  flutter_secure_storage: ^9.0.0

  # LLM (platform channels — no pub.dev package needed)
  # Implemented via MethodChannel in android/ and ios/ directories

  # TTS
  sherpa_onnx: ^1.9.0

  # Audio
  just_audio: ^0.9.0
  # Audio export — ffmpeg_kit_flutter was retired Jan 2025, use the community fork
  # ffmpeg_kit_flutter was retired Jan 2025
  # ffmpeg_kit_flutter_new is the original author's recommended community fork
  # IMPORTANT: First Gradle build fails on AAR download -- run flutter run TWICE
  ffmpeg_kit_flutter_new: ^2.0.0

  # Database -- drift replaces unmaintained isar (abandoned 2023)
  drift: ^2.20.0
  drift_flutter: ^0.2.0
  path_provider: ^2.1.0

  # Sharing & export
  share_plus: ^9.0.0

  # Utilities
  uuid: ^4.3.0
  collection: ^1.18.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
  drift_dev: ^2.20.0       # code generator for drift (run: dart run build_runner build)
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/tts_models/
```

---

## Android Manifest Additions

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>

<!-- Deep link for Reddit OAuth callback -->
<intent-filter android:autoVerify="true">
  <action android:name="android.intent.action.VIEW"/>
  <category android:name="android.intent.category.DEFAULT"/>
  <category android:name="android.intent.category.BROWSABLE"/>
  <data android:scheme="threadcast" android:host="oauth"/>
</intent-filter>
```

## iOS Info.plist Additions

```xml
<!-- ios/Runner/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>threadcast</string>
    </array>
  </dict>
</array>

<key>NSMicrophoneUsageDescription</key>
<string>Threadcast does not use the microphone.</string>

<key>UIBackgroundModes</key>
<array>
  <string>processing</string>
  <string>audio</string>
</array>

<key>BGTaskSchedulerPermittedIdentifiers</key>
<array>
  <string>com.threadcast.app.generate</string>
</array>
```

---

## Phase 2 Checklist (Hybrid Fallback — Do Not Implement Now)

When ready to support older devices:

- [ ] Implement `PrivateModelLlmService` that uses MLC-LLM or llama.cpp bindings
- [ ] Add model download manager (streaming download with progress, resume support)
- [ ] Add `isOsModelAvailable()` check and fallback routing in `llmServiceProvider`
- [ ] Replace "coming soon" screen with download prompt on unsupported devices
- [ ] Implement On-Demand Resources (ODR) for iOS model file delivery (App Store requirement for assets >100MB)
- [ ] Add speculative decoding with SmolLM2 135M as draft model for speed
- [ ] Add Android ADPF thermal callback registration for private model inference
- [ ] Minimum hardware requirements: 8GB RAM, Snapdragon 8 Gen 2 / Apple A15 or equivalent

---

## Known Constraints & Gotchas

1. **espeak-ng-data must be extracted at first launch.** It cannot be loaded from the Flutter asset bundle directly. Use `path_provider` to copy it to the app documents directory on first run, then pass that path to Sherpa-ONNX.

2. **Foundation Models requires the entitlement to be enabled in Xcode.** Go to Signing & Capabilities → + Capability → Foundation Models. Without this, the framework imports fine but all calls fail silently at runtime. This is easy to miss.

3. **Foundation Models does not work on most iOS Simulators.** Only simulators running on Apple silicon Macs with Apple Intelligence enabled will work. Test on a real device (iPhone 15 Pro or newer) for reliable results.

4. **Gemini Nano availability is device-dependent, not OS-version-dependent.** `Generation.getClient().checkStatus()` returns `FeatureStatus.UNAVAILABLE` on unsupported devices even on Android 16. The "coming soon" screen should appear for these users. Call `checkStatus()` before showing any generation UI — never assume the model is present.

5. **iOS Foundation Models combined token limit is 4,096 tokens (input + output).** This is tight for a two-phase pipeline on long posts. The analysis prompt output feeds into the transcript prompt as input, so both prompts plus both responses must collectively stay under budget. Monitor this carefully; very long Reddit posts may need to be trimmed before being sent to the model.

4. **Reddit tokens expire after 1 hour.** Implement silent token refresh using the stored refresh token before every API call. Do not prompt the user to re-login unless the refresh token itself has expired (permanent grants last indefinitely unless revoked).

5. **The LLM output must always be valid JSON.** Both OS models can hallucinate formatting errors. Wrap JSON parsing in a try/catch and retry the generation once on parse failure before surfacing an error to the user.

6. **Audio segment temp files accumulate during generation.** Clean up all per-segment WAV files from the temp directory after stitching is complete, whether successful or not.

7. **Kokoro voice blending** (mixing two voice embeddings) is possible and produces more variety — worth exploring for future commenter voice diversity, but not required for v1.

8. **Context window limits.** Gemini Nano has a context window of approximately 2,048 tokens in its mobile variant. Very long Reddit posts + comments may exceed this. Implement a token counter (approximate: 1 token ≈ 4 characters) and truncate comments from the bottom of the list if the total exceeds 1,800 tokens, preserving the post body and OP replies at all costs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/A-Baji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/A-Baji)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
