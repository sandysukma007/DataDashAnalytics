<template>
  <div class="dashboard-container">
    <!-- Sidebar -->
    <Sidebar :activeMenu="activeMenu" @update-menu="val => activeMenu = val" />

    <!-- Main Content -->
    <main class="main-content">
      <!-- Header Actions -->
      <!-- <HeaderActions @export="exportData" @refresh="refreshData" /> -->

      <!-- Upload Section -->
      <UploadSection v-if="activeMenu === 'upload'" :is-uploaded="isUploaded" @file-uploaded="handleUpload" />

      <!-- Data Table Section -->
      <DataTableSection v-if="activeMenu === 'table'" :columns="columns" :rows="rows"
        :categorical-columns="categoricalColumns" :filters="filters" :filtered-rows="filteredRows"
        :unique-values="uniqueValues" :clear-filters="clearFilters" />


      <!-- Chart Section -->
      <ChartSection v-if="activeMenu === 'chart'" :columns="columns" :rows="rows" :numeric-columns="numericColumns"
        :x-column="xColumn" :y-columns="yColumns" :chart-type="chartType" />


      <!-- EDA Section -->
      <EDASection v-if="activeMenu === 'eda'" :columns="columns" :rows="rows" :numeric-columns="numericColumns"
        :categorical-columns="categoricalColumns" />

      <!-- AI Insight Section -->
      <AIInsightSection v-if="activeMenu === 'insight'" :columns="columns" :rows="rows"
        :numeric-columns="numericColumns" :insights="insights" @generate-insights="generateInsights" />

        <ExportMenu v-if="activeMenu === 'ExportMenu'" :columns="columns" :filtered-rows="filteredRows" />
      <!-- Footer -->
      <Footer />
    </main>
  </div>
</template>

<script setup>

import Sidebar from '../components/Sidebar.vue'
import ExportMenu from '../components/ExportMenu.vue'

// import HeaderActions from '../components/HeaderActions.vue'
import UploadSection from '../components/UploadSection.vue'
import DataTableSection from '../components/DataTableSection.vue'
import ChartSection from '../components/ChartSection.vue'
import EDASection from '../components/EDASection.vue'
import AIInsightSection from '../components/AIInsightSection.vue'
import Footer from '../components/Footer.vue'


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


const handleUpload = (uploadedData) => {
  columns.value = uploadedData.columns
  rows.value = uploadedData.data
  isUploaded.value = true

  // reset filters
  columns.value.forEach(col => filters[col] = '')

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
.dashboard-container {
  display: flex;
  min-height: 100vh;
  background: #f8fafc;
  position: relative;
}

.main-content {
  flex: 1;
  margin-left: 280px;
  /* Sesuaikan dengan lebar sidebar */
  padding: 24px 32px;
  min-height: 100vh;
  background: #f8fafc;
  overflow-x: hidden;
}

/* Responsive */
@media (max-width: 1024px) {
  .main-content {
    margin-left: 240px;
    padding: 20px 24px;
  }
}

@media (max-width: 768px) {
  .dashboard-container {
    flex-direction: column;
  }

  .main-content {
    margin-left: 0;
    padding: 16px;
    margin-top: 60px;
    /* Untuk sidebar mobile */
  }
}

@media (max-width: 480px) {
  .main-content {
    padding: 12px;
  }
}
</style>