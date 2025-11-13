# React Native Streaming Channel — Test Conditions

## Purpose
Comprehensive, measurable test specifications for the React Native mobile streaming client. Covers authentication, audio/text streaming, session management, network resilience, platform-specific features, and observability with technology-agnostic patterns.

---

## 1. Authentication & Session Management

### 1.1 User Sign-In with Valid Credentials (iOS)
**Setup:**
- Fresh app install on iOS 16+ device
- User has valid account credentials

**Steps:**
1. Launch app
2. Tap "Sign In"
3. Enter credentials: email + password
4. Submit authentication request

**Expected:**
- Authentication request sent to Mobile Authentication Service
- Token received: JWT with 24-hour expiry
- Token stored securely in iOS Keychain
- User lands on conversation screen
- Session created with session_id

**Verification:**
- Token retrievable from Keychain
- Token contains user_id, exp claim
- Session_id logged in telemetry
- Authentication latency < 2 seconds

### 1.2 User Sign-In with Valid Credentials (Android)
**Setup:**
- Fresh app install on Android 12+ device
- User has valid account credentials

**Steps:**
1. Launch app
2. Tap "Sign In"
3. Enter credentials: email + password
4. Submit authentication request

**Expected:**
- Authentication request sent to Mobile Authentication Service
- Token received: JWT with 24-hour expiry
- Token stored securely in Android Keystore
- User lands on conversation screen
- Session created with session_id

**Verification:**
- Token retrievable from Keystore
- Token contains user_id, exp claim
- Session_id logged in telemetry
- Authentication latency < 2 seconds

### 1.3 Token Refresh Before Expiry
**Setup:**
- User authenticated with token expiring in 5 minutes
- App in foreground, active conversation

**Steps:**
1. Wait until 2 minutes before expiry
2. App detects approaching expiry
3. App requests token refresh

**Expected:**
- Refresh request sent automatically
- New token received with 24-hour expiry
- Old token invalidated
- Session continues without interruption
- User not prompted to re-authenticate

**Verification:**
- New token stored in secure storage
- Expiry timestamp updated
- No visible UI disruption
- Telemetry event: token.refreshed

### 1.4 Token Refresh Failure (Network Unavailable)
**Setup:**
- User authenticated, token expires in 1 minute
- Network disconnected

**Steps:**
1. App attempts token refresh
2. Network unavailable
3. Token expires

**Expected:**
- Refresh attempt fails with network error
- App displays message: "Connection lost. Reconnecting..."
- App queues outgoing messages locally
- When network returns, app retries refresh
- Session resumes after successful refresh

**Verification:**
- User informed of connectivity issue
- No data loss during offline period
- Queued messages sent after reconnection
- Telemetry: token.refresh_failed, token.refresh_retry

### 1.5 Guest Access (No Authentication)
**Setup:**
- Fresh app install
- User taps "Try as Guest"

**Steps:**
1. Launch app
2. Select "Try as Guest"
3. App requests guest token

**Expected:**
- Guest token received (limited permissions)
- Token valid for 1 hour
- Guest session created with guest_session_id
- Conversation screen displayed
- Limited features (text only, no history)

**Verification:**
- Token contains guest: true claim
- Session_id prefixed with "guest-"
- Telemetry: session.guest_started
- Feature flags limit capabilities

### 1.6 Session Resumption (Background to Foreground)
**Setup:**
- User authenticated, active conversation
- App sent to background for 5 minutes
- Session still valid on server

**Steps:**
1. User taps app icon to return to foreground
2. App checks session validity
3. App resumes session

**Expected:**
- Session reconnects within 2 seconds
- Transcript displays all previous messages
- Pending messages retrieved from server
- Audio stream ready to resume
- No re-authentication required

**Verification:**
- Session_id matches previous session
- Message continuity maintained
- Telemetry: session.resumed
- Latency < 2 seconds

### 1.7 Session Expiry (App Terminated for 24+ Hours)
**Setup:**
- User authenticated
- App terminated by OS
- 25 hours pass (token expired)

**Steps:**
1. User launches app
2. App detects expired token
3. App prompts re-authentication

**Expected:**
- Token refresh fails (expired)
- User sees login screen with message: "Session expired. Please sign in."
- Previous session_id invalidated
- Local transcript retained (if policy allows)
- User must re-authenticate to continue

**Verification:**
- Old token removed from secure storage
- Login prompt displayed
- Telemetry: session.expired, auth.required
- No automatic retry (user action required)

### 1.8 Multi-Device Session Continuity
**Setup:**
- User authenticated on Device A (iPhone)
- Active conversation with session_id: conv-123
- User signs in on Device B (Android tablet)

**Steps:**
1. Device B authenticates with same account
2. Device B requests session list
3. User selects session conv-123 to resume

**Expected:**
- Device B retrieves session state from Session Orchestration
- Transcript synchronized (all messages visible)
- Audio playback position restored (if applicable)
- Both devices can send/receive messages
- Message order preserved across devices

**Verification:**
- Session_id identical on both devices
- Transcript matches exactly
- New messages appear on both devices within 3 seconds
- Telemetry: session.multi_device_resumed

---

## 2. Text Messaging

### 2.1 Send Single Text Message
**Setup:**
- User authenticated, conversation screen open
- Network available (WiFi)

**Steps:**
1. User types message: "Hello Si"
2. User taps send button

**Expected:**
- Message displayed in transcript immediately (optimistic UI)
- Message sent to Mobile Streaming Gateway
- Server acknowledgment received within 1 second
- Message status icon: sending → sent ✓
- Telemetry: message.sent

**Verification:**
- Message in transcript with timestamp
- Acknowledgment confirms delivery
- Server latency < 1 second (P95)
- Message_id assigned

### 2.2 Send Rapid Text Messages
**Setup:**
- User authenticated, conversation active
- Network available

**Steps:**
1. User sends 10 messages in quick succession (1 per second)
2. Each message: "Message N" (N = 1 to 10)

**Expected:**
- All 10 messages displayed in transcript immediately
- All messages sent to server in order
- All acknowledgments received within 5 seconds total
- Message order preserved (1 → 10)
- No messages dropped

**Verification:**
- Transcript shows all 10 messages in order
- All messages show "sent" status ✓
- Server order matches client order
- Telemetry: message.batch_sent (count: 10)

### 2.3 Send Long-Form Message (2000 Characters)
**Setup:**
- User authenticated
- Network available

**Steps:**
1. User types 2000-character message
2. User taps send

**Expected:**
- Message displayed in transcript (possibly scrollable)
- Message sent to server (within size limits)
- Acknowledgment received within 2 seconds
- Character count displayed during composition
- Send button enabled (within limit)

**Verification:**
- Full message content delivered
- No truncation
- Server latency < 2 seconds
- Telemetry: message.sent (size_bytes: ~2000)

### 2.4 Send Message with Emoji and Unicode
**Setup:**
- User authenticated
- Network available

**Steps:**
1. User types message: "Hello 👋 Si! مرحبا"
2. User taps send

**Expected:**
- Emoji and Arabic text displayed correctly in transcript
- Message sent with UTF-8 encoding
- Acknowledgment received
- Emoji rendered consistently across platforms

**Verification:**
- Message content matches original (no encoding errors)
- Emoji displays correctly on iOS and Android
- Arabic text displays right-to-left
- Telemetry: message.sent (encoding: utf-8)

### 2.5 Receive Text Message from Si
**Setup:**
- User authenticated, conversation active
- Si sends response: "Hello! How can I help you today?"

**Steps:**
1. Server pushes message to mobile client via Mobile Streaming Gateway
2. Client receives message

**Expected:**
- Message appears in transcript within 1 second
- Message attributed to Si (avatar/name)
- Timestamp displayed
- Notification shown if app backgrounded
- Audio chime plays (if settings allow)

**Verification:**
- Message latency < 1 second (P95)
- Correct attribution (sender: Si)
- Telemetry: message.received
- Notification delivered if backgrounded

### 2.6 Receive Long Response from Si
**Setup:**
- User asks complex question
- Si responds with 1500-word answer

**Steps:**
1. Si's response streams to client (incremental delivery)
2. Client renders message incrementally

**Expected:**
- Message appears token-by-token or sentence-by-sentence
- Transcript auto-scrolls to show latest content
- Full message rendered within 3 seconds
- User can scroll back while message streams
- Final message shows timestamp when complete

**Verification:**
- Streaming latency < 100ms per chunk
- Total render time < 3 seconds
- No UI freezing during render
- Telemetry: message.streamed (chunks: N, duration_ms: X)

### 2.7 Send Message While Offline (Queue for Later)
**Setup:**
- User authenticated, conversation active
- Network disconnected (airplane mode)

**Steps:**
1. User types message: "Are you there?"
2. User taps send

**Expected:**
- Message displayed in transcript immediately
- Message status icon: queued (clock icon)
- Message stored locally in outbox
- Banner displayed: "No connection. Message will send when online."
- Message remains in queue

**Verification:**
- Message visible in transcript
- Status: queued (not sent)
- Message persisted to local storage
- Telemetry: message.queued

### 2.8 Synchronize Queued Messages (Offline → Online)
**Setup:**
- 5 messages queued during offline period
- Network reconnected

**Steps:**
1. App detects network availability
2. App sends queued messages in order

**Expected:**
- All 5 messages sent within 10 seconds
- Message order preserved (1 → 5)
- Status icons update: queued → sending → sent ✓
- Server acknowledgments received
- Banner displayed: "Messages sent"

**Verification:**
- All messages delivered
- Order matches queue order
- No duplicates
- Telemetry: message.sync_completed (count: 5)

### 2.9 Handle Message Send Failure (Server Error)
**Setup:**
- User authenticated, network available
- Server returns 500 error for message

**Steps:**
1. User sends message: "Hello"
2. Server responds with error

**Expected:**
- Message status icon: failed (red X)
- Error message displayed: "Message failed to send. Tap to retry."
- Message remains in transcript
- User can tap to retry
- Telemetry: message.send_failed (error: 500)

**Verification:**
- User informed of failure
- Retry mechanism available
- No silent failure
- Error logged for debugging

### 2.10 Transcript Persistence (App Restart)
**Setup:**
- User has 50 messages in conversation
- User closes app (force quit)
- User reopens app after 1 hour

**Steps:**
1. User launches app
2. App authenticates and resumes session
3. App loads transcript from local storage

**Expected:**
- All 50 messages visible in transcript
- Message order preserved
- Timestamps accurate
- Transcript loads within 2 seconds
- User can scroll to view history

**Verification:**
- Transcript completeness: 100%
- Load latency < 2 seconds
- No data loss
- Telemetry: transcript.loaded (message_count: 50)

---

## 3. Audio Streaming

### 3.1 Capture and Stream Audio (Push-to-Talk)
**Setup:**
- User authenticated, microphone permission granted
- Network available (WiFi)

**Steps:**
1. User taps and holds microphone button
2. User speaks: "What's the weather today?"
3. User releases button (3 seconds total)

**Expected:**
- Recording indicator displayed (red dot)
- Audio captured at 16kHz sample rate
- Audio streamed to server in 100ms chunks
- Recording stops when button released
- Waveform visualization displayed (optional)
- Audio upload completes within 500ms after release

**Verification:**
- Audio quality: 16kHz, Opus codec
- Chunk size: ~1.6KB per 100ms
- Stream latency < 150ms per chunk (P95)
- Total audio duration: ~3 seconds
- Telemetry: audio.captured (duration_ms: 3000)

### 3.2 Receive and Play Audio Response from Si
**Setup:**
- User sent audio query
- Si responds with audio (5 seconds)

**Steps:**
1. Server streams audio response to client
2. Client buffers and plays audio

**Expected:**
- Audio playback starts within 2 seconds (P95 onset latency)
- Audio plays smoothly without interruptions
- Playback indicator displayed (speaker icon)
- User can pause/resume playback
- Transcript displays text version simultaneously

**Verification:**
- Onset latency < 2 seconds (P95)
- Audio quality: clear, no distortion
- Buffer underruns: 0
- Telemetry: audio.played (duration_ms: 5000, onset_latency_ms: X)

### 3.3 Audio Streaming Under Poor Network (Cellular 3G)
**Setup:**
- User on 3G network (simulated: 1 Mbps, 300ms latency)
- User initiates audio capture

**Steps:**
1. User records 5-second audio message
2. App streams audio to server

**Expected:**
- Adaptive bitrate reduces audio quality (8kHz fallback)
- Audio chunks buffered if network slow
- Upload completes within 10 seconds total
- User sees progress indicator
- Banner: "Sending audio... (slow connection)"

**Verification:**
- Audio delivered successfully (quality reduced)
- No upload failures
- Adaptive encoding applied
- Telemetry: audio.uploaded (network: 3g, duration_upload_ms: X)

### 3.4 Handle Audio Interruption (Incoming Phone Call - iOS)
**Setup:**
- User in active audio conversation with Si
- Incoming phone call received on iOS device

**Steps:**
1. Phone call interrupts audio session
2. User answers call
3. User ends call and returns to app

**Expected:**
- Audio playback pauses immediately
- App saves session state
- After call ends, app displays: "Resume conversation?"
- User taps "Resume"
- Audio session reconnects within 3 seconds

**Verification:**
- No audio loss during interruption
- Session state preserved
- Reconnection successful
- Telemetry: audio.interrupted (reason: phone_call), audio.resumed

### 3.5 Handle Audio Interruption (Incoming Phone Call - Android)
**Setup:**
- User in active audio conversation with Si
- Incoming phone call received on Android device

**Steps:**
1. Phone call interrupts audio session
2. User answers call
3. User ends call and returns to app

**Expected:**
- Audio playback pauses immediately
- App saves session state
- After call ends, app displays: "Resume conversation?"
- User taps "Resume"
- Audio session reconnects within 3 seconds

**Verification:**
- No audio loss during interruption
- Session state preserved
- Reconnection successful
- Telemetry: audio.interrupted (reason: phone_call), audio.resumed

### 3.6 Adaptive Audio Quality (WiFi → Cellular Handoff)
**Setup:**
- User streaming audio on WiFi (high quality: 16kHz Opus)
- User walks outside, device switches to LTE

**Steps:**
1. Network handoff detected
2. App measures new bandwidth
3. App adjusts audio quality

**Expected:**
- Audio quality reduces to 8kHz if bandwidth < 500 Kbps
- Handoff latency < 2 seconds
- Audio stream continues without interruption
- User sees indicator: "Audio quality adjusted for connection"
- Transcript remains fully functional

**Verification:**
- No audio dropouts during handoff
- Quality adjustment applied within 2 seconds
- Streaming continues
- Telemetry: audio.quality_adjusted (from: 16khz, to: 8khz, reason: bandwidth)

### 3.7 Audio Buffering and Jitter Handling
**Setup:**
- User receiving audio from Si
- Network has variable latency (100-500ms jitter)

**Steps:**
1. Si's audio response streams to client
2. Network introduces jitter

**Expected:**
- App buffers 500ms of audio before playback
- Jitter absorbed by buffer
- Playback smooth (no stuttering)
- Buffer size adapts if jitter increases
- Playback latency increases slightly (acceptable trade-off)

**Verification:**
- Playback quality: smooth, no stuttering
- Buffer underruns: 0
- Onset latency < 3 seconds (includes buffering)
- Telemetry: audio.buffer_stats (underruns: 0, size_ms: 500)

### 3.8 Concurrent Audio and Text (Transcript During Playback)
**Setup:**
- User receiving audio response from Si (10 seconds)
- Audio playing through speaker

**Steps:**
1. Audio plays
2. User types text message: "Thanks!"
3. User sends message while audio continues

**Expected:**
- Text input available during audio playback
- Message sent successfully
- Audio playback continues uninterrupted
- Transcript shows both audio and text messages
- No resource contention

**Verification:**
- Audio not interrupted by text send
- Text message delivered successfully
- No UI lag
- Telemetry: audio.played (concurrent_text: true)

### 3.9 Audio Playback with Bluetooth Headset
**Setup:**
- User has Bluetooth headset connected (iOS)
- User receives audio response from Si

**Steps:**
1. Audio response arrives
2. App routes audio to Bluetooth device

**Expected:**
- Audio plays through Bluetooth headset (not speaker)
- Routing happens automatically
- Audio quality preserved
- User can switch to speaker if desired
- Playback controls work on Bluetooth device (if supported)

**Verification:**
- Audio output: Bluetooth device
- No routing errors
- Quality maintained
- Telemetry: audio.output_device (bluetooth)

### 3.10 Microphone Permission Denied (iOS)
**Setup:**
- Fresh app install on iOS
- User attempts to use voice input
- Microphone permission not yet granted

**Steps:**
1. User taps microphone button
2. iOS prompts for permission
3. User denies permission

**Expected:**
- Permission prompt displayed (iOS system dialog)
- User denies permission
- App displays message: "Microphone access required for voice input. Enable in Settings."
- Microphone button disabled or grayed out
- Text input remains available
- Link to app settings provided

**Verification:**
- User informed of requirement
- Clear guidance to enable permission
- No crash or silent failure
- Telemetry: permission.denied (microphone)

### 3.11 Microphone Permission Denied (Android)
**Setup:**
- Fresh app install on Android
- User attempts to use voice input
- Microphone permission not yet granted

**Steps:**
1. User taps microphone button
2. Android prompts for permission
3. User denies permission

**Expected:**
- Permission prompt displayed (Android system dialog)
- User denies permission
- App displays message: "Microphone access required for voice input. Enable in Settings."
- Microphone button disabled or grayed out
- Text input remains available
- Link to app settings provided

**Verification:**
- User informed of requirement
- Clear guidance to enable permission
- No crash or silent failure
- Telemetry: permission.denied (microphone)

---

## 4. App Lifecycle & Background Behavior

### 4.1 Foreground → Background Transition (Conversation Active)
**Setup:**
- User in active conversation
- 5 messages exchanged
- User presses home button (iOS) or back button (Android)

**Steps:**
1. App moves to background
2. App saves session state
3. App maintains connection for 30 seconds

**Expected:**
- Session state persisted to local storage
- Connection kept alive for 30 seconds (grace period)
- After 30 seconds, connection gracefully closed
- Background task registered (if platform allows)
- Notification permission requested (if not already granted)

**Verification:**
- Session state saved: session_id, transcript, audio position
- Connection behavior: alive 30s, then close
- No data loss
- Telemetry: app.backgrounded (session_active: true)

### 4.2 Background → Foreground Transition (Resume Conversation)
**Setup:**
- App in background for 5 minutes
- User taps app icon to return

**Steps:**
1. App moves to foreground
2. App checks session validity
3. App reconnects to server

**Expected:**
- Session resumption within 2 seconds
- Transcript displayed (all previous messages)
- New messages retrieved from server (if any)
- Connection re-established
- User can continue conversation immediately

**Verification:**
- Reconnection latency < 2 seconds
- Transcript completeness: 100%
- New messages appear (if sent during background)
- Telemetry: app.foregrounded, session.resumed

### 4.3 Receive Push Notification (App Backgrounded)
**Setup:**
- User authenticated, app backgrounded
- Si sends message: "Your appointment is tomorrow at 2pm"

**Steps:**
1. Server sends push notification via Notification Service
2. iOS/Android delivers notification

**Expected:**
- Notification appears on lock screen and notification center
- Title: "Si"
- Body: "Your appointment is tomorrow at 2pm"
- Notification includes session_id metadata
- User can tap notification to open app
- Tapping opens directly to conversation

**Verification:**
- Notification delivered within 5 seconds (P95)
- Correct message content
- Deep link functional (opens correct conversation)
- Telemetry: notification.received, notification.tapped

### 4.4 Handle Notification Denial (Permission Not Granted)
**Setup:**
- User denied notification permission
- App backgrounded
- Si sends message

**Steps:**
1. Server attempts to send push notification
2. Notification fails (permission denied)

**Expected:**
- Notification not delivered (expected)
- Message stored on server
- When user returns to app, message appears in transcript
- App prompts user: "Enable notifications to stay updated"
- Link to settings provided

**Verification:**
- Message delivered when app foregrounded
- User informed about notifications
- No silent failure
- Telemetry: notification.failed (reason: permission_denied)

### 4.5 App Terminated by OS (Memory Pressure)
**Setup:**
- App in background
- OS terminates app due to memory pressure
- Session had 20 messages

**Steps:**
1. OS terminates app process
2. User reopens app 1 hour later

**Expected:**
- App cold starts
- Authentication token retrieved from secure storage
- Session resumed (if token valid)
- Transcript loaded from local storage (20 messages)
- Connection re-established within 3 seconds
- User sees: "Reconnecting..."

**Verification:**
- Full app restart (cold start)
- Session continuity maintained
- Transcript complete
- Reconnection successful
- Telemetry: app.cold_start, session.resumed

### 4.6 Background Audio Playback (iOS Background Modes)
**Setup:**
- User receiving audio response from Si (30 seconds)
- User presses home button during playback

**Steps:**
1. App moves to background
2. Audio continues playing

**Expected:**
- Audio playback continues in background (iOS audio session)
- Playback controls appear on lock screen
- User can pause/play from lock screen
- Transcript updates when app returns to foreground
- App maintains audio session for duration of response

**Verification:**
- Background audio session active
- Lock screen controls functional
- Playback completes successfully
- Telemetry: audio.background_playback (duration_ms: X)

### 4.7 Handle Low Battery Mode (iOS)
**Setup:**
- Device in Low Power Mode (iOS)
- User attempts to start conversation

**Steps:**
1. User initiates conversation
2. App detects Low Power Mode
3. App adjusts behavior

**Expected:**
- App functions normally (reduced performance acceptable)
- Background refresh disabled
- Audio quality may reduce
- Notifications still delivered
- Message: "Low Power Mode active. Some features limited."

**Verification:**
- App functional
- Background tasks disabled
- Notifications still work
- Telemetry: app.low_power_mode (enabled: true)

### 4.8 Handle Doze Mode (Android)
**Setup:**
- Device in Doze Mode (screen off, idle)
- Si sends message

**Steps:**
1. Server sends push notification
2. Android delivers high-priority notification

**Expected:**
- Notification delivered during Doze Mode
- App wakes briefly to process notification
- Message stored locally
- User sees notification on lock screen
- App returns to Doze Mode after processing

**Verification:**
- Notification delivered (high priority)
- Message available when app opened
- Battery impact minimal
- Telemetry: notification.doze_delivery

---

## 5. Network Resilience & Connectivity

### 5.1 Handle Network Disconnect (Mid-Conversation)
**Setup:**
- User in active conversation
- Network disconnected (airplane mode enabled)

**Steps:**
1. User sends message
2. Network unavailable

**Expected:**
- Message queued locally
- Status icon: queued (clock)
- Banner: "No connection. Message will send when online."
- Transcript remains accessible
- User can continue composing messages (all queued)

**Verification:**
- Message persisted to outbox
- User informed of status
- No data loss
- Telemetry: network.disconnected

### 5.2 Network Reconnection (Restore After Disconnect)
**Setup:**
- 5 messages queued during offline period
- Network reconnected

**Steps:**
1. Network becomes available
2. App detects connectivity
3. App sends queued messages

**Expected:**
- Banner: "Reconnecting..."
- All 5 messages sent within 10 seconds
- Status updates: queued → sending → sent ✓
- Server acknowledgments received
- Conversation resumes normally

**Verification:**
- All messages delivered
- Order preserved
- No duplicates
- Telemetry: network.reconnected, message.sync_completed (count: 5)

### 5.3 WiFi to Cellular Handoff (Seamless Transition)
**Setup:**
- User on WiFi, active conversation
- User walks outside, WiFi signal lost
- Device switches to cellular (LTE)

**Steps:**
1. WiFi disconnects
2. Device switches to cellular
3. App detects network change

**Expected:**
- Handoff latency < 3 seconds
- Connection re-established on cellular
- Conversation continues without interruption
- Audio quality may adjust (if streaming)
- Message in-flight: retried if needed

**Verification:**
- Handoff successful
- No message loss
- Latency < 3 seconds
- Telemetry: network.handoff (from: wifi, to: cellular)

### 5.4 Cellular to WiFi Handoff (Improve Quality)
**Setup:**
- User on cellular (3G), audio quality reduced
- User enters WiFi zone, device connects

**Steps:**
1. Device connects to WiFi
2. App detects improved bandwidth
3. App increases audio quality

**Expected:**
- Handoff to WiFi within 2 seconds
- Audio quality increased (16kHz)
- Message latency improves
- Banner: "Connected to WiFi"
- User experience improves

**Verification:**
- Quality adjustment applied
- Bandwidth improvement utilized
- No disruption
- Telemetry: network.handoff (from: cellular, to: wifi), audio.quality_upgraded

### 5.5 Handle Intermittent Connectivity (Packet Loss)
**Setup:**
- User on poor network (30% packet loss)
- User sends text message

**Steps:**
1. Message sent
2. Network drops packets
3. App retries delivery

**Expected:**
- Message delivery retried (TCP/WebSocket retries)
- Retry attempts: up to 3
- Backoff: exponential (1s, 2s, 4s)
- Message delivers successfully or fails after 3 attempts
- User informed if delivery fails

**Verification:**
- Retry logic applied
- Message eventually delivered (if network stabilizes)
- User informed of status
- Telemetry: message.retry (attempt: N)

### 5.6 Handle Complete Network Outage (Extended Offline)
**Setup:**
- User offline for 30 minutes
- User sends 10 messages during outage
- Network returns

**Steps:**
1. All 10 messages queued locally
2. Network reconnects
3. App synchronizes messages

**Expected:**
- All 10 messages sent in order
- Synchronization completes within 15 seconds
- No message loss or duplication
- Server timestamps reflect actual send time (not queue time)
- User sees: "Sent 10 messages"

**Verification:**
- All messages delivered
- Order preserved (1-10)
- No duplicates
- Telemetry: message.sync_completed (count: 10, duration_ms: X)

### 5.7 Handle Slow Network (High Latency)
**Setup:**
- User on satellite connection (2000ms latency)
- User sends text message

**Steps:**
1. Message sent
2. Server response delayed

**Expected:**
- Message status: sending... (spinner)
- Acknowledgment arrives after 4+ seconds
- Status updates: sending → sent ✓
- User informed: "Slow connection detected"
- Message eventually delivered

**Verification:**
- Message delivered successfully
- User informed of delay
- No timeout (patient retry)
- Telemetry: message.sent (latency_ms: 4000+)

### 5.8 Handle Server Timeout (No Response)
**Setup:**
- User sends message
- Server unresponsive (30+ seconds)

**Steps:**
1. Message sent
2. No acknowledgment received within 30 seconds
3. App times out

**Expected:**
- Message status: failed (red X)
- Error message: "Server not responding. Tap to retry."
- Message remains in transcript
- User can retry manually
- Automatic retry after 60 seconds (background)

**Verification:**
- User informed of failure
- Retry mechanism available
- No silent failure
- Telemetry: message.timeout, message.retry_scheduled

---

## 6. Platform-Specific Features & Compliance

### 6.1 VoiceOver Support (iOS Accessibility)
**Setup:**
- iOS device with VoiceOver enabled
- User navigates app with VoiceOver

**Steps:**
1. User swipes through conversation screen
2. VoiceOver reads each UI element
3. User focuses on message and activates

**Expected:**
- All buttons have accessibility labels
- Messages read aloud (sender + content + time)
- Microphone button: "Record audio message"
- Send button: "Send message"
- Navigation clear and logical
- User can compose and send messages using VoiceOver

**Verification:**
- All elements accessible via VoiceOver
- Labels descriptive and accurate
- Navigation order logical
- Gestures functional
- Telemetry: accessibility.voiceover_enabled

### 6.2 TalkBack Support (Android Accessibility)
**Setup:**
- Android device with TalkBack enabled
- User navigates app with TalkBack

**Steps:**
1. User swipes through conversation screen
2. TalkBack reads each UI element
3. User focuses on message and activates

**Expected:**
- All buttons have content descriptions
- Messages read aloud (sender + content + time)
- Microphone button: "Record audio message"
- Send button: "Send message"
- Navigation clear and logical
- User can compose and send messages using TalkBack

**Verification:**
- All elements accessible via TalkBack
- Descriptions descriptive and accurate
- Navigation order logical
- Gestures functional
- Telemetry: accessibility.talkback_enabled

### 6.3 Dynamic Text Size (iOS)
**Setup:**
- iOS device with "Larger Accessibility Sizes" enabled
- User sets text size to maximum

**Steps:**
1. User opens conversation screen
2. Text scales to large size

**Expected:**
- All text scales appropriately
- Layout adapts without clipping
- Buttons remain accessible
- Scrolling available for long messages
- No text truncation

**Verification:**
- Text readable at all sizes
- Layout integrity maintained
- No overlapping elements
- Usability preserved
- Telemetry: accessibility.text_size (large)

### 6.4 High Contrast Mode (Android)
**Setup:**
- Android device with high contrast enabled
- User opens app

**Steps:**
1. App detects high contrast setting
2. UI adjusts colors

**Expected:**
- Text contrast ratio ≥ 7:1 (WCAG AAA)
- Background and foreground colors adjusted
- All text readable
- Icons have sufficient contrast
- User experience improved

**Verification:**
- Contrast ratios measured and compliant
- Readability improved
- No color-only indicators (icons + labels)
- Telemetry: accessibility.high_contrast_enabled

### 6.5 Secure Data Storage (iOS Keychain)
**Setup:**
- User authenticates on iOS
- App stores auth token

**Steps:**
1. Token stored in iOS Keychain
2. App terminates
3. App relaunches

**Expected:**
- Token stored with kSecAttrAccessibleWhenUnlockedThisDeviceOnly
- Token not synced to iCloud (device-only)
- Token protected by device passcode/biometric
- Token retrieval requires device unlock
- Token deleted on app uninstall

**Verification:**
- Token secured in Keychain
- No plaintext storage
- Deletion on uninstall
- Telemetry: auth.token_stored (secure: true)

### 6.6 Secure Data Storage (Android Keystore)
**Setup:**
- User authenticates on Android
- App stores auth token

**Steps:**
1. Token stored in Android Keystore
2. App terminates
3. App relaunches

**Expected:**
- Token stored with setUserAuthenticationRequired(false) for background access
- Token encrypted by hardware-backed keystore
- Token protected by device lock
- Token not extractable
- Token deleted on app uninstall

**Verification:**
- Token secured in Keystore
- Hardware-backed encryption (if available)
- Deletion on uninstall
- Telemetry: auth.token_stored (secure: true, hardware_backed: true/false)

### 6.7 Data Export Request (GDPR)
**Setup:**
- User authenticated
- User requests data export via settings

**Steps:**
1. User taps "Download My Data"
2. App requests export from backend
3. Backend generates export
4. User receives download link via email

**Expected:**
- Export request submitted successfully
- Request_id assigned
- Email sent within 24 hours
- Export includes: transcripts, audio metadata, consent records
- Export format: JSON or CSV
- Export link expires after 7 days

**Verification:**
- Request processed
- Export complete
- User informed of timeline
- Telemetry: data_export.requested

### 6.8 Data Deletion Request (GDPR)
**Setup:**
- User authenticated
- User requests data deletion via settings

**Steps:**
1. User taps "Delete My Data"
2. Confirmation dialog displayed
3. User confirms deletion
4. App submits deletion request

**Expected:**
- Confirmation required (destructive action)
- Deletion request submitted to backend
- Request_id assigned
- User receives confirmation email
- Data deleted within 30 days (compliance requirement)
- User account disabled immediately

**Verification:**
- Deletion request processed
- User account deactivated
- Email confirmation sent
- Telemetry: data_deletion.requested

### 6.9 Handle Jailbroken/Rooted Device (Security Risk)
**Setup:**
- User attempts to run app on jailbroken iOS device

**Steps:**
1. App detects jailbreak on launch
2. App displays warning

**Expected:**
- Warning message: "Security risk detected. App may not function properly on jailbroken devices."
- User can proceed (warning only, not blocking)
- Certain security features disabled (if policy requires)
- Telemetry: security.jailbreak_detected

**Verification:**
- User informed of risk
- App functional (policy-dependent)
- Security team notified
- Telemetry logged

### 6.10 Battery Usage Monitoring (Extended Session)
**Setup:**
- User in 2-hour conversation (text + audio)
- Battery monitoring enabled

**Steps:**
1. User starts conversation at 100% battery
2. 2 hours pass with active usage
3. Measure battery consumption

**Expected:**
- Battery consumption ≤ 15% per hour (text heavy)
- Battery consumption ≤ 25% per hour (audio heavy)
- No abnormal drain
- No overheating
- App remains responsive

**Verification:**
- Battery usage within acceptable range
- No thermal issues
- Performance maintained
- Telemetry: battery.usage (percent_per_hour: X)

---

## 7. Error Handling & Recovery

### 7.1 Handle WebSocket Connection Failure
**Setup:**
- User authenticated, conversation starting
- WebSocket connection fails (server unreachable)

**Steps:**
1. App attempts WebSocket connection
2. Connection fails after 5 seconds

**Expected:**
- Retry with exponential backoff: 1s, 2s, 4s, 8s
- Max retries: 5
- Error message: "Unable to connect. Retrying..."
- Fallback to text-only mode if audio unavailable
- User can send messages (queued)

**Verification:**
- Retry logic applied
- User informed of issue
- Graceful degradation
- Telemetry: connection.failed, connection.retry (attempt: N)

### 7.2 Handle Server Error (500 Internal Server Error)
**Setup:**
- User sends message
- Server returns 500 error

**Steps:**
1. Message sent
2. Server responds with 500

**Expected:**
- Message status: failed (red X)
- Error message: "Server error. Tap to retry."
- Automatic retry after 10 seconds (background)
- User can retry manually immediately
- Error logged with request_id

**Verification:**
- User informed of error
- Retry available
- Error logged for debugging
- Telemetry: message.send_failed (error: 500, request_id: X)

### 7.3 Handle Authentication Failure (401 Unauthorized)
**Setup:**
- User session active
- Token invalidated on server (security event)

**Steps:**
1. User sends message
2. Server returns 401

**Expected:**
- Session terminated
- User logged out immediately
- Message: "Session expired. Please sign in again."
- Local data retained (if policy allows)
- User redirected to login screen

**Verification:**
- User logged out
- Session cleared
- Token removed from storage
- Telemetry: auth.failed (error: 401)

### 7.4 Handle Rate Limiting (429 Too Many Requests)
**Setup:**
- User sends 100 messages in 1 minute
- Server rate limits user

**Steps:**
1. Server returns 429 with Retry-After: 60 seconds

**Expected:**
- Message status: rate limited
- Error message: "Sending too fast. Please wait 60 seconds."
- Countdown timer displayed
- Send button disabled for 60 seconds
- User can compose messages (queued)

**Verification:**
- User informed of limit
- Retry delayed appropriately
- No spamming
- Telemetry: message.rate_limited (retry_after_seconds: 60)

### 7.5 Handle Invalid Message Format (400 Bad Request)
**Setup:**
- App has bug causing malformed message
- User sends message

**Steps:**
1. Server returns 400 with error details

**Expected:**
- Message status: failed
- Error message: "Unable to send message. Please try again."
- Error details logged (not shown to user)
- User can retry (after app fix via update)
- Bug reported to monitoring system

**Verification:**
- User informed (user-friendly message)
- Error logged with details
- Development team alerted
- Telemetry: message.send_failed (error: 400, details: X)

### 7.6 Handle Audio Codec Error (Unsupported Format)
**Setup:**
- Server sends audio in unsupported format

**Steps:**
1. Audio received
2. Decoder fails

**Expected:**
- Fallback to text transcript
- Error message: "Audio unavailable. Showing text instead."
- Transcript displayed correctly
- Audio error logged
- User experience maintained (text fallback)

**Verification:**
- Graceful degradation
- Text fallback successful
- Error logged
- Telemetry: audio.decode_failed (format: X)

### 7.7 Handle Out of Memory (Low Memory Warning)
**Setup:**
- Device has low available memory
- App receives memory warning

**Steps:**
1. OS sends low memory warning
2. App receives warning

**Expected:**
- App releases cached data (images, audio buffers)
- Active conversation preserved
- Transcript truncated to recent 100 messages (older messages reload on demand)
- User not interrupted
- Memory usage reduced by ≥ 50%

**Verification:**
- Memory freed successfully
- No app crash
- Conversation continues
- Telemetry: memory.warning, memory.freed (bytes: X)

### 7.8 Handle Crash and Auto-Recovery (iOS)
**Setup:**
- App crashes due to bug
- User relaunches app

**Steps:**
1. App crashes during conversation
2. User reopens app immediately

**Expected:**
- App cold starts
- Crash report generated and queued for upload
- Session recovery attempted
- Last message may not be sent (user warned)
- Message: "App restarted after error. Last message may not have been sent."

**Verification:**
- Crash report generated
- Session recovery attempted
- User informed of potential issue
- Telemetry: app.crash, app.crash_recovery

---

## 8. Performance & Scalability

### 8.1 App Launch Time (Cold Start)
**Setup:**
- App not running (fresh launch)
- Device: iPhone 12, iOS 16

**Steps:**
1. User taps app icon
2. Measure time to interactive

**Expected:**
- Launch time < 2 seconds (P95)
- Splash screen displayed immediately
- UI interactive within 2 seconds
- Authentication check completes within 3 seconds total
- User can start typing immediately

**Verification:**
- Launch latency measured
- Time to interactive < 2 seconds
- No blocking operations on main thread
- Telemetry: app.launch (duration_ms: X, type: cold)

### 8.2 App Launch Time (Warm Start)
**Setup:**
- App in background, not terminated
- User returns to app

**Steps:**
1. User taps app icon
2. Measure time to interactive

**Expected:**
- Launch time < 500ms (P95)
- Screen restores immediately
- Session resumes within 1 second
- User can interact immediately

**Verification:**
- Launch latency measured
- Time to interactive < 500ms
- Session continuity maintained
- Telemetry: app.launch (duration_ms: X, type: warm)

### 8.3 Transcript Rendering (1000 Messages)
**Setup:**
- User has 1000-message conversation history
- User opens conversation

**Steps:**
1. App loads transcript from local storage
2. App renders recent messages (last 50)

**Expected:**
- Initial render: 50 most recent messages
- Render time < 1 second
- Infinite scroll: load older messages on demand (50 at a time)
- Scroll performance: 60 FPS
- No UI freezing

**Verification:**
- Initial load time < 1 second
- Scroll performance smooth
- Memory usage reasonable (<100 MB for 50 messages)
- Telemetry: transcript.loaded (message_count: 1000, render_count: 50)

### 8.4 Handle Rapid User Input (Type Fast)
**Setup:**
- User types very quickly
- Input rate: 10 characters per second

**Steps:**
1. User types 100-character message rapidly
2. Measure UI responsiveness

**Expected:**
- Character input latency < 16ms per character (60 FPS)
- No dropped characters
- No input lag
- Send button enables when content present
- UI remains responsive

**Verification:**
- Input responsiveness: 60 FPS
- No dropped input
- UI smooth
- Telemetry: input.rapid_typing (chars_per_second: 10)

### 8.5 Concurrent Operations (Audio Playback + Text Send)
**Setup:**
- Audio response playing (20 seconds)
- User types and sends text message during playback

**Steps:**
1. Audio plays
2. User sends text message

**Expected:**
- Audio continues uninterrupted
- Text message sends successfully
- No resource contention
- UI remains responsive (60 FPS)
- Both operations complete successfully

**Verification:**
- Audio not interrupted
- Text delivered
- No performance degradation
- Telemetry: concurrent_operations (audio_playback + text_send)

### 8.6 Memory Usage (Extended Session)
**Setup:**
- User in 2-hour conversation
- 500 messages exchanged

**Steps:**
1. Monitor memory usage over 2 hours

**Expected:**
- Memory usage stable (< 200 MB)
- No memory leaks
- Old messages paged out (virtual scrolling)
- Active messages retained in memory
- No app termination due to memory

**Verification:**
- Memory usage < 200 MB throughout
- No leaks detected
- App remains stable
- Telemetry: memory.usage (average_mb: X, peak_mb: Y)

### 8.7 Network Bandwidth Usage (Audio Streaming)
**Setup:**
- User in 10-minute audio conversation
- Network: LTE

**Steps:**
1. Measure data usage during conversation

**Expected:**
- Upload: ~5 MB (audio 16kHz Opus)
- Download: ~8 MB (audio + text)
- Total: ~13 MB for 10 minutes
- Compression efficient
- Adaptive quality reduces usage on poor networks

**Verification:**
- Bandwidth usage within expected range
- Compression applied
- Adaptive quality functional
- Telemetry: bandwidth.usage (upload_mb: X, download_mb: Y)

---

## 9. Observability & Monitoring

### 9.1 Client Telemetry Emission (Session Lifecycle)
**Setup:**
- User completes full session: sign-in → conversation → sign-out

**Steps:**
1. User signs in
2. User sends 10 messages
3. User receives 10 responses
4. User signs out

**Expected Events Emitted:**
- `session.started` (session_id, user_id_hash, device_type)
- `message.sent` (count: 10, session_id)
- `message.received` (count: 10, session_id)
- `session.ended` (session_id, duration_ms, message_count: 20)
- All events tagged with: platform (ios/android), app_version, session_id

**Verification:**
- All events present
- Events correlatable by session_id
- Timestamps sequential
- Platform tags correct
- Telemetry: telemetry.batch_sent (event_count: X)

### 9.2 Client Metrics Collection (Latency)
**Setup:**
- User sends 100 messages over 30 minutes

**Steps:**
1. Measure latency for each message send

**Expected Metrics:**
- `client.message.latency_ms` (histogram: p50, p95, p99)
- P50: ~500ms
- P95: ~1200ms
- P99: ~2000ms
- Tagged by: network_type, platform

**Verification:**
- Metrics calculated correctly
- Percentiles accurate
- Tags present
- Telemetry uploaded to observability platform

### 9.3 Error Logging with Context
**Setup:**
- User encounters send failure (server timeout)

**Steps:**
1. Message send times out
2. App logs error

**Expected Log Entry:**
```json
{
  "timestamp": "2025-11-07T14:30:45.123Z",
  "level": "error",
  "service": "react-native-streaming",
  "component": "message-sender",
  "session_id": "sess-123",
  "event_type": "message.send_failed",
  "message": "Message send timeout after 30 seconds",
  "metadata": {
    "message_id": "msg-456",
    "network_type": "cellular",
    "retry_count": 3,
    "error_code": "TIMEOUT"
  },
  "user_id_hash": "<sha256>",
  "pii_scrubbed": true
}
```

**Verification:**
- Log format compliant
- All required fields present
- No plaintext PII
- Context sufficient for debugging
- Error uploaded to telemetry platform

### 9.4 Crash Reporting (with Stack Trace)
**Setup:**
- App crashes due to null pointer exception

**Steps:**
1. App crashes
2. OS generates crash report
3. App uploads crash on next launch

**Expected Crash Report:**
- Exception type: NullPointerException
- Stack trace: full call stack
- Device info: model, OS version, app version
- Session context: session_id, last action
- User_id_hash (no plaintext PII)
- Timestamp of crash

**Verification:**
- Crash report complete
- Stack trace actionable
- Context sufficient for debugging
- No sensitive user data
- Report uploaded to crash reporting service

### 9.5 Performance Monitoring (Real User Metrics)
**Setup:**
- 1000 users active over 24 hours

**Steps:**
1. Collect performance metrics from all users

**Expected Metrics Dashboard:**
- App launch time: P95 < 2 seconds
- Message send latency: P95 < 1.2 seconds
- Audio onset latency: P95 < 2 seconds
- Crash-free rate: ≥ 99.5%
- Session success rate: ≥ 99%

**Verification:**
- Metrics aggregated correctly
- Dashboard accurate
- SLOs tracked
- Alerts triggered if thresholds breached

### 9.6 Feature Flag Telemetry
**Setup:**
- New feature flag: enable_voice_transcription
- Feature enabled for 50% of users (A/B test)

**Steps:**
1. Users launch app
2. Feature flag evaluated
3. Telemetry emitted

**Expected Events:**
- `feature_flag.evaluated` (flag: enable_voice_transcription, enabled: true/false)
- 50% of users: enabled=true
- 50% of users: enabled=false
- Tagged by: user_cohort, session_id

**Verification:**
- Flag evaluation logged
- Distribution accurate (50/50 split)
- No correlation bias (random assignment)
- Telemetry: feature_flag.evaluated

---

## 10. Compliance & Privacy

### 10.1 PII Redaction in Logs
**Setup:**
- User sends message: "My phone is +1-555-1234"

**Steps:**
1. Message logged for debugging
2. Export logs

**Expected:**
- Message content: **REDACTED**
- Phone number: **REDACTED** or hashed
- User ID: hashed (sha256)
- Session ID: preserved (UUID, not PII)
- Metadata preserved: timestamp, latency, network_type

**Verification:**
- Zero plaintext PII in exported logs
- Hashing consistent (same user = same hash)
- Logs useful for debugging
- Compliance audit passes

### 10.2 Consent Recording (Audio Capture)
**Setup:**
- User first-time audio recording
- User taps microphone button

**Steps:**
1. App prompts: "Allow Si to record audio?"
2. User consents

**Expected:**
- Consent record created:
  - `user_id_hash`: <sha256>
  - `consent_type`: "audio_recording"
  - `status`: "granted"
  - `timestamp`: <ISO 8601>
  - `method`: "in_app_prompt"
- Record stored with 7-year retention
- User can revoke consent in settings

**Verification:**
- Consent recorded
- Audit trail complete
- Revocation mechanism available
- Telemetry: consent.granted (type: audio_recording)

### 10.3 Data Retention Policy Enforcement (Transcript)
**Setup:**
- User has 90-day-old transcript
- Retention policy: 90 days

**Steps:**
1. App checks transcript age on launch
2. Identifies messages > 90 days old

**Expected:**
- Messages > 90 days deleted from local storage
- Recent messages (< 90 days) retained
- User notified: "Old messages removed per privacy policy"
- Deletion logged for compliance

**Verification:**
- Old messages deleted
- Policy enforced
- User informed
- Telemetry: data.retention_cleanup (messages_deleted: X)

### 10.4 Encryption at Rest (Local Storage)
**Setup:**
- User authenticated, transcript stored locally
- Device: iOS with Data Protection enabled

**Steps:**
1. Inspect local storage encryption

**Expected:**
- Transcript database encrypted (iOS Data Protection)
- Encryption key tied to device passcode/biometric
- Data inaccessible when device locked
- No plaintext storage

**Verification:**
- Encryption confirmed (iOS File Protection)
- Key management secure
- Compliance requirement met
- Telemetry: storage.encrypted (enabled: true)

### 10.5 Encryption in Transit (TLS)
**Setup:**
- User sends message to server

**Steps:**
1. Inspect network traffic (using proxy)

**Expected:**
- All traffic uses TLS 1.3
- Certificate validation enforced
- No plaintext transmission
- Secure WebSocket (wss://)
- Certificate pinning (optional, if configured)

**Verification:**
- TLS 1.3 confirmed
- No plaintext traffic
- Certificate valid
- Compliance requirement met

---

## Summary

**Total Test Categories:** 10
**Total Test Scenarios:** 110+

**Coverage:**
- Authentication & session management: 8 tests
- Text messaging: 10 tests
- Audio streaming: 11 tests
- App lifecycle & background behavior: 8 tests
- Network resilience & connectivity: 8 tests
- Platform-specific features & compliance: 10 tests
- Error handling & recovery: 8 tests
- Performance & scalability: 7 tests
- Observability & monitoring: 6 tests
- Compliance & privacy: 5 tests

**Test Quality:**
- All tests have specific setup conditions
- All tests have step-by-step procedures
- All tests have measurable expected outcomes
- All tests have clear verification criteria
- Platform-specific (iOS and Android where applicable)
- Technology-agnostic (works with any React Native implementation)
- Suitable for TDD (Test-Driven Development)
- Suitable for automation

**Key Performance Targets:**
- Authentication latency: < 2 seconds
- Message send latency: < 1 second (P95)
- Audio onset latency: < 2 seconds (P95)
- App launch time (cold): < 2 seconds (P95)
- App launch time (warm): < 500ms (P95)
- Session resumption: < 2 seconds
- Crash-free rate: ≥ 99.5%
- Availability: 99.3% monthly

**Implementation Readiness:** Complete React Native Streaming Channel specification for agent-driven code generation.
