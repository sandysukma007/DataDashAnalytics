<template>
  <section class="card upload-section">
    <div class="card-header">
      <div class="header-content">
        <h3 class="card-title">
          <span class="title-icon">📤</span>
          Upload Dataset
        </h3>
        <p class="card-subtitle">Upload CSV, Excel, or JSON files for analysis</p>
      </div>
    </div>

    <div class="card-body">
      <!-- Upload Area -->
      <div
        class="upload-area"
        :class="{ 'dragover': isDragging, 'success': isUploaded }"
        @dragover.prevent="handleDragOver"
        @dragleave="handleDragLeave"
        @drop.prevent="handleDrop"
        @click="$refs.fileInput.click()"
      >
        <input
          type="file"
          hidden
          @change="uploadFile"
          ref="fileInput"
          accept=".csv,.xlsx,.xls,.json"
        />

        <div class="upload-content">
          <div class="upload-icon">
            <span v-if="!isUploaded" class="icon-upload">📁</span>
            <span v-else class="icon-success">✅</span>
          </div>

          <div class="upload-text">
            <h4 v-if="!isUploaded">Drop files here or click to upload</h4>
            <h4 v-else class="success-text">Upload Successful!</h4>
            <p v-if="!isUploaded">Supports: .csv, .xlsx, .json (Max 50MB)</p>
            <p v-else class="success-subtext">Your dataset is ready for analysis</p>
          </div>

          <button class="upload-btn" @click.stop="$refs.fileInput.click()">
            <span class="btn-icon">🔍</span>
            Browse Files
          </button>
        </div>

        <div v-if="isDragging" class="drag-overlay">
          <div class="drag-message">
            <span class="drag-icon">⬇️</span>
            <p>Drop your file here</p>
          </div>
        </div>
      </div>

      <!-- File Info -->
      <div v-if="currentFile" class="file-info">
        <div class="file-details">
          <span class="file-icon">📄</span>
          <div class="file-text">
            <p class="file-name">{{ currentFile.name }}</p>
            <p class="file-size">{{ formatFileSize(currentFile.size) }}</p>
          </div>
        </div>
        <div class="file-status">
          <span v-if="isUploaded" class="status-badge success">Ready</span>
          <span v-else class="status-badge uploading">Uploading...</span>
        </div>
      </div>

      <!-- Supported Formats -->
      <div class="formats-info">
        <h5 class="formats-title">Supported Formats</h5>
        <div class="formats-grid">
          <div class="format-item">
            <span class="format-icon">📊</span>
            <span class="format-name">CSV</span>
          </div>
          <div class="format-item">
            <span class="format-icon">📈</span>
            <span class="format-name">Excel</span>
          </div>
          <div class="format-item">
            <span class="format-icon">📝</span>
            <span class="format-name">JSON</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'

const props = defineProps({ isUploaded: Boolean })
const emit = defineEmits(['file-uploaded'])

const fileInput = ref(null)
const isDragging = ref(false)
const currentFile = ref(null)

const handleDragOver = () => {
  isDragging.value = true
}

const handleDragLeave = () => {
  isDragging.value = false
}

const handleDrop = (event) => {
  isDragging.value = false
  const file = event.dataTransfer.files[0]
  if (file) {
    uploadFile({ target: { files: [file] } })
  }
}

const uploadFile = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  currentFile.value = file

  const formData = new FormData()
  formData.append('file', file)

  try {
    const res = await fetch('http://127.0.0.1:8000/upload', {
      method: 'POST',
      body: formData
    })
    const data = await res.json()
    emit('file-uploaded', data)
  } catch (err) {
    console.error('Upload failed', err)
    currentFile.value = null
  }
}

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}
</script>

<style scoped>
/* Card Styling */
.card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  border: 1px solid #e5e7eb;
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}

.card-header {
  padding: 28px 32px;
  background: linear-gradient(135deg, #667eea15 0%, #764ba215 100%);
  border-bottom: 1px solid #e5e7eb;
}

.header-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.card-title {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 12px;
}

.title-icon {
  font-size: 28px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.card-subtitle {
  font-size: 15px;
  color: #6b7280;
  margin: 0;
  font-weight: 500;
}

/* Card Body */
.card-body {
  padding: 32px;
}

/* Upload Area */
.upload-area {
  position: relative;
  border: 2px dashed #d1d5db;
  border-radius: 12px;
  padding: 60px 40px;
  text-align: center;
  background: #f9fafb;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  margin-bottom: 24px;
}

.upload-area:hover {
  border-color: #667eea;
  background: rgba(102, 126, 234, 0.02);
  transform: translateY(-2px);
}

.upload-area.dragover {
  border-color: #667eea;
  background: rgba(102, 126, 234, 0.05);
  border-style: solid;
}

.upload-area.success {
  border-color: #10b981;
  background: rgba(16, 185, 129, 0.02);
}

.upload-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  position: relative;
  z-index: 2;
}

.upload-icon {
  font-size: 64px;
  margin-bottom: 8px;
  transition: transform 0.3s ease;
}

.upload-area:hover .upload-icon {
  transform: scale(1.1);
}

.icon-upload {
  opacity: 0.7;
}

.icon-success {
  color: #10b981;
  animation: bounceIn 0.6s ease;
}

@keyframes bounceIn {
  0% { transform: scale(0.5); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

.upload-text h4 {
  font-size: 20px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
}

.success-text {
  color: #10b981;
}

.upload-text p {
  font-size: 15px;
  color: #6b7280;
  margin: 0;
}

.success-subtext {
  color: #059669;
}

/* Upload Button */
.upload-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 14px 32px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.upload-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.upload-btn:active {
  transform: translateY(0);
}

.btn-icon {
  font-size: 18px;
}

/* Drag Overlay */
.drag-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(102, 126, 234, 0.1);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  z-index: 1;
}

.drag-message {
  text-align: center;
  animation: pulse 1.5s infinite;
}

.drag-icon {
  font-size: 48px;
  display: block;
  margin-bottom: 12px;
}

.drag-message p {
  font-size: 18px;
  font-weight: 600;
  color: #667eea;
  margin: 0;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

/* File Info */
.file-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  margin-bottom: 24px;
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.file-details {
  display: flex;
  align-items: center;
  gap: 16px;
}

.file-icon {
  font-size: 32px;
  color: #667eea;
}

.file-text {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.file-name {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0;
}

.file-size {
  font-size: 14px;
  color: #6b7280;
  margin: 0;
}

.file-status {
  display: flex;
  align-items: center;
}

.status-badge {
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.status-badge.success {
  background: rgba(16, 185, 129, 0.1);
  color: #10b981;
  border: 1px solid rgba(16, 185, 129, 0.2);
}

.status-badge.uploading {
  background: rgba(59, 130, 246, 0.1);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.2);
  animation: pulse 1.5s infinite;
}

/* Formats Info */
.formats-info {
  padding: 24px;
  background: #f9fafb;
  border-radius: 10px;
  border: 1px solid #e5e7eb;
}

.formats-title {
  font-size: 16px;
  font-weight: 600;
  color: #374151;
  margin: 0 0 16px 0;
  text-align: center;
}

.formats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

.format-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 16px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.format-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  border-color: #667eea;
}

.format-icon {
  font-size: 24px;
  opacity: 0.8;
}

.format-name {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

/* Responsive Design */
@media (max-width: 768px) {
  .card-header {
    padding: 24px;
  }

  .card-body {
    padding: 24px;
  }

  .upload-area {
    padding: 40px 24px;
  }

  .upload-icon {
    font-size: 56px;
  }

  .upload-text h4 {
    font-size: 18px;
  }

  .upload-btn {
    padding: 12px 24px;
    font-size: 15px;
  }

  .formats-grid {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .file-info {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .file-status {
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .card-header {
    padding: 20px;
  }

  .card-title {
    font-size: 20px;
  }

  .card-body {
    padding: 20px;
  }

  .upload-area {
    padding: 32px 20px;
  }

  .upload-icon {
    font-size: 48px;
  }

  .upload-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>