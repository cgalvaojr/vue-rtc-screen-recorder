<script>
import RecordRTC from 'recordrtc'
import toast from 'toast'

export default {
  name: 'screen-recorder',
  langPrefix: 'components.recorder',

  data () {
    return {
      recorder: null,
      stream: null,
      video: null,
      recording: false,
      startedBefore: false
    }
  },
  props: {
    isSubmitting: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    startRecord () {
      this.captureScreen(screen => {
        this.video.srcObject = screen
        this.recorder = RecordRTC(screen, {
          type: 'video'
        })
        this.recorder.startRecording()
        this.recorder.screen = screen
        this.startedBefore = this.recording = true
      })
    },
    stopRecord () {
      this.recorder.stopRecording(this.stopRecordingCallback)
      this.recording = false
    },
    async stopRecordingCallback () {
      this.video.src = this.video.srcObject = null
      const blob = this.recorder.getBlob()
      this.video.src = URL.createObjectURL(blob)
      this.video.blob = blob
      this.recorder.screen.stop()
      this.recorder.destroy()
      // this.recorder = null
    },
    getStream () {
      if (navigator.mediaDevices.getDisplayMedia) {
        return navigator.mediaDevices.getDisplayMedia({
          video: {
            displaySurface: 'monitor',
            logicalSurface: true,
            cursor: 'always'
          }
        })
      }
    },
    invokeGetDisplayMedia (success, error) {
      let displaymediastreamconstraints = {
        video: {
          displaySurface: 'monitor',
          logicalSurface: true,
          cursor: 'always'
        }
      }
      displaymediastreamconstraints = {
        video: true
      }

      if (navigator.mediaDevices.getDisplayMedia) {
        navigator.mediaDevices.getDisplayMedia(displaymediastreamconstraints).then(success).catch(error)
      } else {
        navigator.getDisplayMedia(displaymediastreamconstraints).then(success).catch(error)
      }
    },
    addStreamStopListener (stream, callback) {
      stream.addEventListener('ended', () => {
        callback()
        callback = () => {}
      }, false)
      stream.addEventListener('inactive', () => {
        callback()
        callback = () => {}
      }, false)
      stream.getTracks().forEach(track => {
        track.addEventListener('ended', () => {
          callback()
          callback = () => {}
        }, false)
        track.addEventListener('inactive', () => {
          callback()
          callback = () => {}
        }, false)
      })
    },
    captureScreen (callback) {
      this.invokeGetDisplayMedia(screen => {
        this.addStreamStopListener(screen, () => {
          this.$refs.stopRecord.click()
        })
        callback(screen)
      }, error => {
        console.error(error)
        toast.error(`${this.$lang('captureError')}\n${error}`)
      })
    }
  },
  mounted () {
    this.video = this.$refs.video
  }
}
</script>

<template>
  <div class="h-recorder-wrapper">
    <q-card class="row">
      <div class="col-12">
        <video ref="video" style="width: 100%" loop controls autoplay playsinline />
      </div>
      <q-card-actions class="col-12 q-mt-sm q-mb-sm" align="center">
        <q-btn
          ref="startRecord"
          @click="startRecord"
          :label="$lang('record')"
          :disabled="isSubmitting"
          v-show="!recording && !startedBefore"
          size="md"
          outline color="red"
          icon="fiber_manual_record"
        >
          <q-tooltip>{{ $lang('record') }}</q-tooltip>
        </q-btn>
        <q-btn
          ref="recordAgain"
          @click="startRecord"
          :label="$lang('recordAgain')"
          v-show="!recording && startedBefore"
          :disabled="isSubmitting"
          size="md"
          outline color="red"
          icon="replay"
        >
          <q-tooltip>{{ $lang('recordAgain') }}</q-tooltip>
        </q-btn>
        <q-btn
          ref="stopRecord"
          v-show="recording"
          @click="stopRecord"
          :label="$lang('finish')"
          size="md"
          outline
          color="red"
          icon="eva-stop-circle"
        />
        <q-btn
          v-show="!recording && startedBefore"
          @click="$emit('handleRecordedVideo', video.blob)"
          :label="$lang('submit')"
          size="md"
          color="green"
          icon="publish"
          :loading="isSubmitting"
        >
          <q-tooltip>{{ $lang('submit') }}</q-tooltip>
        </q-btn>
      </q-card-actions>
    </q-card>
  </div>
</template>
