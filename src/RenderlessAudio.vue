<template>
    <div>
        <slot 
            :controls="expose.controls"
            :time="expose.time"
            :state="expose.state"
            :player="player">
        </slot>
        <audio 
            ref="audio" 
            :autoplay="autoplay"
            :loop="isLooping"
            :preload="preload"
            :crossorigin="crossorigin"
            :controls="native"
            v-show="native"
            v-on="$listeners">
            Your browser does not support the <code>audio</code> element.
            <source 
                v-for="(source, index) in sources" 
                :key="index"
                :src="source.src" 
                :type="source.type">
        </audio>
    </div>
</template>
<script>
export default {
    props: {
        src: {
            type: [String, Array],
            default: null,
            validator(value) {
                if(Array.isArray(value)) {
                    for(let source of value) {
                        if(!source.hasOwnProperty('src')) {
                            return false
                        }
                    }
                }

                return true
            }
        },
        muted: {
            type: Boolean,
            default: false
        },
        volume: {
            type: Number,
            default: 1,
            validator(val) {
                return val >= 0 && val <= 1
            }
        },
        autoplay: {
            type: Boolean,
            default: false
        },
        loop: {
            type: Boolean,
            default: false
        },
        playbackRate: {
            type: Number,
            default: 1
        },
        start: {
            type: Number,
            default: 0,
            validator(value) {
                return value >= 0
            }
        },
        preload: {
            type: String,
            default: 'auto',
            validator(value) {
                return ['none', 'metadata', 'auto'].indexOf(value) !== -1
            }
        },
        crossorigin: {
            type: String,
            default: null,
            validator(value) {
                return ['anonymous', 'use-credentials'].indexOf(value) !== -1
            }
        },
        native: {
            type: Boolean,
            default: false
        }
    },

    mounted() {
        this.player = this.$refs.audio
        this.setPlayerInitState()
        this.setupNativeEvents()

        this.$emit('init', {
            target: this.player,
            controls: this.expose.controls
        })
    },

    data() {
        return {
            playedTime: this.start,
            duration: 0,
            isReady: false,
            isPlaying: false,
            isMuted: this.muted,
            isLooping: this.loop,
            isLoaded: false,
            hasEnded: false,
            currentVolume: this.volume,
            speed: this.playbackRate,
            buffered: [],
            currentSource: null,
            player: null,
        }
    },

    computed: {
        sources() {
            if(Array.isArray(this.src)) {
                return this.src
            }

            return [{
                src: this.src
            }]
        },

        isWaiting() {
            return this.isPlaying && !this.isReady
        },

        progress() {
            if(!this.player) {
                return 0
            }

            return 100 / this.duration * this.playedTime
        },

        remainingTime() {
            return this.duration - this.playedTime
        },

        expose() {
            return {
                controls: {
                    play: this.play,
                    pause: this.pause,
                    stop: this.stop,
                    togglePlay: this.togglePlay,
                    setVolume: this.setVolume,
                    mute: this.mute,
                    unmute: this.unmute,
                    toggleMuted: this.toggleMuted,
                    seek: this.seek,
                    toggleLooping: this.toggleLooping,
                    setPlaybackRate: this.setPlaybackRate,
                    reload: this.reload,
                    download: this.download
                },

                time: {
                    played: this.playedTime,    
                    remaining: this.remainingTime,
                    duration: this.duration,
                    format: this.formatTime    
                },

                state: {
                    progress: this.progress,
                    buffered: this.buffered,
                    isReady: this.isReady,
                    isPlaying: this.isPlaying,
                    isMuted: this.isMuted,
                    isWaiting: this.isWaiting,
                    hasEnded: this.hasEnded,
                    volume: this.currentVolume,
                    isLooping: this.isLooping,
                    isLoaded: this.isLoaded,
                    playbackRate: this.speed,
                    currentSource: this.currentSource
                }
            }
        },
    },

    methods: {
        play() {
            this.player.play()
        },
        pause() {
            this.player.pause()
        },
        stop() {
            this.player.pause()
            this.player.currentTime = 0
            this.$emit('stopped')
        },
        togglePlay() {
            if (this.isPlaying) {
                this.pause()
            } else {
                this.play()
            }
        },
        setVolume(volume) {
            if(volume < 0 || volume > 1) {
                throw new Error('volume should be a number between 0 and 1')
            }

            this.player.volume = volume
        },
        mute() {
            this.player.muted = true
        },
        unmute() {
            this.player.muted = false
        },
        toggleMuted() {
            this.player.muted = !this.player.muted
        },
        seek(progress) {
            if(progress < 0 || progress > 100) {
                throw new Error('progress should be a number between 0 and 100 (percent)')
            }

            this.player.currentTime = this.player.duration / 100 * progress
        },
        toggleLooping() {
            this.isLooping = !this.isLooping
            this.$emit('update:loop', this.isLooping)
        },
        setPlaybackRate(value) {
            this.player.playbackRate = value
        },
        reload() {
            this.player.load()
        },
        download() {
            this.pause()
            window.open(this.currentSource, '_blank')
        },

        formatTime(seconds) {
            const gtOneHour = this.duration >= 3600
            return new Date(seconds * 1000).toISOString().substr(
                gtOneHour ? 11 : 14,
                gtOneHour ? 8 : 5
            )
        },

        setupNativeEvents() {
            this.player.addEventListener('timeupdate', e => {
                this.hasEnded = false
                this.isReady = true
                this.isPlaying = !e.target.paused
                this.playedTime = this.player.currentTime
            })

            this.player.addEventListener('durationchange', e => {
                this.duration = e.target.duration
            })

            this.player.addEventListener('play', () => {
                this.isPlaying = true
                this.updateBufferState()
            })

            this.player.addEventListener('pause', () => {
                this.isPlaying = false
            })

            this.player.addEventListener('ended', e => {
                this.isPlaying = false
                this.hasEnded = true
                
                if(this.isLooping) {
                    this.$emit('looped', e)
                }
            })

            this.player.addEventListener('emptied', () => {
                this.isPlaying = false
                this.hasEnded = false
                this.isReady = false
                this.currentSource = null
                this.buffered = []
                this.playedTime = this.start
                this.duration = 0
            })

            this.player.addEventListener('volumechange', e => {
                this.currentVolume = this.player.volume
                this.isMuted = e.target.muted

                this.$emit('update:volume', this.currentVolume)
                this.$emit('update:muted', this.isMuted)
            })

            this.player.addEventListener('canplay', e => {
                this.updateBufferState()
                this.isReady = true
                this.duration = e.target.duration
            })

            this.player.addEventListener('loadedmetadata', e => {
                this.isLoaded = true
                this.duration = e.target.duration
                this.currentSource = e.target.currentSrc
            })

            this.player.addEventListener('progress', () => {
                this.updateBufferState()
            })

            this.player.addEventListener('waiting', () => {
                this.isReady = false
            })

            this.player.addEventListener('stalled', () => {
                this.isReady = false
            })

            this.player.addEventListener('seeking', e => {
                this.playedTime = e.target.currentTime
                this.isReady = false
            })

            this.player.addEventListener('playing', () => {
                this.isReady = true
            })

            this.player.addEventListener('ratechange', e => {
                this.speed = e.target.playbackRate
                this.$emit('update:playbackRate', this.speed)
            })
        },

        setPlayerInitState() {
            this.player.muted = this.muted
            this.player.volume = this.volume
            this.player.playbackRate = this.speed
            this.player.currentTime = this.start
        },

        updateBufferState() {
            let timeRanges = this.player.buffered
            let duration = this.player.duration
            let buffered = []

            for (let i = 0; i < timeRanges.length; i++) {
                let start = 100 / duration * timeRanges.start(i)
                let end = 100 / duration * timeRanges.end(i)
                let width = end - start

                buffered.push({
                    start,
                    end,
                    width
                })
            }

            this.buffered = buffered
        }
    },

    watch: {
        playbackRate(value) {
            this.setPlaybackRate(value)
        },

        loop(value) {
            this.isLooping = value
        },

        muted(value) {
            this.player.muted = value
        },

        volume(value) {
            this.setVolume(value)
        }
    }
}
</script>
