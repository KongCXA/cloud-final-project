<template>
  <div class="flex flex-col items-center justify-center relative">
    <div class="flex flex-col items-center gap-4">
      <button
        @click="isRecording ? stopRecording() : startRecording()"
        class="w-24 h-24 rounded-full flex items-center justify-center text-white transition-all shadow-2xl active:scale-80 z-10 cursor-pointer"
        :class="isRecording ? 'bg-red-500' : 'bg-white'"
      >
        <IconMicrophone v-if="!isRecording" class="w-10 h-10 text-black" />
        <span v-else class="material-icons text-4xl text-white font-bold">stop</span>
      </button>

      <p v-if="isRecording" class="text-red-500 font-medium animate-pulse">
        Listening... {{ formatTime(recordingTime) }}
      </p>
    </div>

    <Transition name="fade">
      <div
        v-if="showPopup"
        class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm flex items-center justify-center z-50 p-4"
      >
        <div
          class="bg-white rounded-3xl p-8 w-full max-w-sm shadow-2xl flex flex-col items-center gap-[20px]"
        >
          <h3 class="text-xl font-bold text-slate-800 mb-2">Review Your Record</h3>

          <p class="text-slate-500 text-sm text-center">
            Listen to your recording before sending it for analysis.
          </p>

          <audio :key="audioUrl" :src="audioUrl" controls class="w-full"></audio>

          <div class="flex gap-3 w-full">
            <button
              @click="showPopup = false"
              class="flex-1 py-3 px-4 rounded-xl font-medium text-slate-600 bg-slate-100 hover:bg-slate-200 transition-colors"
            >
              Retake
            </button>
            <button
              @click="handleUpload"
              :disabled="isUploading"
              class="flex-1 py-3 px-4 rounded-xl font-medium text-white bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 transition-colors"
            >
              {{ isUploading ? 'Sending...' : 'Send Record' }}
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, onUnmounted } from 'vue'
import IconMicrophone from './icons/IconMicrophone.vue'

const isRecording = ref(false)
const recordingTime = ref(0)
const audioUrl = ref(null)
const isUploading = ref(false)
const showPopup = ref(false)

let mediaRecorder = null
let audioChunks = []
let timerInterval = null

const startRecording = async () => {
  audioChunks = []
  audioUrl.value = null
  showPopup.value = false
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
    const options = { mimeType: 'audio/webm' }
    if (!MediaRecorder.isTypeSupported('audio/webm')) {
      options.mimeType = 'audio/ogg'
    }

    mediaRecorder = new MediaRecorder(stream, options)
    mediaRecorder.ondataavailable = (event) => {
      if (event.data.size > 0) audioChunks.push(event.data)
    }

    mediaRecorder.onstop = async () => {
      stream.getTracks().forEach((track) => track.stop())
      const blob = new Blob(audioChunks, { type: options.mimeType })

      if (blob.size > 0) {
        const buffer = await blob.arrayBuffer()
        const fixedBlob = new Blob([buffer], { type: options.mimeType })
        const reader = new FileReader()
        reader.readAsDataURL(fixedBlob)

        reader.onloadend = () => {
          audioUrl.value = reader.result
          showPopup.value = true

          // --- ส่วนที่เพิ่ม: สั่งรันอัตโนมัติทันที 1 รอบ ---
          // เราสร้าง Audio element ในเมมโมรี่ขึ้นมาสั่งรันเพื่อ Warm up buffer
          const autoPlay = new Audio(reader.result)
          autoPlay.play().catch((e) => {
            // บางครั้ง Browser อาจจะบล็อก Autoplay ถ้า User ยังไม่ได้คลิกอะไรเลย
            console.warn('Autoplay warm-up blocked or failed:', e)
          })
          // -------------------------------------------
        }
      }
    }

    mediaRecorder.start(1000)
    isRecording.value = true
    startTimer()
  } catch (err) {
    console.error('Microphone access error:', err)
    alert('Unable to access microphone. Please check your permissions.')
  }
}

const stopRecording = () => {
  mediaRecorder.stop()
  isRecording.value = false
  stopTimer()
}

const handleUpload = async () => {
  isUploading.value = true
  try {
    // 1. สร้าง Blob จาก chunks ที่อัดไว้
    const audioBlob = new Blob(audioChunks, { type: 'audio/webm' })

    // --- ส่วนสำหรับ TEST: บังคับโหลดไฟล์ลงเครื่อง ---
    const testLink = document.createElement('a')
    testLink.href = URL.createObjectURL(audioBlob)
    testLink.download = `test-upload-${Date.now()}.webm`
    document.body.appendChild(testLink)
    testLink.click()
    document.body.removeChild(testLink)
    // ------------------------------------------

    // จำลองการส่งข้อมูล (ในอนาคตคือส่วนที่จะส่งไป S3)
    console.log('Preparing to upload blob size:', audioBlob.size)

    // Simulate API delay
    await new Promise((resolve) => setTimeout(resolve, 1500))

    alert('Sent successfully! (Test file downloaded)')
    showPopup.value = false
  } catch (error) {
    console.error('Upload failed:', error)
    alert('Failed to send recording. Please try again.')
  } finally {
    isUploading.value = false
  }
}

const startTimer = () => {
  recordingTime.value = 0
  timerInterval = setInterval(() => {
    recordingTime.value++
  }, 1000)
}

const stopTimer = () => {
  clearInterval(timerInterval)
}

const formatTime = (seconds) => {
  const m = Math.floor(seconds / 60)
  const s = seconds % 60
  return `${m}:${s < 10 ? '0' : ''}${s}`
}

onUnmounted(() => {
  stopTimer()
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
