## pierrediffsswift

> This guide explains how to integrate PierreDiffsSwift's full feature set into a consumer app, with specific focus on the inline annotation (code review) system.

# PierreDiffsSwift Integration Guide

This guide explains how to integrate PierreDiffsSwift's full feature set into a consumer app, with specific focus on the inline annotation (code review) system.

## Package Dependency

```swift
// Package.swift
.package(url: "https://github.com/jamesrochabrun/PierreDiffsSwift", from: "1.2.0")
```

## Core View: PierreDiffView

```swift
PierreDiffView(
    oldContent: String,                                          // Original file content
    newContent: String,                                          // Updated file content
    fileName: String,                                            // For syntax highlighting
    diffStyle: Binding<DiffStyle>,                               // .split or .unified
    overflowMode: Binding<OverflowMode>,                         // .scroll or .wrap
    annotations: [DiffAnnotation]? = nil,                        // Inline comments
    onLineClick: ((Int, String) -> Void)? = nil,                 // (lineNumber, side)
    onLineClickWithPosition: ((LineClickPosition, CGPoint) -> Void)? = nil,  // With screen position
    onLineSelectionChange: ((LineSelectionRange) -> Void)? = nil, // Multi-line drag
    onAnnotationClick: ((String, String, Int, CGPoint) -> Void)? = nil,      // (id, side, lineNumber, localPoint)
    onAnnotationDelete: ((String, String, Int) -> Void)? = nil,              // (id, side, lineNumber)
    onExpandRequest: (() -> Void)? = nil,
    onReady: (() -> Void)? = nil
)
```

All callbacks are optional. The view works with just content + fileName + bindings.

## Feature 1: Line Click with Position

Use `onLineClickWithPosition` to get a `CGPoint` for positioning SwiftUI overlays (editors, popovers, menus) relative to the WebView.

```swift
onLineClickWithPosition: { position, localPoint in
    // position.lineNumber: Int — the clicked line
    // position.side: String — "left", "right", or "unified"
    // localPoint: CGPoint — WebView-local coordinates, top-left origin (matches SwiftUI)
    editorOverlay.show(at: localPoint, lineNumber: position.lineNumber, side: position.side)
}
```

The `CGPoint` is computed from `NSEvent.mouseLocation` converted to WebView-local coordinates. It is accurate regardless of scroll position.

## Feature 2: Multi-Line Selection

Use `onLineSelectionChange` to detect when users drag across line numbers.

```swift
onLineSelectionChange: { selection in
    // selection.startLine: Int
    // selection.endLine: Int
    // selection.side: String — "left", "right", or "unified"
    editorOverlay.show(lineRange: selection.startLine...selection.endLine, side: selection.side)
}
```

## Feature 3: Inline Annotations (Code Review)

Annotations are inline comment blocks rendered inside the WebView below the target line. They support click-to-edit and click-to-delete interactions.

### Data Model

```swift
DiffAnnotation(
    side: .additions,        // .additions (right/new) or .deletions (left/old)
    lineNumber: 42,          // Line number to attach to
    metadata: AnnotationMetadata(
        id: "unique-id",     // Used for identification in callbacks (defaults to UUID)
        author: "You",       // Display name shown in annotation header
        body: "Comment text" // The comment body
    )
)
```

### Reactive Data Flow

PierreDiffsSwift is **stateless** regarding annotations. The consumer owns the comment state and passes `[DiffAnnotation]` as a prop. SwiftUI reactivity handles the rest.

```
Consumer State (@Observable)
    ↓ converts comments → [DiffAnnotation]
PierreDiffView(annotations: [...])
    ↓ updateNSView detects change
coordinator.setAnnotations([...])
    ↓ JS bridge
@pierre/diffs renders inline DOM (no full re-render)
```

When annotations change (add/edit/remove), just update your state. The annotations array changes, SwiftUI re-evaluates, and `updateNSView` calls `setAnnotations()` automatically.

### Callbacks

**`onAnnotationClick: (id, side, lineNumber, localPoint)`**
User clicked the annotation body. Use this to open an edit overlay at `localPoint`.

**`onAnnotationDelete: (id, side, lineNumber)`**
User clicked the X button on the annotation. Use this to remove the comment from your state. The annotation disappears reactively when the `annotations` array updates.

## Integration Pattern: Inline Code Review

This is the recommended pattern for building a PR-style inline review system. This is how a consumer like AgentHub should integrate.

### Step 1: Comment State Manager

Create an `@Observable` class that stores review comments and converts them to `[DiffAnnotation]`:

```swift
@Observable @MainActor
class ReviewCommentsState {
    // Store comments keyed by location for deduplication
    var comments: [String: ReviewComment] = [:]

    var orderedComments: [ReviewComment] {
        comments.values.sorted { $0.timestamp < $1.timestamp }
    }

    var commentsByFile: [String: [ReviewComment]] {
        Dictionary(grouping: orderedComments, by: \.filePath)
    }

    // Convert stored comments to DiffAnnotation array for a specific file
    func annotations(for filePath: String) -> [DiffAnnotation] {
        commentsByFile[filePath]?.map { comment in
            DiffAnnotation(
                side: comment.side == "left" ? .deletions : .additions,
                lineNumber: comment.lineNumber,
                metadata: AnnotationMetadata(
                    id: comment.id.uuidString,
                    author: "You",
                    body: comment.text
                )
            )
        } ?? []
    }

    // Lookup by annotation ID (for edit-on-click)
    func comment(byAnnotationId id: String) -> ReviewComment? {
        orderedComments.first { $0.id.uuidString == id }
    }

    func addComment(filePath: String, lineNumber: Int, side: String, lineContent: String, text: String) {
        let key = "\(filePath):\(lineNumber):\(side)"
        comments[key] = ReviewComment(
            id: comments[key]?.id ?? UUID(),
            timestamp: Date(),
            filePath: filePath,
            lineNumber: lineNumber,
            side: side,
            lineContent: lineContent,
            text: text
        )
    }

    func removeComment(byAnnotationId id: String) {
        if let key = comments.first(where: { $0.value.id.uuidString == id })?.key {
            comments.removeValue(forKey: key)
        }
    }

    func clearAll() {
        comments.removeAll()
    }

    // Aggregate all comments into a single prompt for sending to an LLM
    func generatePrompt() -> String {
        var prompt = "I have the following review comments on the code changes:\n"
        for (filePath, fileComments) in commentsByFile {
            prompt += "\n## \(filePath)\n"
            for comment in fileComments {
                let sideLabel = comment.side == "left" ? "old" : "new"
                prompt += "\n**Line \(comment.lineNumber)** (\(sideLabel)):\n"
                prompt += "```\n\(comment.lineContent)\n```\n"
                prompt += "Comment: \(comment.text)\n"
            }
        }
        prompt += "\nPlease address these review comments."
        return prompt
    }
}
```

### Step 2: Wire PierreDiffView

```swift
struct DiffReviewView: View {
    let oldContent: String
    let newContent: String
    let filePath: String
    @State private var diffStyle: DiffStyle = .split
    @State private var overflowMode: OverflowMode = .scroll
    @State private var commentsState = ReviewCommentsState()
    @State private var editorState = InlineEditorState()

    var body: some View {
        GeometryReader { geometry in
            ZStack {
                PierreDiffView(
                    oldContent: oldContent,
                    newContent: newContent,
                    fileName: filePath,
                    diffStyle: $diffStyle,
                    overflowMode: $overflowMode,
                    // Pass annotations derived from comment state
                    annotations: commentsState.annotations(for: filePath),
                    // Open editor for new comment
                    onLineClickWithPosition: { position, localPoint in
                        let fileContent = position.side == "left" ? oldContent : newContent
                        editorState.show(
                            at: localPoint,
                            lineNumber: position.lineNumber,
                            side: position.side,
                            fileName: filePath,
                            lineContent: extractLine(from: fileContent, lineNumber: position.lineNumber)
                        )
                    },
                    // Open editor for editing existing comment
                    onAnnotationClick: { id, side, lineNumber, localPoint in
                        if let comment = commentsState.comment(byAnnotationId: id) {
                            editorState.showEdit(at: localPoint, comment: comment)
                        }
                    },
                    // Remove comment — annotation disappears reactively
                    onAnnotationDelete: { id, side, lineNumber in
                        commentsState.removeComment(byAnnotationId: id)
                    }
                )

                // Your floating editor overlay positioned at editorState.anchorPoint
                if editorState.isShowing {
                    InlineEditorOverlay(
                        state: editorState,
                        onSubmit: { text in
                            commentsState.addComment(
                                filePath: filePath,
                                lineNumber: editorState.lineNumber,
                                side: editorState.side,
                                lineContent: editorState.lineContent,
                                text: text
                            )
                            editorState.dismiss()
                            // Comment state updates → annotations array updates
                            // → PierreDiffView re-renders annotation inline
                        }
                    )
                }
            }
        }
    }
}
```

### Step 3: Send All Comments

When the user is done reviewing, aggregate all comments and send:

```swift
Button("Send \(commentsState.orderedComments.count) Comments") {
    let prompt = commentsState.generatePrompt()
    // Send prompt to your LLM / terminal / API
    sendToLLM(prompt)
    commentsState.clearAll()  // Clears state → all annotations disappear
}
```

## How Annotations Work Under the Hood

1. `PierreDiffView.updateNSView` compares `coordinator.lastAnnotations` with the new `annotations` array
2. If changed, calls `coordinator.setAnnotations(annotations)` which base64-encodes the array and calls `window.pierreBridge.setAnnotations(data)` in JS
3. The JS bridge calls `currentDiffInstance.setLineAnnotations(annotations)` which is the `@pierre/diffs` API for dynamic annotation updates (no full re-render)
4. For each annotation, `@pierre/diffs` calls the `renderAnnotation(annotation)` callback registered during `FileDiff` construction
5. `createAnnotationDOM(annotation)` in `diff-entry.js` creates an HTML element with:
   - Author header
   - Comment body
   - Delete (X) button (visible on hover)
   - Click handler → posts `annotationClicked` to Swift
   - Delete handler → posts `annotationDeleteRequested` to Swift (with `stopPropagation`)

## Position Coordinate System

All `CGPoint` values from callbacks (`onLineClickWithPosition`, `onAnnotationClick`) use:
- **Origin**: Top-left of the WKWebView
- **Coordinate system**: Matches SwiftUI (Y increases downward)
- **Source**: `NSEvent.mouseLocation` converted from screen → window → WebView local coordinates

This means you can directly use the point for SwiftUI overlay positioning within a `GeometryReader` or `ZStack` that contains the `PierreDiffView`.

---
> Source: [jamesrochabrun/PierreDiffsSwift](https://github.com/jamesrochabrun/PierreDiffsSwift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
