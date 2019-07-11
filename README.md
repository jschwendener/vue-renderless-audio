# vue-renderless-audio
> A completely __renderless__ Vue (2.x) component to help you build __custom audio players__.

⚠️ This component is still in heavy development and should not be used in any production environment (or at your own risk)!

- [Install](#install)
- [Setup](#setup)
- [Example implementation](#example-implementation)
- [Documentation](#documentation)
    - [Props](#props)
    - [Slot](#slot)
    - [Events](#events)
    - [Access from outside](#access-player-controls-from-outside)
- [Browser support](#browser-support)
- [License](#license)

#### What does "renderless" mean?
A renderless component only provides __functionality__ and leaves you in full control over __markup and styling__.

That said, it's important to understand that this component __does not provide any UI__ whatsoever!

## Install
```bash
# npm
$ npm install vue-renderless-audio --save

# yarn
$ yarn add vue-renderless-audio
```

## Setup
Register the component either globally or locally
```js
// Global registration
import Vue from 'vue'
import RenderlessAudio from 'vue-renderless-audio'
Vue.component('renderless-audio', RenderlessAudio)
```

```js
// Local registration
import RenderlessAudio from 'vue-renderless-audio'
export default {
    components: {
        RenderlessAudio
    },
    
    ...
}
```

Basic usage in your markup looks like this
```html
<renderless-audio src="/my-audio-file.mp3">
    <template slot-scope="{ controls, time, state }">

        <!-- Start building your UI here -->

    </template>    
</renderless-audio>
```

## Example implementation
This snippet only shows the general idea behind this component and does not include any real styling yet:
```html
<template>
    <renderless-audio 
        :src="sources" 
        :autoplay="true"
        :loop="true"
        :volume.sync="globalVolume"
        @canplaythrough="doSomething()">

        <div slot-scope="{ controls, time, state }" class="my-music-player">
            <!--
                Progress bar as a range slider, bind progress as value and
                listen for the input event to seek to a different position -->
            <input 
                type="range" 
                :value="state.progress" 
                @input="controls.seek($event.target.value)"
                min="0" max="100" step="0.1" />

            <!-- 
                Control playback with provided functions in the 'controls' object.
                Conditionally display Play or Pause, depending on the current state -->
            <button v-if="!state.isPlaying" @click="controls.play()">Play</button>
            <button v-else @click="controls.pause()">Pause</button>

            <!-- Mute/Unmute control -->
            <button @click="controls.toggleMuted()">Mute/Unmute</button>

            <!-- 
                Volume control as a range slider, bind current volume as value and 
                listen for the input event to set it -->
            <input
                type="range" 
                :value="state.volume" 
                @input="controls.setVolume($event.target.value)" 
                min="0" max="1" step="0.1" />

        </div>

    </renderless-audio>
</template>
<script>
import RenderlessAudio from 'vue-renderless-audio'
export default {
    components: {
        RenderlessAudio
    },

    data() {
        return {
            // Control volume globally from parent (using .sync-modifier on prop)
            globalVolume: 0.5,
        }
    }

    computed: {
        sources() {
            return [
                { src: '/some-audio-file.mp3', type: 'audio/mp3' },
                { src: '/some-audio-file.wav', type: 'audio/wav' }
            ]
        }
    }
}
</script>
```

## Documentation
### Props
__src__ _(required)_<br>
The source of your audio file as a string or as an object array for multiple file types (containing _src_ and _type_ properties).

- type: `String` or `Array`
- default: `null`

```html
<!-- Single file type -->
<renderless-audio src="/some-audio-file.mp3"></renderless-audio>

<!-- Multiple file types -->
<renderless-audio :src="[
    { src: '/some-audio-file.mp3', type: 'audio/mp3' },
    { src: '/some-audio-file.ogg', type: 'audio/ogg' }
]"></renderless-audio>
```

__muted__<br>
Sets player muted attribute
- type: `Boolean` 
- default: `false`

__autoplay__<br>
Sets player autoplay attribute
- type: `Boolean` 
- default: `false`
> Be aware of browsers [autoplay policies](https://developers.google.com/web/updates/2017/09/autoplay-policy-changes). In most modern browsers, some kind of user action (eg. click) is required to autoplay media.

__loop__<br>
Sets player loop attribute
- type: `Boolean` 
- default: `false`

__volume__<br>
Sets player volume, must be a floating digit between 0 and 1. (eg. `0.75` stands for 75% volume)
- type: `Number` 
- default: `1`

__playbackRate__<br>
Sets the speed at which the media is being played back. The normal playback rate is multiplied by this value to obtain the current rate, so a value of `1.0` indicates normal speed.
- type: `Number` 
- default: `1`

__start__<br>
Offsets the position where playback begins (in seconds). Must be greater than or equals `0`.
- type: `Number`
- default: `0`

__preload__<br>
Sets the player preload attribute. Hints to the browser how or if to preload data. Must be `'none'` `'metadata'` or `'auto'`
- type: `String``
- default: `'auto'`

__crossorigin__<br>
Sets the player crossorigin attribute. Lets you configure the [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) requests for the element's fetched data. Must be `'anonymous'` or `'use-credentials'`
- type: `String``
- default: `null`

__native__<br>
Shows the native audio player if set to true. Most likely you would only want to use this for development and debugging purposes.
- type: `Boolean` 
- default: `false`

#### Sync props back to parent
For some props it's useful to have "two-way-binding". You can use the `.sync` modifier to automatically listen and update the parent data property. (Vue documentation: [.sync modifier](https://vuejs.org/v2/guide/components-custom-events.html#sync-Modifier))
```html
<renderless-audio 
    src="/my-audio-file.mp3" 
    :muted.sync="noSound">
</renderless-audio>
```
```js
export default {
    ...
    data() {
        return {
            /* will be passed as a prop but updates itself 
               if 'muted' state inside component changes */
            noSound: true
        }
    }
    ...
}
```

### Slot
All your markup goes into the __default__ slot.
```html
<renderless-audio src="/my-audio-file.mp3">
    <template slot-scope="{ controls, time, state }">

        <!-- Default slot: Your custom markup -->

    </template>    
</renderless-audio>
```

The slot scope exposes objects `controls`, `time`, `state` and `el` containing all necessary info and methods to control playback:

#### controls
Method | Description
--- | ---
`controls.play()` | Starts playback
`controls.pause()` | Pauses playback
`controls.stop()` | Pauses playback and sets the position back to the beginning
`controls.togglePlay()` | Starts or pauses playback, depending on the current state
`controls.mute()` | Mutes audio
`controls.unmute()` | Unmutes audio
`controls.toggleMuted()` | Mutes or unmutes audio, depending on the current state
`controls.setVolume(volume)` | Sets the volume (`volume`: floating number between 0 and 1)
`controls.seek(progress)` | Sets playback position to the given percentage value (`progress`: Number between 0 and 100)
`controls.toggleLooping()` | Activates or deactivates looping of the audio, depending on the current state
`controls.setPlaybackRate(rate)` | Sets playback rate
`controls.reload()` | Stops playback and reloads the audio file
`controls.download()` | Opens/downloads the audio source file

#### time
Property / Method | Type | Description
--- | --- | ---
`time.played` | `Number` | The time played in seconds
`time.remaining` | `Number` | The time remaining in seconds
`time.duration` | `Number` | The total duration of the audio file in seconds
`time.format(seconds)` | `Function` | Helper function to convert seconds into a human readable format (`hh:mm:ss`)

#### state
Property | Type | Description
--- | --- | ---
`state.isReady` | `Boolean` | Is true if player is ready for playback (first frames loaded)
`state.isWaiting` | `Boolean` | Is true if player is buffering and waiting for data
`state.isMuted` | `Boolean` | Is true if player is muted
`state.isLooping` | `Boolean` | Is true if looping is active
`state.isLoaded` | `Boolean` | Is true if the metadata is loaded
`state.hasEnded` | `Boolean` | Is true if playback has reached the end and is not looping
`state.progress` | `Number` | Current progress of playback in percent
`state.volume` | `Number` | Current volume
`state.playbackRate` | `Number` | Current playback speed
`state.buffered` | `Array` | An array of all buffered timeranges, containing info about _start_, _end_ and _width_ (length) of buffered range. All numbers are percentages.
`state.currentSource`| `String`| Currently loaded source file

#### el
The `el` object allows direct acces to the `<audio>` element.

### Events
All native DOM events ([Media events](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Media_events)) of the underlying `<audio>` element are passed to the root of the component.
Which means you can listen for native events directly on the `<renderless-audio>` component (Without the `.native` modifier!).
```html
<renderless-audio
    src="/my-audio-file.mp3"
    @loadeddata="doSomething()"
    @canplaythrough="doSomethingElse()">
    ...
</renderless-audio>
```

__Additional events__<br>
In addition to the native media events, the component emits a few more useful events.

Event | Data | Description
--- | --- | ---
`init`| `Object` | Fires when component is mounted and has been initialized.
`source-changed`| `Object` | Fires when the source file changed. (Also passes the new source as _source_ to the listener)
`stopped`| `Object` | Fires when playback has stopped after calling `controls.stop()`
`looped`| `Object` | Fires when playback has reached the end and looping is active

All of these events pass an object with (at least) the _target_ and _controls_ fields to the listener:
```js
{
    // The `<audio>` element itself
    target: HTMLAudioElement

    // All functions described in the slot-scope 'controls' section
    controls: Array<Function>

    ...
}
```

This can be very useful if you want to directly react to certain events.
In the following example we're immediately starting playback after the source has changed and muting the sound after the first loop:
```html
<renderless-audio
    :src="myDynamicSource"
    @source-changed="$event.controls.play()"
    @looped="$event.controls.mute()">
    ...
</renderless-audio>
```

__Prop update events__<br>
You can listen to prop update events with `@update:propName` ('propName' being the actual name of the prop) or simply use the `.sync` modifier. (See: [Sync props back to parent](#sync-props-back-to-parent))

### Access player controls from outside
Usually, all method calls to the player should happen inside your markup in the default slot.
Although _not really recommended_, there are a few ways to call the control methods from outside:

__Control methods are passed on the `init` event__
```html
<template>
    <renderless-audio
        @init="playerControls = $event.controls"
        src="/some-audio-file.mp3">
    </renderless-audio>

    <button @click="someActionOnParent()">Button outside slot</button>
</template>
<script>
export default {
    ...

    data() {
        return {
            playerControls: null
        }
    }

    methods: {
        someActionOnParent() {
            this.playerControls.seek(25)
            this.playerControls.play()
        }
    }

    ...
}
</script>
```

or

__Make a reference to the component__
```html
<template>
    <renderless-audio
        ref="audioPlayer"
        src="/some-audio-file.mp3">
    </renderless-audio>

    <button @click="someActionOnParent()">Button outside slot</button>
</template>
<script>
export default {
    ...

    methods: {
        someActionOnParent() {
            this.$refs.audioPlayer.seek(25)
            this.$refs.audioPlayer.play()
        }
    }

    ...
}
</script>
```

## Browser Support
This component has not been properly tested yet, but I'm planning to target all browsers supporting Vue 2.x and the `<audio>` element.

## License
[MIT](https://github.com/sagalbot/vue-select/blob/master/LICENSE.md)
