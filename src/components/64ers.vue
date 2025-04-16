<template>
  <div class="container">
    <div class="toggle-switch">
      <label>
        <input type="checkbox" v-model="isDecodeMode" @change="clearAll" />
        <span class="encode">Encode</span>
        <span class="decode">Decode</span>
      </label>
    </div>

    <div class="card">
      <div v-show="!isDecodeMode" id="encode-input-area">
        <div class="drag-drop-area" @click="triggerFileInput" @dragover.prevent @drop.prevent="handleDrop">
          <p>Drag & Drop File Here or Click to Upload</p>
          <input type="file" ref="fileInput" @change="handleFile" hidden />
        </div>
        <div class="file-info">{{ fileInfo }}</div>
      </div>

      <div v-show="isDecodeMode" id="decode-input-area">
        <textarea v-model="base64Text" placeholder="Enter Base64 text here..."></textarea>
      </div>

      <div class="buttons">
        <button @click="process">Process</button>
        <button @click="copyOutput">Copy to Clipboard</button>
        <button @click="clearAll">Clear All</button>
      </div>

      <div class="output">
        <p>Output:</p>
        <textarea :value="output" readonly></textarea>
      </div>

      <div class="preview" v-html="preview"></div>
      <div class="error-message">{{ errorMessage }}</div>
      <div class="progress-bar" :style="{ width: progress + '%' }"></div>

      <div class="download-link" v-if="downloadUrl">
        <a :href="downloadUrl" :download="downloadName">Download File</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

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
}

function process() {
  clearOutput()
  progress.value = 30

  if (!isDecodeMode.value) {
    if (!file.value) {
      errorMessage.value = 'Please select a file to encode.'
      return
    }
    const reader = new FileReader()
    reader.onload = () => {
      try {
        progress.value = 70
        output.value = btoa(reader.result)
        progress.value = 100
      } catch {
        errorMessage.value = 'Error encoding file.'
      }
    }
    reader.readAsBinaryString(file.value)
  } else {
    if (!base64Text.value.trim()) {
      errorMessage.value = 'Please enter Base64 text to decode.'
      return
    }
    try {
      const byteCharacters = atob(base64Text.value.trim())
      const byteArray = new Uint8Array([...byteCharacters].map(c => c.charCodeAt(0)))
      const blob = new Blob([byteArray])
      downloadUrl.value = URL.createObjectURL(blob)
      downloadName.value = 'decoded_file'
      output.value = 'File decoded successfully.'
      progress.value = 100
      preview.value = ''
    } catch {
      errorMessage.value = 'Invalid Base64 input.'
    }
  }
}

function copyOutput() {
  navigator.clipboard.writeText(output.value)
}
</script>

<style scoped>
body {
    margin: 0;
    padding: 0;
    background-color: #121212;
    font-family: 'Roboto', sans-serif;
    color: #CCCCCC;
}

.container {
    padding: 20px;
}

.toggle-switch {
    text-align: center;
    margin-bottom: 20px;
}

.toggle-switch input {
    display: none;
}

.toggle-switch label {
    position: relative;
    display: inline-block;
    width: 60px;
    height: 30px;
    background-color: #555555;
    border-radius: 15px;
    cursor: pointer;
}

.toggle-switch label::after {
    content: '';
    position: absolute;
    top: 3px;
    left: 3px;
    width: 24px;
    height: 24px;
    background-color: #FFFFFF;
    border-radius: 50%;
    transition: 0.3s;
}

.toggle-switch input:checked + label {
    background-color: #8A2BE2;
}

.toggle-switch input:checked + label::after {
    transform: translateX(30px);
}

.toggle-switch .encode,
.toggle-switch .decode {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    font-size: 16px;
    width: 60px;
    text-align: center;
    pointer-events: none;
}

.toggle-switch .encode {
    left: -70px;
}

.toggle-switch .decode {
    right: -70px;
}

.card {
    background-color: #1E1E1E;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.5);
    max-width: 600px;
    margin: 0 auto;
}

.drag-drop-area {
    width: 100%;
    height: 200px;
    border: 2px dashed #8A2BE2;
    border-radius: 5px;
    text-align: center;
    padding-top: 60px;
    margin-bottom: 15px;
    position: relative;
    color: #CCCCCC;
    cursor: pointer;
}

.drag-drop-area p {
    margin: 0;
    font-size: 16px;
}

.drag-drop-area input {
    position: absolute;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: pointer;
}

.file-info {
    font-size: 14px;
    margin-bottom: 15px;
}

#text-input,
#output-text {
    width: 100%;
    background-color: #2A2A2A;
    color: #FFFFFF;
    border: 1px solid #8A2BE2;
    border-radius: 5px;
    padding: 10px;
    font-size: 16px;
    resize: none;
    margin-bottom: 15px;
}

#text-input {
    height: 100px;
}

#output-text {
    height: 150px;
}

.buttons {
    display: flex;
    gap: 10px;
    margin-bottom: 15px;
}

.buttons button {
    background-color: #8A2BE2;
    color: #FFFFFF;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.3s;
}

.buttons button:hover {
    background-color: #9B30FF;
}

.buttons button:active {
    background-color: #7A1FA3;
}

.buttons button:disabled {
    background-color: #555555;
    color: #888888;
    cursor: not-allowed;
}

.output p {
    margin-bottom: 5px;
}

.preview {
    margin-top: 15px;
}

.preview img,
.preview iframe,
.preview video,
.preview audio {
    max-width: 100%;
    border-radius: 5px;
}

.error-message {
    color: #FF6347;
    font-size: 14px;
    margin-top: 10px;
}

.progress-bar {
    height: 5px;
    background-color: #8A2BE2;
    width: 0%;
    transition: width 0.3s;
}

.color-picker {
    position: fixed;
    top: 20px;
    right: 20px;
}

#color-picker-btn {
    background: none;
    border: none;
    font-size: 16px;
    cursor: pointer;
    color: #FFFFFF;
}

.color-options {
    display: none;
    margin-top: 10px;
}

.color-swatch {
    display: inline-block;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    margin-right: 5px;
    cursor: pointer;
}

.color-swatch:nth-child(1) { background-color: #8A2BE2; }
.color-swatch:nth-child(2) { background-color: #1E90FF; }
.color-swatch:nth-child(3) { background-color: #32CD32; }
.color-swatch:nth-child(4) { background-color: #FF4500; }

.info-modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(18, 18, 18, 0.9);
    align-items: center;
    justify-content: center;
}

.modal-content {
    background-color: #1E1E1E;
    padding: 20px;
    border-radius: 10px;
    position: relative;
    text-align: center;
    max-width: 400px;
    width: 80%;
}

.close-modal {
    position: absolute;
    top: 10px;
    right: 10px;
    cursor: pointer;
    color: #FFFFFF;
}

.download-link {
    text-align: center;
    margin-top: 15px;
}

.download-link a {
    background-color: #8A2BE2;
    color: #FFFFFF;
    text-decoration: none;
    padding: 10px 20px;
    border-radius: 5px;
    display: inline-block;
    font-weight: bold;
    transition: background-color 0.3s;
}

.download-link a:hover {
    background-color: #9B30FF;
}

.download-link a:active {
    background-color: #7A1FA3;
}

@media (max-width: 480px) {
    .container {
        padding: 10px;
    }

    .buttons {
        flex-direction: column;
    }

    .buttons button {
        width: 100%;
    }
}
</style>
