# rn-incall-manager

# !! NOTICE !!

This repository is created so I can easily update the code to my needs; some decisions may not be to your liking so it might be best to just use the original: [react-native-incall-manager](https://github.com/react-native-webrtc/react-native-incall-manager)

# -----------------

Handling media-routes/sensors/events during a audio/video chat on React Native

## Purpose:

The purpose of this module is to handle actions/events during a phone call (audio/video) on `react-native`, ex:

- Manage devices events like wired-headset plugged-in state, proximity sensors and expose functionalities to javascript.
- Automatically route audio to proper devices based on events and platform API.
- Toggle speaker or microphone on/off, toggle flashlight on/off
- Play ringtone/ringback/dtmftone

Basically, it is a telecommunication module which handles most of the requirements when making/receiving/talking with a call.

## TODO / Contribution Wanted:

- Make operations run on the main thread. ( iOS/Android )
- Fix iOS audio shared instance singleton conflict with internal webrtc.
- Detect hardware button press event and react to it.  
  ex: press bluetooth button, send an event to JS to answer/hangup.  
  ex: press power button to mute incoming ringtone.
- Use config-based to decide which event should start and report. maybe control behavior as well.
- Flash API on Android.

## Installation:

**From npm package**: `npm install rn-incall-manager`  
**From git package**: `npm install git://github.com/marqroldan/rn-incall-manager.git`

===================================================

#### Optional sound files on android

If you want to use bundled ringtone/ringback/busytone sound instead of system sound,  
put files in `android/app/src/main/res/raw`  
and rename file correspond to sound type:

```
incallmanager_busytone.mp3
incallmanager_ringback.mp3
incallmanager_ringtone.mp3
```

On android, as long as your file extension supported by android, this module will load it.

===================================================

#### Optional sound files on iOS

If you want to use bundled ringtone/ringback/busytone sound instead of system sound

1. Add files into your_project directory under your project's xcodeproject root. ( or drag into it as described above. )
2. Check `copy file if needed`
3. Make sure filename correspond to sound type:

```
incallmanager_busytone.mp3
incallmanager_ringback.mp3
incallmanager_ringtone.mp3
```

On ios, we only support mp3 files currently.

## Usage:

This module implements a basic handle logic automatically, just:

```javascript
import InCallManager from "rn-incall-manager";

// --- start manager when the chat start based on logics of your app
// On Call Established:
InCallManager.start({ media: "audio" }); // audio/video, default: audio

// ... it will also register and emit events ...

// --- On Call Hangup:
InCallManager.stop();
// ... it will also remove event listeners ...
```

If you want to use ringback:

```javascript
// ringback is basically for OUTGOING call. and is part of start().

InCallManager.start({ media: "audio", ringback: "_BUNDLE_" }); // or _DEFAULT_ or _DTMF_
//when callee answered, you MUST stop ringback explicitly:
InCallManager.stopRingback();
```

If you want to use busytone:

```javascript
// busytone is basically for OUTGOING call. and is part of stop()
// If the call failed or callee are busing,
// you may want to stop the call and play busytone
InCallManager.stop({ busytone: "_DTMF_" }); // or _BUNDLE_ or _DEFAULT_
```

If you want to use ringtone:

```javascript
// ringtone is basically for INCOMING call. it's independent to start() and stop()
// if you receiving an incoming call, before user pick up,
// you may want to play ringtone to notify user.
InCallManager.startRingtone("_BUNDLE_"); // or _DEFAULT_ or system filename with extension

// when user pickup
InCallManager.stopRingtone();
InCallManager.start();

// or user hangup
InCallManager.stopRingtone();
InCallManager.stop();
```

Also can interact with events if you want:
See API section.

```javascript
import { DeviceEventEmitter } from "react-native";

DeviceEventEmitter.addListener("Proximity", function (data) {
  // --- do something with events
});
```

## Automatic Basic Behavior:

**On start:**

- Store current settings, set KeepScreenOn flag = true, and register some event listeners.
- If media type is `audio`, route voice to earpiece, otherwise route to speaker.
- Audio will enable proximity sensor which is disabled by default if media=video
- When proximity detects user close to screen, turn off screen to avoid accident touch and route voice to the earpiece.
- When newly external device plugged, such as wired-headset, route audio to an external device.
- Optional play ringback

**On stop:**

- Set KeepScreenOn flag = false, remote event listeners, restore original user settings.
- Optionally play busytone

## Custom Behavior:

You can customize behavior using API/events exposed by this module. See `API` section.

Note: iOS only supports `auto` currently.

## API:

**Methods**

| Method                                                                                              | android |   ios   | description                                                                                                                                                                                                                                                                                                   |
| :-------------------------------------------------------------------------------------------------- | :-----: | :-----: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| start(`{media: ?string, auto: ?boolean, ringback: ?string}`)                                        | :smile: | :smile: | start incall manager.</br> ringback accept non-empty string or it won't play</br>default: `{media:'audio', auto: true, ringback: ''}`                                                                                                                                                                         |
| stop(`{busytone: ?string}`)                                                                         | :smile: | :smile: | stop incall manager</br> busytone accept non-empty string or it won't play</br> default: `{busytone: ''}`                                                                                                                                                                                                     |
| turnScreenOn()                                                                                      | :smile: | :rage:  | force turn screen on                                                                                                                                                                                                                                                                                          |
| turnScreenOff()                                                                                     | :smile: | :rage:  | force turn screen off                                                                                                                                                                                                                                                                                         |
| setKeepScreenOn(`enable: ?boolean`)                                                                 | :smile: | :smile: | set KeepScreenOn flag = true or false</br>default: false                                                                                                                                                                                                                                                      |
| setSpeakerphoneOn(`enable: ?boolean`)                                                               | :smile: | :rage:  | toggle speaker ON/OFF once. but not force</br>default: false                                                                                                                                                                                                                                                  |
| setForceSpeakerphoneOn(`flag: ?boolean`)                                                            | :smile: | :smile: | true -> force speaker on</br> false -> force speaker off</br> null -> use default behavior according to media type</br>default: null                                                                                                                                                                          |
| setMicrophoneMute(`enable: ?boolean`)                                                               | :smile: | :rage:  | mute/unmute micophone</br>default: false</br>p.s. if you use webrtc, you can just use `track.enabled = false` to mute                                                                                                                                                                                         |
| async getAudioUriJS()                                                                               | :smile: | :smile: | get audio Uri path. this would be useful when you want to pass Uri into another module.                                                                                                                                                                                                                       |
| startRingtone(`ringtone: string, ?vibrate_pattern: array, ?ios_category: string, ?seconds: number`) | :smile: | :smile: | play ringtone. </br>`ringtone`: '_DEFAULT_' or '_BUNDLE_'</br>`vibrate_pattern`: same as RN, but does not support repeat</br>`ios_category`: ios only, if you want to use specific audio category</br>`seconds`: android only, specify how long do you want to play rather than play once nor repeat. in sec. |
| stopRingtone()                                                                                      | :smile: | :smile: | stop play ringtone if previous started via `startRingtone()`                                                                                                                                                                                                                                                  |
| stopRingback()                                                                                      | :smile: | :smile: | stop play ringback if previous started via `start()`                                                                                                                                                                                                                                                          |
| setFlashOn(`enable: ?boolean, brightness: ?number`)                                                 | :rage:  | :smile: | set flash light on/off                                                                                                                                                                                                                                                                                        |
| async getIsWiredHeadsetPluggedIn()                                                                  | :rage:  | :smile: | return wired headset plugged in state                                                                                                                                                                                                                                                                         |

**Events**

| Event                | android |   ios   | description                                                                                                                                                                                                      |
| :------------------- | :-----: | :-----: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 'Proximity'          | :smile: | :smile: | proximity sensor detected changes.<br>data: `{'isNear': boolean}`                                                                                                                                                |
| 'WiredHeadset'       | :smile: | :smile: | fire when wired headset plug/unplug<br>data: `{'isPlugged': boolean, 'hasMic': boolean, 'deviceName': string }`                                                                                                  |
| 'NoisyAudio'         | :smile: | :rage:  | see [andriod doc](http://developer.android.com/reference/android/media/AudioManager.html#ACTION_AUDIO_BECOMING_NOISY).<br>data: `null`                                                                           |
| 'MediaButton'        | :smile: | :rage:  | when external device controler pressed button. see [android doc](http://developer.android.com/reference/android/content/Intent.html#ACTION_MEDIA_BUTTON) <br>data: `{'eventText': string, 'eventCode': number }` |
| 'onAudioFocusChange' | :smile: | :rage:  | see [andriod doc](<http://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener.html#onAudioFocusChange(int)>) <br>data: `{'eventText': string, 'eventCode': number }`           |

**NOTE: platform OS always has the final decision, so some toggle API may not work in some cases
be careful when customizing your own behavior**

## LICENSE:

**[ISC License](https://opensource.org/licenses/ISC)** ( functionality equivalent to **MIT License** )

## Original Author:

[![zxcpoiu](https://github.com/zxcpoiu.png)](https://github.com/zxcpoiu)
