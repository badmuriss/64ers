<template>
  <div class="w-full max-w-2xl mx-auto p-6">
    <!-- Toggle -->
    <div class="flex justify-center items-center space-x-10 text-gray-400 mb-6">
      <span class="font-bold">Encode</span>
      <label class="relative inline-flex items-center cursor-pointer">
        <input id="mode-switch" type="checkbox" v-model="isDecodeMode" @change="clearAll" class="sr-only peer" />
        <div class="w-14 h-7 bg-gray-400 rounded-full peer-checked:bg-violet-500 transition-colors duration-300"></div>
        <div class="absolute left-1 top-1 w-5 h-5 bg-white rounded-full peer-checked:translate-x-7 transition-transform duration-300 shadow-md"></div>
      </label>
      <span class="font-bold">Decode</span>
    </div>

    <!-- Card -->
    <div class="bg-gray-800 p-6 rounded-xl shadow-lg space-y-6 overflow-y-auto">
      <!-- Encode Area -->
      <div v-show="!isDecodeMode" class="space-y-2">
        <div @click="triggerFileInput" @dragover.prevent @drop.prevent="handleDrop"
             class="h-50 border-4 border-dashed border-violet-500 rounded-lg flex items-center justify-center text-gray-400 hover:border-violet-400 transition-colors">
          <p>Drag & Drop or Click to Upload</p>
          <input type="file" ref="fileInput" @change="handleFile" class="hidden" />
        </div>
        <p class="text-violet-400 text-sm break-words">{{ fileInfo }}</p>
      </div>

      <!-- Decode Area -->
      <div v-show="isDecodeMode">
        <textarea v-model="base64Text" placeholder="Paste Base64 text..."
                  class="w-full h-49 bg-gray-700 text-gray-200 border border-gray-600 rounded-lg p-4 placeholder-gray-500 resize-none focus:ring-2 focus:ring-violet-400 transition duration-200">
        </textarea>
      </div>

      <!-- Actions -->
      <div class="flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0">
        <button @click="process"
                class="flex-1 py-3 rounded-lg font-semibold text-white bg-gradient-to-r from-violet-500 to-violet-600 hover:from-violet-600 hover:to-violet-700 transition-colors shadow-md">
          Process
        </button>
        <button @click="clearAll"
                class="flex-1 py-3 rounded-lg font-semibold text-white bg-gradient-to-r from-violet-500 to-violet-600 hover:from-violet-600 hover:to-violet-700 transition-colors shadow-md">
          Clear All
        </button>
      </div>

      <!-- Output -->
      <div v-show="!isDecodeMode && output" class="space-y-4">
        <p class="text-gray-200 font-medium">Output:</p>
        <textarea ref="outputTextarea" :value="output" readonly
                  class="w-full h-40 bg-gray-700 text-gray-200 border border-gray-600 rounded-lg p-4 font-mono resize-none">
        </textarea>
        <div class="flex justify-end">
          <button @click="copyOutput"
                  class="py-2 px-4 rounded-lg font-semibold text-white bg-violet-500 hover:bg-violet-600 transition-colors shadow">
            Copy to clipboard
          </button>
        </div>
      </div>

      <!-- File Details & Download -->
      <div v-if="downloadUrl && isDecodeMode" class="bg-gray-700 rounded-lg p-4 space-y-2">
        <p class="text-violet-400 font-medium">File Details:</p>
        <ul class="text-gray-300 text-sm list-disc list-inside space-y-1">
          <li>Name: {{ downloadName }}</li>
          <li>Format: {{ fileFormat }}</li>
          <li>Size: {{ fileSize }}</li>
        </ul>
      </div>

      <a v-if="downloadUrl" :href="downloadUrl" :download="downloadName"
         class="block w-full py-3 text-center rounded-lg font-semibold text-white bg-gradient-to-r from-violet-500 to-violet-600 hover:from-violet-600 hover:to-violet-700 transition-colors shadow-md">
        Download File
      </a>

      <!-- Preview -->
      <!-- <div v-if="preview" class="bg-gray-900 rounded-lg p-4">
        <h3 class="text-violet-400 font-medium mb-2">Preview:</h3>
        <div v-html="preview"></div>
      </div> -->

      <!-- Error & Progress -->
      <div class="flex flex-col items-center space-x-4">
        <div class="flex-1 bg-gray-800 rounded-full h-1 overflow-hidden">
          <div class="bg-violet-500 h-full transition-all" :style="{ width: progress + '%' }"></div>
        </div>
        <p class="text-red-500 flex-shrink-0">{{ errorMessage }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import mime from 'mime-types';

const isDecodeMode = ref(false)
const file = ref(null)
const fileInfo = ref('')
const base64Text = ref('')
const output = ref('')
const errorMessage = ref('')
const progress = ref(0)
const preview = ref('')
const downloadUrl = ref('')
const downloadName = ref('')
const fileInput = ref(null)
const outputTextarea = ref(null)
const fileFormat = ref('')
const fileSize = ref('')

function clearAll() {
  file.value = null
  fileInfo.value = ''
  base64Text.value = ''
  output.value = ''
  errorMessage.value = ''
  progress.value = 0
  preview.value = ''
  downloadUrl.value = ''
  downloadName.value = ''
  fileFormat.value = ''
  fileSize.value = ''
}

function triggerFileInput() {
  fileInput.value.click()
}

function handleFile(event) {
  const selectedFile = event.target.files[0]
  if (!selectedFile) return
  file.value = selectedFile
  fileInfo.value = `Name: ${selectedFile.name}, Type: ${selectedFile.type}, Size: ${selectedFile.size} bytes`
  clearOutput()
}

function handleDrop(e) {
  const droppedFile = e.dataTransfer.files[0]
  if (!droppedFile) return
  file.value = droppedFile
  fileInfo.value = `Name: ${droppedFile.name}, Type: ${droppedFile.type}, Size: ${droppedFile.size} bytes`
  clearOutput()
}

function clearOutput() {
  output.value = ''
  preview.value = ''
  errorMessage.value = ''
  progress.value = 0
  downloadUrl.value = ''
  downloadName.value = ''
  fileFormat.value = ''
  fileSize.value = ''
}

function guessMimeType(base64String) {
  const signatures = {
    '/9j/': 'image/jpeg',
    'iVBOR': 'image/png',
    'R0lGOD': 'image/gif',
    'JVBER': 'application/pdf',
    'UEsDB': 'application/zip',
    'AAABA': 'image/x-icon',
    'PD94b': 'application/xml',
    'SUQz': 'audio/mpeg',
    'T2dn': 'video/ogg',
    'f0VM': 'application/x-executable',
    '77u/': 'text/plain'
  }

  for (const sig in signatures) {
    if (base64String.startsWith(sig)) {
      return signatures[sig]
    }
  }

  return null;
}

async function process() {
  clearOutput()
  progress.value = 30

  if (!isDecodeMode.value) {
    if (!file.value) {
      errorMessage.value = 'Please select a file to encode.'
      return
    }
    try {
      const reader = new FileReader()
      reader.onload = () => {
        try {
          progress.value = 70
          const base64 = btoa(reader.result)
          output.value = base64
          progress.value = 100

          // if (file.value.type.startsWith('text/')) {
          //   preview.value = `<pre>${atob(base64)}</pre>`
          // } else if (file.value.type.startsWith('image/')) {
          //   preview.value = `<img src="data:${file.value.type};base64,${base64}" style="max-width: 100%;" />`
          // }
        } catch (error) {
          errorMessage.value = 'Error encoding file: ' + error.message
        }
      }
      reader.onerror = () => {
        errorMessage.value = 'Error reading file'
      }
      reader.readAsBinaryString(file.value)
    } catch (error) {
      errorMessage.value = 'Error processing file: ' + error.message
    }
  } else {
    if (!base64Text.value.trim()) {
      errorMessage.value = 'Please enter Base64 text to decode.'
      return
    }
    try {
      progress.value = 70
      const byteCharacters = atob(base64Text.value.trim())
      const byteArray = new Uint8Array([...byteCharacters].map(c => c.charCodeAt(0)))
      let mimeType = guessMimeType(base64Text.value.trim())
      if (!mimeType) {
        const ext = window.prompt(
          'Formato não reconhecido. Digite a extensão (ex: png, pdf, txt):'
        )?.trim().toLowerCase()
        if (ext) {
          const lookup = mime.lookup(`file.${ext}`)
          if (lookup) mimeType = lookup
        }
      }
      const blob = new Blob([byteArray], { type: mimeType })
      const extension = mime.extension(mimeType)
      downloadUrl.value = URL.createObjectURL(blob)
      downloadName.value = `decoded_file.${extension}`
      fileFormat.value = mimeType
      fileSize.value = formatFileSize(blob.size)
      output.value = 'File decoded successfully.'
      progress.value = 100
    } catch (error) {
      errorMessage.value = 'Invalid Base64 input'
    }
  }
}

function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

function copyOutput() {
  if (outputTextarea.value) {
    outputTextarea.value.select()
    navigator.clipboard.writeText(output.value)
  }
}
</script>

<style scoped>
</style>
