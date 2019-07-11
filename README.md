# vue-renderless-audio

> A completely __renderless__ Vue component to help you build __custom audio players__ (using the HTML5 `<audio>` element).

## Install
```bash
# npm
$ npm install vue-renderless-audio

# yarn
$ yarn add vue-renderless-audio
```

## Setup
Register the component either globally or locally
```javascript
// Global registration
import Vue from 'vue'
import RenderlessAudio from 'vue-renderless-audio'
Vue.component('renderless-audio', RenderlessAudio)
```

```javascript
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

        <!-- your custom markup -->

    </template>    
</renderless-audio>
```

## License
[MIT](https://github.com/sagalbot/vue-select/blob/master/LICENSE.md)