<template>
  <div class="export-menu">
    <div class="export-header">
      <h3 class="export-title">
        <span class="export-icon">📤</span>
        Export Data
      </h3>
      <p class="export-subtitle">Export your dataset in various formats</p>
    </div>

    <div class="export-stats" v-if="filteredRows && filteredRows.length">
      <div class="stat-item">
        <span class="stat-icon">📊</span>
        <div class="stat-content">
          <div class="stat-value">{{ filteredRows.length }}</div>
          <div class="stat-label">Rows to Export</div>
        </div>
      </div>
      <div class="stat-item">
        <span class="stat-icon">📋</span>
        <div class="stat-content">
          <div class="stat-value">{{ columns.length }}</div>
          <div class="stat-label">Columns</div>
        </div>
      </div>
    </div>

    <div class="export-grid">
      <div class="export-card" @click="exportData('csv')" :class="{ 'disabled': !hasData }">
        <div class="card-icon">
          <span class="icon">📄</span>
        </div>
        <div class="card-content">
          <h4 class="card-title">CSV Format</h4>
          <p class="card-description">Comma-separated values, ideal for spreadsheets</p>
          <div class="card-features">
            <span class="feature">Universal</span>
            <span class="feature">Lightweight</span>
            <span class="feature">Excel Ready</span>
          </div>
        </div>
        <button class="export-btn" :disabled="!hasData">
          Export CSV
        </button>
      </div>

      <div class="export-card" @click="exportData('excel')" :class="{ 'disabled': !hasData }">
        <div class="card-icon">
          <span class="icon">📊</span>
        </div>
        <div class="card-content">
          <h4 class="card-title">Excel Format</h4>
          <p class="card-description">Microsoft Excel workbook with formatting</p>
          <div class="card-features">
            <span class="feature">Formatted</span>
            <span class="feature">Multi-sheet</span>
            <span class="feature">Professional</span>
          </div>
        </div>
        <button class="export-btn" :disabled="!hasData">
          Export Excel
        </button>
      </div>

      <div class="export-card" @click="exportData('json')" :class="{ 'disabled': !hasData }">
        <div class="card-icon">
          <span class="icon">🔤</span>
        </div>
        <div class="card-content">
          <h4 class="card-title">JSON Format</h4>
          <p class="card-description">Structured data format for web applications</p>
          <div class="card-features">
            <span class="feature">Structured</span>
            <span class="feature">API Ready</span>
            <span class="feature">Developer Friendly</span>
          </div>
        </div>
        <button class="export-btn" :disabled="!hasData">
          Export JSON
        </button>
      </div>

      <div class="export-card" @click="exportData('pdf')" :class="{ 'disabled': !hasData }">
        <div class="card-icon">
          <span class="icon">📑</span>
        </div>
        <div class="card-content">
          <h4 class="card-title">PDF Report</h4>
          <p class="card-description">Printable document with table formatting</p>
          <div class="card-features">
            <span class="feature">Printable</span>
            <span class="feature">Formatted</span>
            <span class="feature">Shareable</span>
          </div>
        </div>
        <button class="export-btn" :disabled="!hasData">
          Export PDF
        </button>
      </div>
    </div>

    <!-- Export Options -->
    <div class="export-options" v-if="hasData">
      <div class="options-section">
        <h4 class="options-title">
          <span class="options-icon">⚙️</span>
          Export Options
        </h4>

        <div class="options-grid">
          <div class="option-item">
            <label class="option-label">
              <input type="checkbox" v-model="options.includeHeaders" class="option-checkbox" />
              <span class="option-text">Include Column Headers</span>
            </label>
          </div>

          <div class="option-item">
            <label class="option-label">
              <input type="checkbox" v-model="options.formatNumbers" class="option-checkbox" />
              <span class="option-text">Format Numbers</span>
            </label>
          </div>

          <div class="option-item">
            <label class="option-label">
              <input type="checkbox" v-model="options.includeMetadata" class="option-checkbox" />
              <span class="option-text">Include Metadata</span>
            </label>
          </div>

          <div class="option-item">
            <label class="option-label">
              <input type="checkbox" v-model="options.compressed" class="option-checkbox" />
              <span class="option-text">Compressed Export</span>
            </label>
          </div>
        </div>
      </div>

      <div class="filename-section">
        <label class="filename-label">File Name</label>
        <div class="filename-input">
          <input
            type="text"
            v-model="filename"
            placeholder="Enter file name"
            class="filename-field"
          />
          <span class="filename-extension">.{{ currentExtension }}</span>
        </div>
        <small class="filename-hint">File will be saved as: {{ filename }}.{{ currentExtension }}</small>
      </div>
    </div>

    <!-- No Data State -->
    <div v-else class="no-data-state">
      <div class="no-data-icon">📊</div>
      <h4 class="no-data-title">No Data Available</h4>
      <p class="no-data-description">Upload or filter data to enable export options</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import * as XLSX from "xlsx"

const props = defineProps({
  columns: {
    type: Array,
    default: () => []
  },
  filteredRows: {
    type: Array,
    default: () => []
  },
  datasetName: {
    type: String,
    default: 'dataset'
  }
})

// State
const exportType = ref('csv')
const filename = ref('')
const options = ref({
  includeHeaders: true,
  formatNumbers: true,
  includeMetadata: false,
  compressed: false
})

// Computed properties
const hasData = computed(() => {
  return props.filteredRows && props.filteredRows.length > 0 && props.columns && props.columns.length > 0
})

const currentExtension = computed(() => {
  return exportType.value === 'excel' ? 'xlsx' : exportType.value
})

// Initialize filename
watch(() => props.datasetName, (newName) => {
  filename.value = newName || 'dataset'
}, { immediate: true })

// Export methods
const exportData = (type) => {
  if (!hasData.value) {
    showNoDataAlert()
    return
  }

  exportType.value = type

  // Update filename extension
  const baseName = filename.value.replace(/\.[^/.]+$/, "") // Remove existing extension
  filename.value = baseName

  switch(type) {
    case "csv": exportCSV(); break;
    case "excel": exportExcel(); break;
    case "json": exportJSON(); break;
    case "pdf": exportPDF(); break;
  }
}

const showNoDataAlert = () => {
  alert("⚠️ Tidak ada data untuk diexport\n\nSilakan upload dataset atau sesuaikan filter untuk mendapatkan data.")
}

const prepareDataForExport = () => {
  let dataToExport = [...props.filteredRows]

  // Format numbers if option is enabled
  if (options.value.formatNumbers) {
    dataToExport = dataToExport.map(row => {
      const newRow = { ...row }
      props.columns.forEach(col => {
        const value = newRow[col]
        if (typeof value === 'number' || !isNaN(parseFloat(value))) {
          const num = parseFloat(value)
          if (Number.isInteger(num)) {
            newRow[col] = num.toLocaleString()
          } else {
            newRow[col] = num.toLocaleString(undefined, {
              minimumFractionDigits: 2,
              maximumFractionDigits: 2
            })
          }
        }
      })
      return newRow
    })
  }

  return dataToExport
}

const exportCSV = () => {
  const dataToExport = prepareDataForExport()
  const csvRows = []

  // Add headers if enabled
  if (options.value.includeHeaders) {
    csvRows.push(props.columns.join(","))
  }

  // Add data rows
  dataToExport.forEach(row => {
    const values = props.columns.map(col => {
      let value = row[col]
      // Escape quotes and wrap in quotes if contains comma or quotes
      if (typeof value === 'string') {
        value = value.replace(/"/g, '""')
        if (value.includes(',') || value.includes('"') || value.includes('\n')) {
          value = `"${value}"`
        }
      }
      return value
    })
    csvRows.push(values.join(","))
  })

  // Add metadata if enabled
  if (options.value.includeMetadata) {
    csvRows.unshift(`# Generated: ${new Date().toISOString()}`)
    csvRows.unshift(`# Rows: ${dataToExport.length}, Columns: ${props.columns.length}`)
  }

  const csvContent = csvRows.join("\n")
  const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" })
  downloadFile(blob, `${filename.value}.csv`, 'text/csv')

  showExportSuccess(`${filename.value}.csv`)
}

const exportExcel = () => {
  const dataToExport = prepareDataForExport()

  // Create workbook
  const wb = XLSX.utils.book_new()

  // Prepare data for worksheet
  const wsData = []

  // Add headers if enabled
  if (options.value.includeHeaders) {
    wsData.push(props.columns)
  }

  // Add data rows
  dataToExport.forEach(row => {
    const rowData = props.columns.map(col => row[col])
    wsData.push(rowData)
  })

  // Create worksheet
  const ws = XLSX.utils.aoa_to_sheet(wsData)

  // Add metadata sheet if enabled
  if (options.value.includeMetadata) {
    const metadata = [
      ['Dataset Information'],
      ['Generated Date', new Date().toISOString()],
      ['Total Rows', dataToExport.length],
      ['Total Columns', props.columns.length],
      ['Export Options', JSON.stringify(options.value)]
    ]
    const wsMeta = XLSX.utils.aoa_to_sheet(metadata)
    XLSX.utils.book_append_sheet(wb, wsMeta, "Metadata")
  }

  // Add main data sheet
  XLSX.utils.book_append_sheet(wb, ws, "Data")

  // Generate and download
  XLSX.writeFile(wb, `${filename.value}.xlsx`)

  showExportSuccess(`${filename.value}.xlsx`)
}

const exportJSON = () => {
  const dataToExport = prepareDataForExport()

  // Prepare JSON object
  const jsonData = {
    metadata: options.value.includeMetadata ? {
      generated: new Date().toISOString(),
      rows: dataToExport.length,
      columns: props.columns.length,
      options: options.value
    } : undefined,
    columns: options.value.includeHeaders ? props.columns : undefined,
    data: dataToExport
  }

  // Stringify with pretty print if not compressed
  const jsonString = options.value.compressed
    ? JSON.stringify(jsonData)
    : JSON.stringify(jsonData, null, 2)

  const blob = new Blob([jsonString], { type: "application/json" })
  downloadFile(blob, `${filename.value}.json`, 'application/json')

  showExportSuccess(`${filename.value}.json`)
}

const exportPDF = () => {
  const dataToExport = prepareDataForExport()

  // Dynamically import jsPDF and autoTable
  import('jspdf').then(jsPDFModule => {
    // Check which version of jsPDF is being used
    const jsPDF = jsPDFModule.default || jsPDFModule

    // Import autoTable separately
    import('jspdf-autotable').then(autoTableModule => {
      // Initialize jsPDF
      const doc = new jsPDF({
        orientation: 'landscape',
        unit: 'mm',
        format: 'a4'
      })

      // Add title
      doc.setFontSize(16)
      doc.text(`Dataset Export - ${filename.value}`, 14, 15)

      // Add metadata if enabled
      if (options.value.includeMetadata) {
        doc.setFontSize(10)
        doc.text(`Generated: ${new Date().toLocaleString()}`, 14, 25)
        doc.text(`Rows: ${dataToExport.length}, Columns: ${props.columns.length}`, 14, 30)
      }

      // Prepare table data
      const tableColumn = options.value.includeHeaders ? props.columns : props.columns.map((_, i) => `Column ${i + 1}`)
      const tableRows = dataToExport.map(row =>
        props.columns.map(col => {
          const value = row[col]
          return value !== null && value !== undefined ? String(value) : ''
        })
      )

      // Start Y position based on metadata
      const startY = options.value.includeMetadata ? 35 : 20

      // Use the autoTable plugin correctly
      // Note: autoTable might be attached differently depending on version
      if (doc.autoTable) {
        // If autoTable is already attached to jsPDF
        doc.autoTable({
          head: [tableColumn],
          body: tableRows,
          startY: startY,
          margin: { left: 14, right: 14 },
          styles: {
            fontSize: 8,
            cellPadding: 2,
            overflow: 'linebreak',
            cellWidth: 'wrap'
          },
          headStyles: {
            fillColor: [41, 128, 185],
            textColor: 255,
            fontStyle: 'bold'
          },
          alternateRowStyles: {
            fillColor: [245, 245, 245]
          },
          didDrawPage: (data) => {
            // Add page number
            const pageCount = doc.internal.getNumberOfPages()
            doc.setFontSize(8)
            doc.text(
              `Page ${data.pageNumber} of ${pageCount}`,
              doc.internal.pageSize.width - 20,
              doc.internal.pageSize.height - 10
            )
          }
        })
      } else if (autoTableModule.default) {
        // If we need to use the imported autoTable function
        autoTableModule.default(doc, {
          head: [tableColumn],
          body: tableRows,
          startY: startY,
          margin: { left: 14, right: 14 },
          styles: {
            fontSize: 8,
            cellPadding: 2,
            overflow: 'linebreak',
            cellWidth: 'wrap'
          },
          headStyles: {
            fillColor: [41, 128, 185],
            textColor: 255,
            fontStyle: 'bold'
          },
          alternateRowStyles: {
            fillColor: [245, 245, 245]
          },
          didDrawPage: (data) => {
            // Add page number
            const pageCount = doc.internal.getNumberOfPages()
            doc.setFontSize(8)
            doc.text(
              `Page ${data.pageNumber} of ${pageCount}`,
              doc.internal.pageSize.width - 20,
              doc.internal.pageSize.height - 10
            )
          }
        })
      } else {
        // Try using autoTable as a function
        autoTableModule.autoTable(doc, {
          head: [tableColumn],
          body: tableRows,
          startY: startY,
          margin: { left: 14, right: 14 },
          styles: {
            fontSize: 8,
            cellPadding: 2,
            overflow: 'linebreak',
            cellWidth: 'wrap'
          },
          headStyles: {
            fillColor: [41, 128, 185],
            textColor: 255,
            fontStyle: 'bold'
          },
          alternateRowStyles: {
            fillColor: [245, 245, 245]
          },
          didDrawPage: (data) => {
            // Add page number
            const pageCount = doc.internal.getNumberOfPages()
            doc.setFontSize(8)
            doc.text(
              `Page ${data.pageNumber} of ${pageCount}`,
              doc.internal.pageSize.width - 20,
              doc.internal.pageSize.height - 10
            )
          }
        })
      }

      // Download PDF
      doc.save(`${filename.value}.pdf`)

      showExportSuccess(`${filename.value}.pdf`)
    }).catch(error => {
      console.error('Failed to load autoTable plugin:', error)
      showPDFFallback(dataToExport)
    })
  }).catch(error => {
    console.error('Failed to load jsPDF:', error)
    showPDFFallback(dataToExport)
  })
}

// Fallback method if jsPDF fails
const showPDFFallback = (dataToExport) => {
  alert(`PDF export requires additional libraries. For now, please use CSV or Excel export.\n\nAs an alternative, ${dataToExport.length} rows are ready for export in other formats.`)
}

const downloadFile = (blob, fileName, mimeType) => {
  const url = URL.createObjectURL(blob)
  const link = document.createElement("a")
  link.href = url
  link.setAttribute("download", fileName)
  link.style.display = 'none'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)

  // Clean up
  setTimeout(() => {
    URL.revokeObjectURL(url)
  }, 100)
}

const showExportSuccess = (fileName) => {
  // Create success notification
  const notification = document.createElement('div')
  notification.className = 'export-success-notification'
  notification.innerHTML = `
    <div class="success-content">
      <span class="success-icon">✅</span>
      <div class="success-text">
        <strong>Export Successful!</strong>
        <p>File "${fileName}" has been downloaded.</p>
      </div>
    </div>
  `

  // Add styles
  notification.style.cssText = `
    position: fixed;
    top: 20px;
    right: 20px;
    background: #10b981;
    color: white;
    padding: 16px 20px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    z-index: 10000;
    animation: slideIn 0.3s ease;
  `

  document.body.appendChild(notification)

  // Remove after 3 seconds
  setTimeout(() => {
    notification.style.animation = 'slideOut 0.3s ease'
    setTimeout(() => {
      document.body.removeChild(notification)
    }, 300)
  }, 3000)
}

// Add CSS animations to document head
const addStyles = () => {
  if (document.querySelector('#export-notification-styles')) return

  const style = document.createElement('style')
  style.id = 'export-notification-styles'
  style.textContent = `
    @keyframes slideIn {
      from {
        transform: translateX(100%);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
    }

    @keyframes slideOut {
      from {
        transform: translateX(0);
        opacity: 1;
      }
      to {
        transform: translateX(100%);
        opacity: 0;
      }
    }

    .success-content {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .success-icon {
      font-size: 24px;
    }

    .success-text strong {
      display: block;
      margin-bottom: 4px;
    }

    .success-text p {
      margin: 0;
      font-size: 14px;
      opacity: 0.9;
    }
  `
  document.head.appendChild(style)
}

// Add styles on component mount
addStyles()
</script>

<style scoped>
.export-menu {
  background: white;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
}

.export-header {
  margin-bottom: 24px;
  text-align: center;
}

.export-title {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
  margin: 0 0 8px 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.export-icon {
  color: #667eea;
}

.export-subtitle {
  font-size: 15px;
  color: #6b7280;
  margin: 0;
}

.export-stats {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 32px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 20px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  min-width: 140px;
}

.stat-icon {
  font-size: 20px;
  width: 40px;
  height: 40px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stat-content {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.stat-value {
  font-size: 20px;
  font-weight: 700;
  color: #1f2937;
  line-height: 1;
}

.stat-label {
  font-size: 13px;
  color: #6b7280;
  font-weight: 500;
}

.export-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 32px;
}

.export-card {
  background: white;
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  display: flex;
  flex-direction: column;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.export-card:hover:not(.disabled) {
  border-color: #667eea;
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.15);
}

.export-card.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.export-card.disabled:hover {
  border-color: #e5e7eb;
  transform: none;
  box-shadow: none;
}

.card-icon {
  margin-bottom: 16px;
  text-align: center;
}

.card-icon .icon {
  font-size: 48px;
  display: inline-block;
  padding: 16px;
  background: linear-gradient(135deg, #667eea15 0%, #764ba215 100%);
  border-radius: 12px;
  color: #667eea;
}

.card-content {
  flex: 1;
  margin-bottom: 20px;
}

.card-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
  text-align: center;
}

.card-description {
  font-size: 14px;
  color: #6b7280;
  line-height: 1.5;
  margin: 0 0 16px 0;
  text-align: center;
}

.card-features {
  display: flex;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
}

.feature {
  padding: 4px 10px;
  background: #f3f4f6;
  color: #4b5563;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.export-btn {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.export-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #5a6fd9 0%, #6a4199 100%);
  transform: translateY(-1px);
}

.export-btn:disabled {
  background: #9ca3af;
  cursor: not-allowed;
  transform: none;
}

.export-options {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  margin-top: 32px;
}

.options-section {
  margin-bottom: 24px;
}

.options-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 16px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.options-icon {
  color: #667eea;
}

.options-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
}

.option-item {
  display: flex;
  align-items: center;
}

.option-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  user-select: none;
}

.option-checkbox {
  width: 18px;
  height: 18px;
  border: 2px solid #d1d5db;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.option-checkbox:checked {
  background-color: #667eea;
  border-color: #667eea;
}

.option-text {
  font-size: 14px;
  color: #374151;
  font-weight: 500;
}

.filename-section {
  border-top: 1px solid #e5e7eb;
  padding-top: 24px;
}

.filename-label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
}

.filename-input {
  display: flex;
  align-items: center;
  gap: 0;
  max-width: 400px;
}

.filename-field {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  border-right: none;
  border-radius: 6px 0 0 6px;
  font-size: 14px;
  color: #374151;
  background: white;
}

.filename-field:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.filename-extension {
  padding: 10px 14px;
  background: #f3f4f6;
  border: 1px solid #d1d5db;
  border-left: none;
  border-radius: 0 6px 6px 0;
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
  min-width: 60px;
  text-align: center;
}

.filename-hint {
  display: block;
  margin-top: 8px;
  font-size: 13px;
  color: #9ca3af;
}

.no-data-state {
  text-align: center;
  padding: 48px 24px;
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border-radius: 12px;
  border: 2px dashed #d1d5db;
}

.no-data-icon {
  font-size: 64px;
  margin-bottom: 20px;
  opacity: 0.3;
  display: block;
}

.no-data-title {
  font-size: 20px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
}

.no-data-description {
  color: #6b7280;
  font-size: 15px;
  margin: 0;
  max-width: 300px;
  margin: 0 auto;
}

/* Responsive Design */
@media (max-width: 768px) {
  .export-grid {
    grid-template-columns: 1fr;
  }

  .export-stats {
    flex-direction: column;
    align-items: center;
  }

  .stat-item {
    width: 100%;
    max-width: 250px;
  }

  .options-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .export-menu {
    padding: 16px;
  }

  .export-title {
    font-size: 20px;
  }

  .export-card {
    padding: 20px;
  }

  .card-icon .icon {
    font-size: 40px;
    padding: 12px;
  }

  .export-options {
    padding: 20px;
  }
}
</style>