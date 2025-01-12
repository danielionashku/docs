# How to use Mixpanel iOS Session Replay

# Overview

Welcome to the Mixpanel Session Replay iOS SDK(Closed Alpha)!

Mixpanel Session Replay helps you quickly understand your customers and make better product decisions by combining quantitative and qualitative user insights.

# Quick Start Guide

## Prerequisite

You are already a Mixpanel customer and have the Mixpanel iOS SDK installed. If not, please follow this [doc](https://docs.mixpanel.com/docs/what-is-mixpanel) to get started.

## Install

You can integrate the Mixpanel iOS Session Replay SDK into your iOS project using Swift Package Manager (SPM).

### Open Your Xcode Project

Open your existing Xcode project where you want to integrate the Mixpanel iOS Session Replay SDK.

### Add Mixpanel Session Replay Package

Download and Unzip the below zip to your local drive

v0.2.0 - Updated 08/01/2024

[MixpanelSessionReplay.xcframework.zip](/downloads/MixpanelSessionReplay_v020.xcframework.zip)


- In Xcode, navigate to File > Add Package....

![Add Local Package](/ios_sr_add_local.png)

- Pick the Mixpanel Session Replay package you just saved, and make sure to select a target.

![Choose Package](/ios_sr_choose_package.png)

- Click Add Package.

## Initialize

You should have the main Mixpanel SDK installed, if not, please refer to [Prerequisite](https://www.notion.so/How-to-use-Mixpanel-iOS-Session-Replay-6215958920b349a49cfcafd651de3f44?pvs=21) Add the initialization code at the start of your app's lifecycle.

SwiftUI

```swift
import Mixpanel
import MixpanelSessionReplay

struct SessionReplayDemoApp: App {
    @State private var isActive = true
    @Environment(\\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ...
        }
        .onChange(of: scenePhase) { _, newPhase in
            if newPhase == .active {
                let sessionReplayInstance = SessionReplay.initialize(token: Mixpanel.mainInstance().apiToken, distinctId: Mixpanel.mainInstance().distinctId)
                // Uncomment if you want to record the entire session
                // sessionReplayInstance = SessionReplay.getInstance().startRecording()
            }
        }
    }
}

```

UIKit

```swift
import Foundation
import UIKit

import Mixpanel
import MixpanelSessionReplay

class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {
        Mixpanel.initialize(token: token, trackAutomaticEvents: true)
        Mixpanel.mainInstance().loggingEnabled = true

        return true
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        let sessionReplayInstance = SessionReplay.initialize(token: Mixpanel.mainInstance().apiToken, distinctId: Mixpanel.mainInstance().distinctId)
        // Uncomment if you want to record the entire session
        // sessionReplayInstance.startRecording()
    }

}

```

## Configuration Options

### Session Replay Config

Upon initialization you can provide a SessionReplayConfig object to specify the configuration. Currently, there are only two config options:

`wifiOnly` - when set to `true` replay events will only be flushed to the server when the device has a WiFi connection, if there is no WiFi, flushes are skipped and the events remain in the in-memory queue until WiFi is restored (or until the queue reaches its limit and the oldest events are evicted to make room for newer events). When set to `false` replay events will be flushed with any network connection, including cellular. - Default: `true`

`recordSessionsPercent` - this is a value between 0.0 and 100.0 that controls the sampling rate for recording session replays, at 0.0 no sessions will be recorded, at 100.0 all sessions will be recorded. - Default: 0.0

`recordIdleTimeout` - The time interval (in seconds) after which the session is considered idle and a new session should start. This value is used to determine when the user has been inactive for too long. If the user does not interact with the session within this time frame, the current session will be ended and a new session will be started. Example: If the `recordIdleTimeout` is set to 30 seconds, a new session will start if there is no user interaction for 30 seconds.

`recordMaxLength` - The maximum duration (in seconds) that a single session can last. This value is used to limit the length of a session to ensure it does not exceed a certain duration. Once the session reaches this maximum duration, it will be ended and a new session will start. Example: If the `recordMaxLength` is set to 3600 seconds (1 hour), a new session will start after 1 hour of continuous session activity.

`maskAllText` - which allows users to determine whether all text elements in the session replay should be masked. By default, this option is enabled (`true`)

`maskAllImages` - to control whether all images in the session replay should be masked. This option is also enabled by default (`true`)

### Logging

Developers can enable or disable logging with the `loggingEnabled` property of the `SessionReplay` object.

### Example

```swift
let token = Mixpanel.mainInstance().apiToken
let distinctId = Mixpanel.mainInstance().distinctId
let config = SessionReplayConfig(wifiOnly: false, recordSessionsPercent: 50.0)
let sessionReplayInstance = SessionReplay.initialize(token: token, distinctId: distinctId, config: config)
sessionReplayInstance.loggingEnabled = true

```

## Enable Session Replay

To start the session replay recording, you only need to call this once so we recommend calling this at the beginning of your app’s lifecycle:

```swift
SessionReplay.getInstance().startRecording()

```

To stop the session replay recording:

```swift
SessionReplay.getInstance().stopRecording()
```

## Privacy Settings

By default, Mixpanel will hide UILabel and UITextField to exclude more sensitive views from recording, to mark any view as sensitive:

```swift
// SwiftUI
Image("family photo")
	.replaySensitive()

// UIKit
SessionReplay.getInstance().addSensitiveView(mySensiveView)

```

## Using Session Replay

Please refer to [Using Session Replay](https://docs.mixpanel.com/docs/features/session-replay#using-session-replay)

# FAQ

**How does Session Replay work in iOS?**

Session Replay observes user interactions within your app, capturing UI hierarchy changes and storing them as images, which are then sent to Mixpanel. Mixpanel reconstructs these images, applying recorded events as an end-user completes them. Within Mixpanel’s platform, you can view a reconstruction of your end-user’s screen as they navigate your app. However, Session Replay is not a literal video recording of your end-user’s screen; end-user actions are not video-recorded.

**What is the expected impact on my app performance?**

There is no impact on your app’s performance when there are no user interactions or nothing changes on the screen. When there are user interactions, expect approximately 1% to 3% more CPU usage and around 1MB more memory consumption. There is no impact on disk I/O because Session Replay does not write anything to your disk. In our own testing, the overhead is unnoticeable, however if you experience any performance degradations after installing Session Replay, please reach out to us.

**What is the bandwidth impact and will it cause users to incur additional data charges?**

The bandwidth impact of Session Replay depends on the setting of the `wifiOnly` parameter.

When `wifiOnly` is set to `true` (default setting), replay events are only flushed to the server when the device has a WiFi connection. If there is no WiFi, flushes are skipped, and the events remain in the in-memory queue until WiFi is restored. This ensures no additional cellular data is used, preventing users from incurring additional data charges.

When `wifiOnly` is set to `false`, replay events are flushed with any available network connection, including cellular. In this case, the amount of cellular data consumed depends on the intensity of user interactions and the typical session length of your app. Users may incur additional data charges if large amounts of data are transmitted over cellular connections.

**How does Session Replay for mobile work if my app is offline?**

Session Replay for mobile currently doesn’t work in offline mode.

**Am I able to sample our session replay collection rate?**

Yes, you can configure the percentage of total replays that our SDK will capture as below.

```jsx
let config = SessionReplayConfig(wifiOnly: false, recordSessionsPercent: 50.0)
let sessionReplayInstance = SessionReplay.initialize(token: token, distinctId: distinctId, config: config)
```

This out-of-the-box sampling method is random sampling: your SDK will decide randomly whether the currently SDK instance load falls into the sample or not. We recommend starting at 1% and increasing from there. Please note: if you expect low traffic, you can increase the starting sampling percentage. It is a good idea to control this with your own API, so you can change it on the fly without redeploying your app. While you're testing, we recommend that you set it to 100 and this ensure every user session will be sent to Mixpanel. Once testing is done, we recommend lowering this value in production.

Does it work in SwiftUI/UIKit apps?

Yes, please refer to this [section](https://www.notion.so/How-to-use-Mixpanel-iOS-Session-Replay-6215958920b349a49cfcafd651de3f44?pvs=21)

Does it support Obj-C based app?

Yes, Objective-C and Swift are fully interoperable.

**Can I prevent Session Replay from recording sensitive content?**

If your app is UIKit-based, all `UITextField` and `UILabel` components are masked by default, and there is no way to unmask them. You can also mask any view manually by calling:

```jsx
// UIKit
SessionReplay.getInstance().addSensitiveView(mySensiveView);
```

If your app is SwiftUI-based, the automatic masking for `UITextField` and `UILabel` does not work well in the current alpha version, so you need to manually mask any view.

```jsx
Image("family photo").replaySensitive();
```

For More FAQ, please refer to Mixpanel Session Replay [FAQ](https://docs.mixpanel.com/docs/features/session-replay#faq)
