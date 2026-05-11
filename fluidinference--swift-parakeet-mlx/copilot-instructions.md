## testing-and-examples

> This rule outlines testing patterns and example usage for the ParakeetMLX Swift library.

# Testing and Examples Guide

This rule outlines testing patterns and example usage for the ParakeetMLX Swift library.

## Test Structure

### Main Test File
**[ParakeetMLXTests.swift](mdc:Tests/ParakeetMLXTests/ParakeetMLXTests.swift)** contains unit tests covering:

- Configuration encoding/decoding (`testConfigurationCoding`)
- Data structure creation and validation (`testAlignedTokenCreation`, `testAlignedSentenceCreation`)
- Audio processing window functions (`testAudioProcessingWindowFunctions`)
- Error handling and types (`testErrorTypes`)
- MLXArray extensions (`testMLXArrayExtensions`)

### Test Patterns
```swift
// Configuration testing
let config = PreprocessConfig(sampleRate: 16000, ...)
let data = try JSONEncoder().encode(config)
let decoded = try JSONDecoder().decode(PreprocessConfig.self, from: data)
XCTAssertEqual(config.sampleRate, decoded.sampleRate)

// MLX type testing with explicit type specification
let stdResult = array.std()
XCTAssertGreaterThan(stdResult.item(Float.self), 0.0)
```

## Usage Examples

### Basic Example File
**[BasicTranscription.swift](mdc:Examples/BasicTranscription.swift)** demonstrates:

1. **Model Loading**
```swift
let model = try loadParakeetModel(
    from: "nvidia/parakeet-tdt_ctc-1.1b",
    dtype: .bfloat16
)
```

2. **Basic Transcription**
```swift
let audioData = MLXArray(samples)
let result = try model.transcribe(audioData: audioData)
print("Transcription: \(result.text)")
```

3. **Chunked Processing**
```swift
let chunkedResult = try model.transcribe(
    audioData: longAudioData,
    chunkDuration: 3.0,
    overlapDuration: 1.0,
    chunkCallback: { current, total in
        let progress = (current / total) * 100
        print("Progress: \(String(format: "%.1f", progress))%")
    }
)
```

4. **Streaming Transcription**
```swift
let streamingSession = model.transcribeStream(
    contextSize: (128, 128),
    depth: 1
)

for chunkIndex in 0..<numChunks {
    let chunk = Array(samples[startIdx..<endIdx])
    try streamingSession.addAudio(MLXArray(chunk))
    let partialResult = streamingSession.result
    print("Streaming chunk \(chunkIndex + 1): \(partialResult.text)")
}
```

## CLI Usage Examples

### Command Line Interface
**[main.swift](mdc:Sources/ParakeetCLI/main.swift)** provides:

```bash
# Basic usage
swift run ParakeetCLI --input audio.wav

# Advanced usage with options
swift run ParakeetCLI \
    --input long_audio.wav \
    --output transcription.txt \
    --model nvidia/parakeet-tdt_ctc-1.1b \
    --chunk-duration 30 \
    --verbose \
    --timing
```

### Audio Loading Pattern
```swift
func loadAudioFile(_ path: String) throws -> MLXArray {
    let url = URL(fileURLWithPath: path)
    let audioFile = try AVAudioFile(forReading: url)
    
    // Target format: 16kHz mono
    guard let targetFormat = AVAudioFormat(
        commonFormat: .pcmFormatFloat32,
        sampleRate: 16000,
        channels: 1,
        interleaved: false
    ) else {
        throw AudioError.formatError("Could not create target format")
    }
    
    // Convert and return MLXArray
    let samples = Array(UnsafeBufferPointer(start: floatData, count: frameLength))
    return MLXArray(samples)
}
```

## Integration Patterns

### iOS App Integration
Example SwiftUI integration pattern:
```swift
class TranscriptionService: ObservableObject {
    @Published var transcription = ""
    @Published var isLoading = false
    
    private var model: ParakeetTDT?
    
    func loadModel() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            model = try loadParakeetModel(from: "nvidia/parakeet-tdt_ctc-1.1b")
        } catch {
            print("Failed to load model: \(error)")
        }
    }
}
```

## Performance Testing

### Benchmarking Pattern
```swift
let startTime = Date()
let result = try model.transcribe(audioData: audioData)
let endTime = Date()

let processingTime = endTime.timeIntervalSince(startTime)
let audioLength = Float(audioData.shape[0]) / 16000.0
let realTimeFactor = processingTime / Double(audioLength)

print("Real-time factor: \(String(format: "%.2f", realTimeFactor))x")
```

## Error Handling

### Common Error Types
```swift
enum ParakeetError: Error, LocalizedError {
    case invalidModelType(String)
    case unsupportedDecoding(String)
    case audioProcessingError(String)
    case modelLoadingError(String)
}
```

### Error Testing Pattern
```swift
let invalidModelError = ParakeetError.invalidModelType("Test error")
XCTAssertEqual(invalidModelError.errorDescription, "Invalid model type: Test error")
```

---
> Source: [FluidInference/swift-parakeet-mlx](https://github.com/FluidInference/swift-parakeet-mlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
