<template>
  <div class="dashboard-container">
    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-header">
        <div class="logo-container">
          <div class="logo-icon">📊</div>
          <h1 class="logo-text">DataDash</h1>
          <span class="logo-subtitle">Analytics</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <ul>
          <li :class="{ active: activeMenu === 'upload' }" @click="activeMenu = 'upload'" class="nav-item">
            <span class="nav-icon">📤</span>
            <span class="nav-text">Upload Data</span>
          </li>
          <li :class="{ active: activeMenu === 'table' }" @click="activeMenu = 'table'" class="nav-item">
            <span class="nav-icon">📋</span>
            <span class="nav-text">Data Table</span>
          </li>
          <li :class="{ active: activeMenu === 'chart' }" @click="activeMenu = 'chart'" class="nav-item">
            <span class="nav-icon">📈</span>
            <span class="nav-text">Visualisasi</span>
          </li>
          <li :class="{ active: activeMenu === 'eda' }" @click="activeMenu = 'eda'" class="nav-item">
            <span class="nav-icon">🔍</span>
            <span class="nav-text">Auto Profiling</span>
            <span class="nav-badge">EDA</span>
          </li>
          <li :class="{ active: activeMenu === 'insight' }" @click="activeMenu = 'insight'" class="nav-item">
            <span class="nav-icon">🧠</span>
            <span class="nav-text">AI Insight</span>
            <span class="nav-badge">NEW</span>
          </li>
        </ul>
      </nav>

      <div class="sidebar-footer">
        <div class="user-info">
          <div class="user-avatar">👤</div>
          <div class="user-details">
            <p class="user-name">Admin User</p>
            <p class="user-role">Data Analyst</p>
          </div>
        </div>
      </div>
    </aside>

    <!-- Main Content -->
    <main class="main-content">
      <div class="content-header">
        <h2 class="page-title">{{ getPageTitle }}</h2>
        <div class="header-actions">
          <button class="btn btn-secondary" @click="exportData">
            <span class="btn-icon">📥</span> Export
          </button>
          <button class="btn btn-primary" @click="refreshData">
            <span class="btn-icon">🔄</span> Refresh
          </button>
        </div>
      </div>

      <!-- Upload Section -->
      <section v-if="activeMenu === 'upload'" class="card upload-section">
        <div class="card-header">
          <h3 class="card-title">Upload Dataset</h3>
          <p class="card-subtitle">Upload file CSV, Excel, atau JSON untuk analisis</p>
        </div>
        <div class="card-body">
          <div class="upload-area" @dragover.prevent @drop="handleDrop">
            <div class="upload-icon">📁</div>
            <h4 class="upload-title">Drop files here or click to upload</h4>
            <p class="upload-description">Supports: .csv, .xlsx, .json (Max 10MB)</p>
            <label class="upload-btn">
              <input type="file" @change="uploadFile" hidden accept=".csv,.xlsx,.xls,.json" />
              Browse Files
            </label>
          </div>

          <!-- Upload Status -->
          <div v-if="isUploaded" class="status-success">
            <div class="status-icon">✅</div>
            <div class="status-content">
              <h4>Upload Successful!</h4>
              <p>Data has been successfully uploaded and is ready for analysis.</p>
            </div>
          </div>
        </div>
      </section>

      <!-- Data Table Section -->
      <section v-if="activeMenu === 'table'" class="card">
        <div class="card-header">
          <h3 class="card-title">Dataset Preview</h3>
          <div class="table-stats" v-if="columns.length">
            <span class="stat-item">
              <span class="stat-icon">📊</span>
              {{ rows.length }} rows
            </span>
            <span class="stat-item">
              <span class="stat-icon">📋</span>
              {{ columns.length }} columns
            </span>
          </div>
        </div>

        <div class="card-body">
          <!-- Filter Section -->
          <div v-if="columns.length" class="table-filters">
            <div v-for="col in columns" :key="col" class="filter-control">
              <label>{{ col }}</label>
              <select v-if="categoricalColumns.includes(col)" v-model="filters[col]">
                <option value="">All</option>
                <option v-for="val in uniqueValues(col)" :key="val" :value="val">{{ val }}</option>
              </select>
              <input v-else type="number" placeholder="Filter value" v-model.number="filters[col]" />
            </div>
            <button @click="applyFilters" class="apply-btn">Apply Filters</button>
            <button @click="clearFilters" class="clear-btn">Clear Filters</button>
          </div>

          <div v-if="columns.length" class="table-section">
            <!-- Scroll Container -->
            <div class="table-scroll-container">
              <div class="table-wrapper">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th v-for="(col, index) in columns" :key="col" class="table-header">
                        <div class="header-content">

                          <span class="column-name">{{ col }}</span>
                        </div>
                      </th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(row, i) in filteredRows" :key="i" :class="{ 'alternate-row': i % 2 === 0 }">
                      <td v-for="col in columns" :key="col" class="table-cell" :title="row[col]">
                        {{ row[col] }}
                      </td>
                    </tr>

                  </tbody>
                </table>
              </div>


            </div>

          </div>

          <div v-else class="empty-state">
            <div class="empty-icon">📊</div>
            <h4 class="empty-title">Belum ada data yang diupload</h4>
            <p class="empty-description">Upload dataset untuk melihat preview tabel</p>
          </div>
        </div>





      </section>

      <!-- Visualization Section -->
      <section v-if="activeMenu === 'chart'" class="card">
        <div class="card-header">
          <h3 class="card-title">Data Visualization</h3>
          <p class="card-subtitle">Create charts from your dataset</p>
        </div>
        <div class="card-body">
          <div v-if="columns.length">
            <div class="chart-controls">
              <div class="control-group">
                <div class="control-item">
                  <label class="control-label">X-Axis Column</label>
                  <select v-model="xColumn" class="control-select">
                    <option value="">Select X-axis</option>
                    <option v-for="col in columns" :key="col" :value="col">{{ col }}</option>
                  </select>
                </div>
                <div class="control-item">
                  <label class="control-label">Y-Axis Columns</label>
                  <select v-model="yColumns" multiple class="control-select multiple">
                    <option v-for="col in numericColumns" :key="col" :value="col">{{ col }}</option>
                  </select>
                  <small class="control-hint">Hold Ctrl to select multiple</small>
                </div>
                <div class="control-item">
                  <label class="control-label">Chart Type</label>
                  <div class="chart-type-selector">
                    <button v-for="type in chartTypes" :key="type.value" :class="{ active: chartType === type.value }"
                      @click="chartType = type.value" class="chart-type-btn">
                      <span class="chart-type-icon">{{ type.icon }}</span>
                      {{ type.label }}
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div class="chart-container">
              <canvas ref="chartCanvas" class="chart-canvas"></canvas>
            </div>
          </div>
          <div v-else class="empty-state">
            <div class="empty-icon">📈</div>
            <h4>No Data for Visualization</h4>
            <p>Upload data and select numeric columns for Y-axis</p>
          </div>
        </div>
      </section>

      <!-- EDA Section -->
      <section v-if="activeMenu === 'eda'" class="card">
        <div class="card-header">
          <h3 class="card-title">Exploratory Data Analysis</h3>
          <p class="card-subtitle">Automated data profiling and insights</p>
        </div>
        <div class="card-body">
          <div v-if="columns.length">
            <!-- Stats Overview -->
            <div class="eda-overview">
              <div class="stat-row">
                <div class="stat-item">
                  <div class="stat-icon">📊</div>
                  <div class="stat-content">
                    <div class="stat-value">{{ rows.length }}</div>
                    <div class="stat-label">Total Baris</div>
                  </div>
                </div>
                <div class="stat-item">
                  <div class="stat-icon">📋</div>
                  <div class="stat-content">
                    <div class="stat-value">{{ columns.length }}</div>
                    <div class="stat-label">Total Kolom</div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Column Types -->
            <div class="column-types">
              <div class="type-section">
                <h4 class="type-title">
                  <span class="type-icon">🔢</span>
                  Kolom Numerik
                </h4>
                <div class="type-content">
                  <div v-if="numericColumns.length" class="tags-container">
                    <span v-for="col in numericColumns" :key="col" class="tag tag-numeric">
                      {{ col }}
                    </span>
                  </div>
                  <div v-else class="empty-message">
                    <span class="empty-icon">—</span>
                    Tidak ada kolom numerik
                  </div>
                </div>
              </div>

              <div class="type-section">
                <h4 class="type-title">
                  <span class="type-icon">🏷️</span>
                  Kolom Kategori
                </h4>
                <div class="type-content">
                  <div v-if="categoricalColumns.length" class="tags-container">
                    <span v-for="col in categoricalColumns" :key="col" class="tag tag-categorical">
                      {{ col }}
                    </span>
                  </div>
                  <div v-else class="empty-message">
                    <span class="empty-icon">—</span>
                    Tidak ada kolom kategori
                  </div>
                </div>
              </div>
            </div>

            <!-- Column Summary -->
            <div class="column-summary">
              <h4 class="summary-title">Summary Columns</h4>
              <div class="summary-grid">
                <div v-for="col in columns" :key="col" class="summary-item">
                  <div class="summary-header">
                    <span class="column-name">{{ col }}</span>
                    <span :class="[
                      'column-badge',
                      numericColumns.includes(col) ? 'badge-numeric' : 'badge-categorical'
                    ]">
                      {{ numericColumns.includes(col) ? 'Numeric' : 'Categorical' }}
                    </span>
                  </div>
                  <div class="summary-stats">
                    <div class="stat-mini">
                      <span class="stat-mini-label">Type</span>
                      <span class="stat-mini-value">
                        {{ numericColumns.includes(col) ? 'Number' : 'String' }}
                      </span>
                    </div>
                    <div class="stat-mini">
                      <span class="stat-mini-label">Unique</span>
                      <span class="stat-mini-value">—</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Empty State -->
          <div v-else class="empty-state">
            <div class="empty-icon">📊</div>
            <h4 class="empty-title">Belum ada data untuk dianalisis</h4>
            <p class="empty-description">Upload dataset untuk memulai analisis EDA</p>
            <button class="empty-action" @click="activeMenu = 'upload'">
              <span class="action-icon">📤</span>
              Upload Data
            </button>
          </div>
        </div>
      </section>



      <!-- AI Insight Section -->
      <section v-if="activeMenu === 'insight'" class="card">
        <div class="card-header">
          <h3 class="card-title">AI Insights</h3>
          <p class="card-subtitle">Powered by AI for deeper data understanding</p>
        </div>
        <div class="card-body">
          <div v-if="insights.length" class="insights-container">
            <div v-for="(insight, idx) in insights" :key="idx" class="insight-card">
              <div class="insight-icon">💡</div>
              <div class="insight-content">
                <p>{{ insight }}</p>
              </div>
            </div>
          </div>
          <div v-else class="empty-state">
            <div class="empty-icon">🧠</div>
            <h4>No Insights Generated</h4>
            <p>Upload data and create visualizations to generate AI insights</p>
            <button class="btn btn-primary" @click="generateInsights">
              <span class="btn-icon">✨</span> Generate Insights
            </button>
          </div>
        </div>
      </section>

      <!-- Footer -->
      <footer class="footer">
        <p>DataDash Analytics v1.0 • © 2024 All rights reserved</p>
        <div class="footer-links">
          <a href="#" class="footer-link">Documentation</a>
          <a href="#" class="footer-link">Support</a>
          <a href="#" class="footer-link">Settings</a>
        </div>
      </footer>
    </main>
  </div>
</template>

<script setup>
import { ref, watch, computed, nextTick } from 'vue'
import Chart from 'chart.js/auto'
import Swal from 'sweetalert2'
import { reactive } from 'vue'

const activeMenu = ref('upload')
const columns = ref([])
const rows = ref([])
const chartCanvas = ref(null)
const xColumn = ref('')
const yColumns = ref([])
const chartType = ref('line')
const insights = ref([])
const isUploaded = ref(false)
let chartInstance = null
const filters = reactive({})  // menampung nilai filter per kolom

const uniqueValues = (col) => {
  const vals = rows.value.map(r => r[col])
  return [...new Set(vals)]
}

const numericColumns = computed(() =>
  columns.value.filter(col => rows.value.every(r => !isNaN(parseFloat(r[col]))))
)
const categoricalColumns = computed(() =>
  columns.value.filter(col => rows.value.some(r => isNaN(parseFloat(r[col]))))
)



const isNumericColumn = (col) => numericColumns.value.includes(col)

const uploadFile = async (event) => {
  const file = event.target.files[0]
  const formData = new FormData()
  formData.append('file', file)

  const res = await fetch('http://127.0.0.1:8000/upload', {
    method: 'POST',
    body: formData
  })
  const data = await res.json()
  columns.value = data.columns
  rows.value = data.data

  isUploaded.value = true

  // Inisialisasi filters
columns.value.forEach(col => {
  filters[col] = ''
})



  xColumn.value = columns.value[0]
  yColumns.value = numericColumns.value.slice(0, 2)
}

// Validasi Y menggunakan SweetAlert
watch(yColumns, (newCols) => {
  for (const col of newCols) {
    if (!isNumericColumn(col)) {
      Swal.fire({
        icon: 'warning',
        title: 'Kolom Tidak Valid',
        text: `Kolom "${col}" bukan numerik. Insight dan grafik hanya mendukung data angka.`,
        confirmButtonColor: '#4f46e5'
      })
      yColumns.value = []
      insights.value = []
      return
    }
  }
})

// Chart Dinamis
watch([rows, xColumn, yColumns, chartType], async () => {
  if (!rows.value.length || !xColumn.value || !yColumns.value.length) return

  await nextTick()
  if (!chartCanvas.value) return
  if (chartInstance) chartInstance.destroy()

  // Grouping
  const grouped = {}
  rows.value.forEach(row => {
    const x = row[xColumn.value]
    if (!grouped[x]) grouped[x] = {}
    yColumns.value.forEach(y => {
      if (!grouped[x][y]) grouped[x][y] = []
      grouped[x][y].push(Number(row[y]) || 0)
    })
  })

  const labels = Object.keys(grouped)
  const datasets = yColumns.value.map(y => ({
    label: y,
    data: labels.map(x => grouped[x][y].reduce((a, b) => a + b, 0) / grouped[x][y].length),
    borderWidth: 2
  }))

  chartInstance = new Chart(chartCanvas.value, {
    type: chartType.value,
    data: { labels, datasets },
    options: {
      responsive: true,
      interaction: { mode: 'index', intersect: false },
      scales: { y: { beginAtZero: true } }
    }
  })

  // Fetch AI Insight
  const res = await fetch("http://127.0.0.1:8000/insight", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      x: rows.value.map(r => r[xColumn.value]),
      y: rows.value.map(r => Number(r[yColumns.value[0]])),
      x_name: xColumn.value,
      y_name: yColumns.value[0]
    })
  })
  const data = await res.json()
  insights.value = data.insights
})



const applyFilters = () => {
  // cukup biarkan filters reactive, filteredRows computed akan otomatis update
  console.log('Filter diterapkan:', filters)
}

const clearFilters = () => {
  Object.keys(filters).forEach(key => filters[key] = '')
}


const filteredRows = computed(() => {
  if (!filters) return rows.value
  return rows.value.filter(row => {
    return Object.keys(filters).every(col => {
      const filterVal = filters[col]
      if (filterVal === '' || filterVal == null) return true

      // cek numeric atau bukan
      if (!isNaN(filterVal) && !isNaN(row[col])) {
        return Number(row[col]) === Number(filterVal)
      }
      return String(row[col]) === String(filterVal)
    })
  })
})



// Fungsi export ke CSV
const exportData = () => {
  if (!filteredRows.value.length) {
    Swal.fire({
      icon: 'info',
      title: 'Data Kosong',
      text: 'Tidak ada data untuk diexport',
      confirmButtonColor: '#4f46e5'
    })
    return
  }

  const csvRows = []
  // Header
  csvRows.push(columns.value.join(','))
  // Data
  filteredRows.value.forEach(row => {
    const values = columns.value.map(col => {
      let val = row[col]
      if (typeof val === 'string') {
        // Escape double quotes
        val = `"${val.replace(/"/g, '""')}"`
      }
      return val
    })
    csvRows.push(values.join(','))
  })

  const csvContent = csvRows.join('\n')
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)

  const link = document.createElement('a')
  link.setAttribute('href', url)
  link.setAttribute('download', 'dataset_export.csv')
  link.style.visibility = 'hidden'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

</script>

<style scoped>
/* Reset and Base Styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.dashboard-container {
  display: flex;
  min-height: 100vh;
  background: #f8fafc;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* Sidebar Styles */
.sidebar {
  width: 260px;
  background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
  color: white;
  display: flex;
  flex-direction: column;
  box-shadow: 4px 0 20px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.sidebar-header {
  padding: 24px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.logo-container {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.logo-icon {
  font-size: 32px;
  margin-bottom: 8px;
}

.logo-text {
  font-size: 24px;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa 0%, #3b82f6 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.logo-subtitle {
  font-size: 12px;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.sidebar-nav {
  padding: 16px;
  flex: 1;
}

.nav-item {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  margin: 4px 0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #cbd5e1;
  position: relative;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.05);
  color: white;
}

.nav-item.active {
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.nav-icon {
  font-size: 20px;
  margin-right: 12px;
  width: 24px;
  text-align: center;
}

.nav-text {
  flex: 1;
  font-size: 14px;
  font-weight: 500;
}

.nav-badge {
  background: rgba(255, 255, 255, 0.1);
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #60a5fa 0%, #3b82f6 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
}

.user-details {
  flex: 1;
}

.user-name {
  font-size: 14px;
  font-weight: 600;
  color: white;
}

.user-role {
  font-size: 12px;
  color: #94a3b8;
}

/* Main Content Styles */
.main-content {
  flex: 1;
  padding: 24px;
  overflow-y: auto;
}

.content-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.page-title {
  font-size: 28px;
  font-weight: 700;
  color: #1e293b;
  background: linear-gradient(135deg, #1e293b 0%, #475569 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-actions {
  display: flex;
  gap: 12px;
}

/* Button Styles */
.btn {
  padding: 10px 20px;
  border-radius: 8px;
  border: none;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
}

.btn-primary {
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4);
}

.btn-secondary {
  background: white;
  color: #475569;
  border: 1px solid #e2e8f0;
}

.btn-secondary:hover {
  background: #f8fafc;
  border-color: #cbd5e1;
}

.btn-icon {
  font-size: 16px;
}

/* Card Styles */
.card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
  margin-bottom: 24px;
  overflow: hidden;
  border: 1px solid #e2e8f0;
}

.card-header {
  padding: 24px;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 20px;
  font-weight: 700;
  color: #1e293b;
  margin-bottom: 4px;
}

.card-subtitle {
  font-size: 14px;
  color: #64748b;
}

.card-body {
  padding: 24px;
}

/* Upload Section */
.upload-area {
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 48px 24px;
  text-align: center;
  transition: all 0.3s ease;
  background: #f8fafc;
  cursor: pointer;
}

.upload-area:hover {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.02);
}

.upload-icon {
  font-size: 48px;
  margin-bottom: 16px;
  color: #94a3b8;
}

.upload-title {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin-bottom: 8px;
}

.upload-description {
  color: #64748b;
  margin-bottom: 24px;
  font-size: 14px;
}

.upload-btn {
  display: inline-block;
  padding: 12px 32px;
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: none;
}

.upload-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.status-success {
  display: flex;
  align-items: center;
  gap: 16px;
  background: #10b98110;
  border: 1px solid #10b98130;
  border-radius: 12px;
  padding: 20px;
  margin-top: 24px;
}

.status-icon {
  font-size: 32px;
}

.status-content h4 {
  color: #047857;
  margin-bottom: 4px;
}

.status-content p {
  color: #065f46;
  font-size: 14px;
}

/* Table Styles */
.table-container {
  overflow-x: auto;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.table-header {
  background: #f8fafc;
  padding: 16px;
  text-align: left;
  font-weight: 600;
  color: #475569;
  border-bottom: 2px solid #e2e8f0;
  white-space: nowrap;
}

.header-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.header-type {
  font-size: 11px;
  color: #94a3b8;
  font-weight: 500;
  text-transform: uppercase;
}

.table-cell {
  padding: 16px;
  border-bottom: 1px solid #e2e8f0;
  color: #475569;
}

.even-row {
  background: #f8fafc;
}

/* Pagination */
.pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  padding: 24px;
}

.pagination-btn {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
  background: white;
  color: #475569;
  cursor: pointer;
  transition: all 0.3s ease;
}

.pagination-btn:hover:not(:disabled) {
  background: #f8fafc;
  border-color: #cbd5e1;
}

.pagination-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pagination-info {
  font-size: 14px;
  color: #64748b;
}

/* Chart Styles */
.chart-controls {
  background: #f8fafc;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 24px;
}

.control-group {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 24px;
}

.control-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.control-label {
  font-size: 14px;
  font-weight: 600;
  color: #475569;
}

.control-select {
  padding: 12px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  color: #1e293b;
  background: white;
  transition: all 0.3s ease;
}

.control-select:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.control-select.multiple {
  min-height: 120px;
}

.control-hint {
  font-size: 12px;
  color: #94a3b8;
}

.chart-type-selector {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.chart-type-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 16px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  background: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.chart-type-btn:hover {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.02);
}

.chart-type-btn.active {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.1);
  color: #3b82f6;
}

.chart-type-icon {
  font-size: 24px;
}

.chart-container {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 24px;
  height: 400px;
}

.chart-canvas {
  width: 100% !important;
  height: 100% !important;
}

/* Stats Grid */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 32px;
}

.stat-card {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border-radius: 12px;
  padding: 24px;
  display: flex;
  align-items: center;
  gap: 16px;
  border: 1px solid #e2e8f0;
}

.stat-icon {
  font-size: 32px;
  background: white;
  width: 64px;
  height: 64px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #1e293b;
  line-height: 1;
}

.stat-label {
  font-size: 14px;
  color: #64748b;
  margin-top: 4px;
}

/* Column Analysis */
.section-title {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin-bottom: 16px;
}

.column-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.column-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: #f8fafc;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.column-name {
  font-weight: 500;
  color: #475569;
}

.column-type {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
}

.type-numeric {
  background: #dbeafe;
  color: #1d4ed8;
}

.type-string {
  background: #f0f9ff;
  color: #0369a1;
}

.type-date {
  background: #fef3c7;
  color: #92400e;
}

.type-unknown {
  background: #f1f5f9;
  color: #64748b;
}

/* AI Insights */
.insights-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.insight-card {
  display: flex;
  gap: 16px;
  padding: 20px;
  background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
  border-radius: 12px;
  border-left: 4px solid #0ea5e9;
}

.insight-icon {
  font-size: 24px;
}

.insight-content {
  flex: 1;
}

.insight-content p {
  color: #0c4a6e;
  line-height: 1.6;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 48px 24px;
}

.empty-icon {
  font-size: 64px;
  margin-bottom: 16px;
  opacity: 0.5;
}

.empty-state h4 {
  font-size: 20px;
  font-weight: 600;
  color: #475569;
  margin-bottom: 8px;
}

.empty-state p {
  color: #94a3b8;
  margin-bottom: 24px;
}

/* Footer */
.footer {
  margin-top: 48px;
  padding-top: 24px;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #64748b;
  font-size: 14px;
}

.footer-links {
  display: flex;
  gap: 24px;
}

.footer-link {
  color: #64748b;
  text-decoration: none;
  transition: color 0.3s ease;
}

.footer-link:hover {
  color: #3b82f6;
}


/* EDA Card Styling */
.eda-overview {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 24px;
  border: 1px solid #e2e8f0;
}

.stat-row {
  display: flex;
  gap: 20px;
}

.stat-item {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-icon {
  font-size: 32px;
  width: 56px;
  height: 56px;
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stat-content {
  flex: 1;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #1e293b;
  line-height: 1;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: #64748b;
  font-weight: 500;
}

/* Column Types */
.column-types {
  display: flex;
  flex-direction: column;
  gap: 24px;
  margin-bottom: 32px;
}

.type-section {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  overflow: hidden;
}

.type-title {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 20px;
  background: #f8fafc;
  font-size: 16px;
  font-weight: 600;
  color: #1e293b;
  border-bottom: 1px solid #e2e8f0;
}

.type-icon {
  font-size: 20px;
}

.type-content {
  padding: 20px;
}

.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tag {
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 13px;
  font-weight: 500;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  transition: all 0.3s ease;
}

.tag-numeric {
  background: linear-gradient(135deg, #dbeafe 0%, #bfdbfe 100%);
  color: #1d4ed8;
  border: 1px solid #93c5fd;
}

.tag-numeric::before {
  content: "🔢";
  font-size: 12px;
}

.tag-categorical {
  background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
  color: #0369a1;
  border: 1px solid #7dd3fc;
}

.tag-categorical::before {
  content: "🏷️";
  font-size: 12px;
}

.tag:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.empty-message {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #94a3b8;
  font-size: 14px;
  padding: 8px 0;
}

.empty-icon {
  font-size: 20px;
  opacity: 0.5;
}

/* Column Summary */
.column-summary {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 24px;
}

.summary-title {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.summary-title::before {
  content: "📋";
  font-size: 20px;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
}

.summary-item {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 16px;
  transition: all 0.3s ease;
}

.summary-item:hover {
  background: white;
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.summary-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.column-name {
  font-weight: 600;
  color: #475569;
  font-size: 15px;
}

.column-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.badge-numeric {
  background: #dbeafe;
  color: #1d4ed8;
}

.badge-categorical {
  background: #f0f9ff;
  color: #0369a1;
}

.summary-stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.stat-mini {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 0;
  border-bottom: 1px dashed #e2e8f0;
}

.stat-mini:last-child {
  border-bottom: none;
}

.stat-mini-label {
  font-size: 12px;
  color: #64748b;
}

.stat-mini-value {
  font-size: 13px;
  font-weight: 600;
  color: #475569;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 60px 40px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border-radius: 12px;
  border: 2px dashed #cbd5e1;
}

.empty-icon {
  font-size: 64px;
  margin-bottom: 20px;
  opacity: 0.3;
  display: block;
}

.empty-title {
  font-size: 20px;
  font-weight: 600;
  color: #475569;
  margin-bottom: 8px;
}

.empty-description {
  color: #94a3b8;
  font-size: 15px;
  margin-bottom: 24px;
  max-width: 400px;
  margin-left: auto;
  margin-right: auto;
}

.empty-action {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.empty-action:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.action-icon {
  font-size: 18px;
}

/* Table Scroll Container */
.table-scroll-container {
  position: relative;
  width: 100%;
  max-height: 600px;
  overflow: auto;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  margin-bottom: 12px;
}

.table-wrapper {
  min-width: 100%;
  width: fit-content;
  /* Ini yang bikin tabel bisa expand */
  display: inline-block;
}

/* Responsive Design */
@media (max-width: 768px) {
  .stat-row {
    flex-direction: column;
    gap: 12px;
  }

  .summary-grid {
    grid-template-columns: 1fr;
  }

  .type-section {
    margin-bottom: 16px;
  }

  .tags-container {
    justify-content: center;
  }

  .empty-state {
    padding: 40px 20px;
  }
}

@media (max-width: 480px) {
  .column-types {
    gap: 16px;
  }

  .type-title {
    padding: 14px 16px;
    font-size: 15px;
  }

  .type-content {
    padding: 16px;
  }

  .summary-item {
    padding: 12px;
  }

  .summary-stats {
    grid-template-columns: 1fr;
    gap: 8px;
  }
}

/* Animation */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.eda-overview,
.column-types,
.column-summary {
  animation: fadeInUp 0.5s ease-out;
}

.eda-overview {
  animation-delay: 0.1s;
}

.column-types {
  animation-delay: 0.2s;
}

.column-summary {
  animation-delay: 0.3s;
}

/* Scrollbar for tags */
.tags-container {
  max-height: 200px;
  overflow-y: auto;
  padding-right: 8px;
}

.tags-container::-webkit-scrollbar {
  width: 6px;
}

.tags-container::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 3px;
}

.tags-container::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

.tags-container::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}


/* Data Table */
.data-table {
  width: 100%;
  min-width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 14px;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

/* Header Styling */
.table-header {
  padding: 16px 12px;
  text-align: left;
  font-weight: 600;
  color: #374151;
  background: #f9fafb;
  border-bottom: 2px solid #e5e7eb;
  border-right: 1px solid #f3f4f6;
  position: sticky;
  left: 0;
  min-width: 150px;
  white-space: nowrap;
}

.table-header:last-child {
  border-right: none;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.column-index {
  background: #3b82f6;
  color: white;
  font-size: 11px;
  font-weight: 600;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.column-name {
  font-weight: 600;
  color: #1f2937;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Body Styling */
.table-cell {
  padding: 14px 12px;
  border-bottom: 1px solid #f3f4f6;
  border-right: 1px solid #f3f4f6;
  color: #4b5563;
  background: white;
  min-height: 48px;
  vertical-align: middle;
  max-width: 250px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.table-cell:last-child {
  border-right: none;
}

/* Row Styling */
.data-table tbody tr:hover {
  background: #f8fafc;
}

.data-table tbody tr:hover .table-cell {
  background: #f8fafc;
}

.alternate-row .table-cell {
  background: #f9fafb;
}

.alternate-row:hover .table-cell {
  background: #f1f5f9;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 60px 20px;
  background: #f9fafb;
  border-radius: 8px;
  margin: 20px;
}

.empty-icon {
  font-size: 56px;
  margin-bottom: 20px;
  opacity: 0.3;
  display: block;
}

.empty-title {
  font-size: 18px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
}

.empty-description {
  color: #6b7280;
  font-size: 14px;
  margin: 0;
  max-width: 300px;
  margin: 0 auto;
}

/* Scrollbar Styling */
.table-wrapper::-webkit-scrollbar {
  height: 8px;
  width: 8px;
}

.table-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

/* Responsive */
@media (max-width: 768px) {
  .card-header {
    padding: 16px 20px;
  }

  .card-title {
    font-size: 18px;
  }

  .table-stats {
    flex-wrap: wrap;
    gap: 10px;
  }

  .stat-item {
    font-size: 13px;
    padding: 5px 10px;
  }

  .table-header {
    padding: 12px 8px;
    min-width: 120px;
  }

  .table-cell {
    padding: 10px 8px;
    font-size: 13px;
  }

  .empty-state {
    padding: 40px 16px;
    margin: 16px;
  }
}


/* Responsive Design */
@media (max-width: 1024px) {
  .dashboard-container {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    height: auto;
  }

  .sidebar-nav ul {
    display: flex;
    overflow-x: auto;
    padding: 8px;
  }

  .nav-item {
    flex: 0 0 auto;
    white-space: nowrap;
  }

  .sidebar-footer {
    display: none;
  }
}

@media (max-width: 768px) {
  .main-content {
    padding: 16px;
  }

  .content-header {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .header-actions {
    justify-content: flex-end;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .control-group {
    grid-template-columns: 1fr;
  }

  .footer {
    flex-direction: column;
    gap: 16px;
    text-align: center;
  }
}

/* Animations */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fadeIn 0.3s ease-out;
}

/* Scrollbar Styling */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.table-filters {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 20px;
  align-items: flex-end;
}

.filter-control {
  display: flex;
  flex-direction: column;
  min-width: 150px;
}

.filter-control label {
  font-size: 13px;
  font-weight: 500;
  color: #555;
  margin-bottom: 5px;
}

.filter-control select,
.filter-control input {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #ddd;
  font-size: 14px;
}

.apply-btn,
.clear-btn {
  padding: 8px 14px;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  font-weight: 500;
  margin-left: 5px;
}

.apply-btn {
  background-color: #4f46e5;
  color: #fff;
}

.apply-btn:hover {
  background-color: #4338ca;
}

.clear-btn {
  background-color: #e5e7eb;
  color: #333;
}

.clear-btn:hover {
  background-color: #d1d5db;
}

</style>
