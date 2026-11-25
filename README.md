# Video Moderation Editor Starter Kit for iOS

Video moderation made easy. Check every clip, image and text on the timeline against a moderation service and see where content may violate your guidelines. Built with [CE.SDK](https://img.ly/creative-sdk) by [IMG.LY](https://img.ly).

<p>
  <a href="https://img.ly/docs/cesdk/ios/quickstart/">Documentation</a>
</p>

## Getting Started

### Prerequisites

- [Xcode](https://developer.apple.com/xcode/)
- Swift 6+
- iOS 16+ deployment target
- An IMG.LY API key (`sk_…`) from the [IMG.LY Dashboard](https://img.ly/dashboard) — the moderation check cannot run without one

### Clone the Repository

```bash
git clone https://github.com/imgly/starterkit-video-moderation-ios.git
cd starterkit-video-moderation-ios
```

### Open in Xcode

```bash
open StarterKit-VideoModerationEditor.xcodeproj
```

Xcode will resolve the Swift Package dependencies automatically. Select an iOS Simulator or device and press **Run** (Cmd+R).

## Configuration

Add your CE.SDK license key and your IMG.LY API key in `StarterKit-VideoModerationEditor/Secrets.swift`:

```swift
let secrets = Secrets(
  // ...
  licenseKey: "your-license-key",
  gatewayApiKey: "sk_…"
)
```

Without a license key, the editor runs in evaluation mode with a watermark. The API key must have access to `imgly/detection` and a credit balance.

## How Video Moderation Works

Tapping the shield in the navigation bar opens the moderation panel. The user starts a check with the button at the bottom of the sheet. A run then:

1. **Finds every video clip, image, text and caption** in the current scene. A clip whose file cannot load is listed as unchecked.
2. **Sends each image and text**, up to four at a time, then **each video source file once**, one at a time, to the `imgly/detection` model. A clip or a photo from the photo roll or the camera is uploaded first. For a video, the model scores a frame every two seconds.
3. **Keeps only the frames that play.** Frames that the user trimmed away do not flag a clip, and a split clip is checked once.
4. **Displays results grouped by block**, with each flagged category, its severity (certain, likely) and, for a clip, the flagged times, for example `0:04–0:10`.
5. **Selects a block** when the user taps **Select**, and moves the playhead to the first flagged moment of a clip, or to the start of an image or a text.

Export is gated on the check. It goes ahead without asking only when the last run checked every block of the unchanged video and flagged nothing. Otherwise an alert says why, and offers to moderate now, to export anyway, or to cancel.

### Content Categories

Each video frame and image is scored for Weapons, Alcohol, Drugs, Nudity, Offensive symbols, and Gore. Text is scored for Toxic, Insulting, Discriminatory, Violent, and Sexual Language, and for Self-Harm. A category the model does not score is not reported.

### Cost

The gateway charges credits for each sampled frame, so a longer clip costs more than a short one. The service keeps the result of each source file, image and text for the editor session, so checking again after a trim or a move costs nothing.

### Configuring the Moderation Service

The moderation backend sits behind the `VideoModerationService` protocol. The bundled `GatewayVideoModerationService` calls the IMG.LY gateway. To use your own service, implement `VideoModerationService` and pass it to `VideoModerationStore` in `VideoModerationEditorStarterKit.swift`:

```swift
@StateObject private var moderation = VideoModerationStore(
  service: MyVideoModerationService(),
)
```

Set `uploadsLocalFiles: false` on `GatewayVideoModerationService` to keep the user's own videos and photos on the device. Those blocks are then reported as unchecked, so the export asks first.

## License

This project is licensed under the terms in [LICENSE](LICENSE).
