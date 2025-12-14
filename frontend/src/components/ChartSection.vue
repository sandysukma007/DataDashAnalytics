<template>
  <section class="card">
    <div class="card-header">
      <div class="header-content">
        <h3 class="card-title">
          <span class="title-icon">📈</span>
          Data Visualization
        </h3>
        <p class="card-subtitle">Create interactive charts from your dataset</p>
      </div>

      <div class="header-stats" v-if="columns.length">
        <div class="stat-item">
          <span class="stat-icon">📊</span>
          <div class="stat-content">
            <div class="stat-value">{{ numericColumns.length }}</div>
            <div class="stat-label">Numeric Columns</div>
          </div>
        </div>
      </div>
    </div>

    <div class="card-body" v-if="columns.length">
      <!-- Chart Controls -->
      <div class="visualization-controls">
        <div class="controls-section">
          <div class="control-group">
            <h4 class="control-title">
              <span class="control-icon">📍</span>
              Chart Configuration
            </h4>

            <div class="control-grid">
              <!-- X-Axis Control -->
              <div class="control-item">
                <label class="control-label">
                  <span class="label-icon">↔️</span>
                  X-Axis Column
                  <span class="label-hint">(Horizontal axis)</span>
                </label>
                <div class="select-wrapper">
                  <select
                    v-model="selectedX"
                    class="chart-select"
                  >
                    <option value="">Select X-axis</option>
                    <option
                      v-for="col in columns"
                      :key="col"
                      :value="col"
                      :class="{ 'numeric-option': numericColumns.includes(col) }"
                    >
                      {{ col }}
                      <template v-if="numericColumns.includes(col)"> (numeric)</template>
                    </option>
                  </select>
                  <span class="select-icon">▼</span>
                </div>
                <div v-if="selectedX" class="selected-value">
                  Selected: <span class="value-text">{{ selectedX }}</span>
                </div>
              </div>

              <!-- Y-Axis Control -->
              <div class="control-item">
                <label class="control-label">
                  <span class="label-icon">↕️</span>
                  Y-Axis Columns
                  <span class="label-hint">(Vertical axis - multiple select)</span>
                </label>
                <div class="select-wrapper">
                  <select
                    multiple
                    v-model="selectedY"
                    class="chart-select multiple"
                  >
                    <option
                      v-for="col in numericColumns"
                      :key="col"
                      :value="col"
                    >
                      {{ col }}
                    </option>
                  </select>
                  <span class="select-icon multiple">▲▼</span>
                </div>
                <div class="selection-info">
                  <div class="selected-count">
                    Selected: <span class="count-value">{{ selectedY.length }}</span> columns
                  </div>
                  <small class="selection-hint">Hold Ctrl/Cmd to select multiple</small>
                </div>
                <div v-if="selectedY.length > 0" class="selected-columns">
                  <span
                    v-for="col in selectedY"
                    :key="col"
                    class="column-tag"
                    @click="removeColumn(col)"
                  >
                    {{ col }}
                    <span class="remove-icon">×</span>
                  </span>
                </div>
              </div>

              <!-- Chart Type Control -->
              <div class="control-item">
                <label class="control-label">
                  <span class="label-icon">🎨</span>
                  Chart Type
                </label>
                <div class="chart-type-selector">
                  <div
                    v-for="type in chartTypes"
                    :key="type.value"
                    :class="['chart-type-option', { 'active': chartType === type.value }]"
                    @click="chartType = type.value"
                  >
                    <span class="type-icon">{{ type.icon }}</span>
                    <span class="type-label">{{ type.label }}</span>
                  </div>
                </div>
                <div class="type-description">
                  {{ getChartTypeDescription(chartType) }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Advanced Settings -->
        <div class="advanced-controls" v-if="selectedY.length > 0">
          <div class="advanced-toggle" @click="showAdvanced = !showAdvanced">
            <span class="toggle-icon">{{ showAdvanced ? '▼' : '▶' }}</span>
            <span class="toggle-text">Advanced Settings</span>
          </div>

          <div v-if="showAdvanced" class="advanced-content">
            <div class="advanced-grid">
              <!-- Aggregation Method -->
              <div class="advanced-item">
                <label class="advanced-label">
                  <span class="advanced-icon">📊</span>
                  Aggregation Method
                </label>
                <select v-model="aggregation" class="advanced-select">
                  <option value="average">Average</option>
                  <option value="sum">Sum</option>
                  <option value="min">Minimum</option>
                  <option value="max">Maximum</option>
                  <option value="count">Count</option>
                </select>
              </div>

              <!-- Chart Colors -->
              <div class="advanced-item">
                <label class="advanced-label">
                  <span class="advanced-icon">🎨</span>
                  Color Scheme
                </label>
                <div class="color-scheme">
                  <div
                    v-for="scheme in colorSchemes"
                    :key="scheme.name"
                    :class="['color-option', { 'active': colorScheme === scheme.name }]"
                    @click="colorScheme = scheme.name"
                    :style="`background: ${scheme.colors[0]}`"
                    :title="scheme.name"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Chart Container -->
      <div class="chart-section">
        <div class="chart-header">
          <div class="chart-info">
            <h4 class="chart-title">Visualization Preview</h4>
            <div class="chart-summary" v-if="selectedX && selectedY.length > 0">
              <span class="summary-item">
                {{ selectedY.length }} metric{{ selectedY.length > 1 ? 's' : '' }} vs {{ selectedX }}
              </span>
              <span class="summary-item">
                {{ chartType.toUpperCase() }} Chart
              </span>
            </div>
          </div>
          <div class="chart-actions">
            <button class="action-btn" @click="downloadChart" title="Download as PNG">
              <span class="btn-icon">📥</span>
              Download
            </button>
            <button class="action-btn" @click="resetChart" title="Reset to defaults">
              <span class="btn-icon">🔄</span>
              Reset
            </button>
          </div>
        </div>

        <!-- Chart Canvas -->
        <div class="chart-container" :class="{ 'has-data': selectedX && selectedY.length > 0 }">
          <canvas ref="canvas" class="chart-canvas"></canvas>

          <!-- Empty State -->
          <div v-if="!selectedX || selectedY.length === 0" class="chart-empty">
            <div class="empty-icon">📈</div>
            <h4 class="empty-title">Configure Your Chart</h4>
            <p class="empty-description">
              Select X-axis and at least one Y-axis column to generate visualization
            </p>
            <div class="empty-hints">
              <div class="hint-item">
                <span class="hint-icon">1</span>
                <span class="hint-text">Choose X-axis (typically categorical data)</span>
              </div>
              <div class="hint-item">
                <span class="hint-icon">2</span>
                <span class="hint-text">Select Y-axis (numeric columns)</span>
              </div>
              <div class="hint-item">
                <span class="hint-icon">3</span>
                <span class="hint-text">Pick chart type and customize</span>
              </div>
            </div>
          </div>

          <!-- Loading State -->
          <div v-if="isLoading" class="chart-loading">
            <div class="loading-spinner"></div>
            <p class="loading-text">Generating visualization...</p>
          </div>
        </div>
      </div>
    </div>

    <!-- No Data State -->
    <div v-else class="card-body">
      <div class="empty-state">
        <div class="empty-icon">📊</div>
        <h4 class="empty-title">No Data Available</h4>
        <p class="empty-description">Upload a dataset to create visualizations</p>
        <button class="empty-action" @click="$emit('navigate-upload')">
          <span class="action-icon">📤</span>
          Upload Data
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, watch, nextTick, computed, onMounted } from 'vue'
import Chart from 'chart.js/auto'

const props = defineProps({
  columns: Array,
  rows: Array
})

const emit = defineEmits(['navigate-upload'])

const canvas = ref(null)
let chartInstance = null

// State
const selectedX = ref('')
const selectedY = ref([])
const chartType = ref('line')
const showAdvanced = ref(false)
const aggregation = ref('average')
const colorScheme = ref('vibrant')
const isLoading = ref(false)

// Chart types with icons
const chartTypes = [
  { value: 'line', label: 'Line Chart', icon: '📈' },
  { value: 'bar', label: 'Bar Chart', icon: '📊' },
  { value: 'scatter', label: 'Scatter Plot', icon: '•' },
  { value: 'pie', label: 'Pie Chart', icon: '🥧' },
  { value: 'doughnut', label: 'Doughnut', icon: '🍩' }
]

// Color schemes with enough colors for multiple datasets
const colorSchemes = [
  {
    name: 'vibrant',
    colors: ['#667eea', '#764ba2', '#f093fb', '#f5576c', '#ffd166', '#06d6a0', '#118ab2', '#ef476f', '#ff9a76', '#7bdff2']
  },
  {
    name: 'pastel',
    colors: ['#a5b4fc', '#c4b5fd', '#f0abfc', '#f9a8d4', '#fecaca', '#bbf7d0', '#a7f3d0', '#99f6e4', '#c7d2fe', '#ddd6fe']
  },
  {
    name: 'corporate',
    colors: ['#3b82f6', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6', '#06b6d4', '#84cc16', '#f97316', '#ec4899', '#6366f1']
  },
  {
    name: 'dark',
    colors: ['#1e3a8a', '#065f46', '#92400e', '#7c2d12', '#4c1d95', '#0e7490', '#3f6212', '#9a3412', '#831843', '#3730a3']
  }
]

// Filter kolom numerik
const numericColumns = computed(() =>
  props.columns.filter(col =>
    props.rows.every(r => !isNaN(parseFloat(r[col])))
  )
)

// Set default values
watch(() => props.columns, (cols) => {
  if (cols.length > 0) {
    selectedX.value = cols[0] || ''
    selectedY.value = numericColumns.value.slice(0, Math.min(2, numericColumns.value.length))
  }
}, { immediate: true })

// Watch for changes and update chart
watch([() => props.rows, selectedX, selectedY, chartType, aggregation, colorScheme],
  async () => {
    if (!props.rows.length || !selectedX.value || !selectedY.value.length) return

    isLoading.value = true
    await nextTick()

    if (!canvas.value) {
      isLoading.value = false
      return
    }

    try {
      if (chartInstance) chartInstance.destroy()

      const data = prepareChartData()
      const colors = getColorScheme(data.datasets.length)
      const chartConfig = getChartConfig(data, colors)

      chartInstance = new Chart(canvas.value, chartConfig)
    } catch (error) {
      console.error('Error creating chart:', error)
    } finally {
      isLoading.value = false
    }
  },
  { deep: true }
)

// Methods
const prepareChartData = () => {
  const grouped = {}

  // Group data by X-axis
  props.rows.forEach(row => {
    const xValue = row[selectedX.value]
    if (!grouped[xValue]) grouped[xValue] = {}

    selectedY.value.forEach(yCol => {
      if (!grouped[xValue][yCol]) grouped[xValue][yCol] = []
      const value = Number(row[yCol]) || 0
      if (!isNaN(value)) {
        grouped[xValue][yCol].push(value)
      }
    })
  })

  const labels = Object.keys(grouped)

  const datasets = selectedY.value.map(yCol => {
    const data = labels.map(xValue => {
      const values = grouped[xValue][yCol] || [0]

      switch (aggregation.value) {
        case 'sum': return values.reduce((a, b) => a + b, 0)
        case 'min': return Math.min(...values)
        case 'max': return Math.max(...values)
        case 'count': return values.length
        case 'average':
        default: return values.reduce((a, b) => a + b, 0) / values.length
      }
    })

    return {
      label: yCol,
      data
    }
  })

  return { labels, datasets }
}

const getColorScheme = (numColors) => {
  const scheme = colorSchemes.find(s => s.name === colorScheme.value) || colorSchemes[0]
  // Return only the number of colors we need
  return scheme.colors.slice(0, Math.max(numColors, scheme.colors.length))
}

const getChartConfig = (data, colors) => {
  const isSingleDataset = data.datasets.length === 1

  if (chartType.value === 'pie' || chartType.value === 'doughnut') {
    // For pie/doughnut charts, we need special handling
    if (isSingleDataset && data.labels.length > 0) {
      // Single dataset with multiple categories (labels)
      return getPieChartConfig(data, colors)
    } else {
      // Multiple datasets - not ideal for pie/doughnut
      return getMultiDatasetPieConfig(data, colors)
    }
  } else {
    // For line, bar, scatter charts
    return getStandardChartConfig(data, colors)
  }
}

const getStandardChartConfig = (data, colors) => {
  const datasets = data.datasets.map((dataset, index) => {
    const colorIndex = index % colors.length
    const baseColor = colors[colorIndex]

    return {
      ...dataset,
      backgroundColor: chartType.value === 'line'
        ? baseColor + '20'  // 20% opacity for line charts
        : baseColor,
      borderColor: baseColor,
      borderWidth: chartType.value === 'line' ? 3 : 2,
      pointBackgroundColor: baseColor,
      pointBorderColor: '#ffffff',
      pointBorderWidth: 2,
      pointRadius: chartType.value === 'line' ? 4 : 5,
      pointHoverRadius: 6,
      tension: chartType.value === 'line' ? 0.4 : 0,
      fill: chartType.value === 'line'
    }
  })

  return {
    type: chartType.value,
    data: {
      labels: data.labels,
      datasets
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'top',
          labels: {
            padding: 20,
            usePointStyle: true,
            font: { size: 13, family: "'Inter', sans-serif" }
          }
        },
        tooltip: {
          mode: chartType.value === 'scatter' ? 'point' : 'index',
          intersect: false,
          backgroundColor: 'rgba(0, 0, 0, 0.8)',
          titleFont: { size: 12 },
          bodyFont: { size: 13 },
          padding: 12,
          callbacks: {
            label: (context) => {
              let label = context.dataset.label || ''
              if (label) label += ': '
              label += context.parsed.y.toLocaleString()
              return label
            }
          }
        }
      },
      scales: {
        x: {
          grid: { display: false },
          ticks: { font: { size: 12 } }
        },
        y: {
          beginAtZero: chartType.value !== 'scatter',
          grid: { color: 'rgba(0, 0, 0, 0.05)' },
          ticks: {
            font: { size: 12 },
            callback: (value) => value.toLocaleString()
          }
        }
      },
      interaction: {
        mode: chartType.value === 'scatter' ? 'point' : 'nearest',
        intersect: chartType.value === 'scatter'
      },
      animation: {
        duration: 1000,
        easing: 'easeOutQuart'
      }
    }
  }
}

const getPieChartConfig = (data, colors) => {
  // For single dataset, use labels for segments
  const dataset = data.datasets[0]

  return {
    type: chartType.value,
    data: {
      labels: data.labels,
      datasets: [{
        ...dataset,
        backgroundColor: colors.slice(0, data.labels.length),
        borderColor: '#ffffff',
        borderWidth: 2,
        hoverOffset: 15
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'right',
          labels: {
            padding: 20,
            font: { size: 13 }
          }
        },
        tooltip: {
          callbacks: {
            label: (context) => {
              const label = context.label || ''
              const value = context.raw || 0
              const total = dataset.data.reduce((a, b) => a + b, 0)
              const percentage = total > 0 ? ((value / total) * 100).toFixed(1) : 0
              return `${label}: ${value.toLocaleString()} (${percentage}%)`
            }
          }
        }
      },
      animation: {
        animateScale: true,
        animateRotate: true
      }
    }
  }
}

const getMultiDatasetPieConfig = (data, colors) => {
  // For multiple datasets in pie/doughnut (less common but we'll handle it)
  // We'll create a stacked pie chart or show first dataset
  const datasets = data.datasets.map((dataset, index) => {
    const colorIndex = index % colors.length
    return {
      ...dataset,
      backgroundColor: colors[colorIndex],
      borderColor: '#ffffff',
      borderWidth: 2
    }
  })

  return {
    type: chartType.value,
    data: {
      labels: data.labels,
      datasets: datasets.slice(0, 1) // Show only first dataset for simplicity
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'right',
          labels: {
            padding: 20,
            font: { size: 13 }
          }
        },
        tooltip: {
          callbacks: {
            label: (context) => {
              const label = context.dataset.label || ''
              const value = context.raw || 0
              return `${label}: ${value.toLocaleString()}`
            }
          }
        }
      }
    }
  }
}

const getChartTypeDescription = (type) => {
  const descriptions = {
    line: 'Shows trends and changes over time',
    bar: 'Compares different categories',
    scatter: 'Shows relationships between variables',
    pie: 'Shows proportions of a whole (single Y-column)',
    doughnut: 'Similar to pie chart with center hole (single Y-column)'
  }
  return descriptions[type] || 'Visualize your data'
}

const removeColumn = (col) => {
  selectedY.value = selectedY.value.filter(c => c !== col)
}

const downloadChart = () => {
  if (!chartInstance) return

  const link = document.createElement('a')
  link.download = `chart-${new Date().getTime()}.png`
  link.href = canvas.value.toDataURL('image/png')
  link.click()
}

const resetChart = () => {
  if (props.columns.length > 0) {
    selectedX.value = props.columns[0]
    selectedY.value = numericColumns.value.slice(0, 2)
  }
  chartType.value = 'line'
  aggregation.value = 'average'
  colorScheme.value = 'vibrant'
  showAdvanced.value = false
}

// Add warning for pie/doughnut with multiple Y columns
watch([selectedY, chartType], () => {
  if ((chartType.value === 'pie' || chartType.value === 'doughnut') && selectedY.value.length > 1) {
    console.warn('Pie/Doughnut charts work best with a single Y-column. Showing first Y-column only.')
  }
}, { immediate: true })

onMounted(() => {
  if (props.columns.length > 0 && selectedX.value && selectedY.value.length > 0) {
    nextTick(() => {
      if (canvas.value && props.rows.length > 0) {
        selectedY.value = [...selectedY.value]
      }
    })
  }
})
</script>

<style scoped>
/* Card Styling */
.card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  border: 1px solid #e5e7eb;
}

.card-header {
  padding: 24px 32px;
  background: linear-gradient(135deg, #667eea15 0%, #764ba215 100%);
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-content {
  flex: 1;
}

.card-title {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
  margin: 0 0 8px 0;
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

.header-stats {
  margin-left: 24px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 20px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  min-width: 160px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.stat-icon {
  font-size: 24px;
  color: #667eea;
  background: rgba(102, 126, 234, 0.1);
  width: 40px;
  height: 40px;
  border-radius: 10px;
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
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* Card Body */
.card-body {
  padding: 32px;
}

/* Visualization Controls */
.visualization-controls {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 32px;
}

.controls-section {
  margin-bottom: 20px;
}

.control-group {
  margin-bottom: 0;
}

.control-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 20px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.control-icon {
  color: #667eea;
}

.control-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

.control-item {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.control-label {
  font-size: 15px;
  font-weight: 600;
  color: #374151;
  display: flex;
  align-items: center;
  gap: 8px;
}

.label-icon {
  font-size: 18px;
}

.label-hint {
  font-size: 12px;
  color: #6b7280;
  font-weight: 400;
  margin-left: auto;
}

.select-wrapper {
  position: relative;
}

.chart-select {
  width: 100%;
  padding: 12px 16px;
  padding-right: 40px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  color: #374151;
  background: white;
  appearance: none;
  cursor: pointer;
  transition: all 0.3s ease;
}

.chart-select.multiple {
  min-height: 120px;
  padding-right: 40px;
}

.chart-select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.select-icon {
  position: absolute;
  right: 16px;
  top: 50%;
  transform: translateY(-50%);
  color: #6b7280;
  pointer-events: none;
}

.select-icon.multiple {
  top: 20px;
}

.numeric-option {
  color: #3b82f6;
  font-weight: 500;
}

.selected-value {
  font-size: 13px;
  color: #6b7280;
  padding: 8px 12px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
}

.value-text {
  color: #667eea;
  font-weight: 600;
}

.selection-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.selected-count {
  font-size: 13px;
  color: #374151;
  font-weight: 500;
}

.count-value {
  color: #667eea;
  font-weight: 600;
}

.selection-hint {
  color: #9ca3af;
  font-size: 11px;
}

.selected-columns {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 8px;
}

.column-tag {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border: 1px solid rgba(102, 126, 234, 0.2);
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.column-tag:hover {
  background: rgba(102, 126, 234, 0.2);
  transform: translateY(-1px);
}

.remove-icon {
  font-size: 16px;
  line-height: 1;
  margin-left: 2px;
}

.chart-type-selector {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 8px;
}

.chart-type-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 16px 8px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  background: white;
  cursor: pointer;
  transition: all 0.3s ease;
  text-align: center;
}

.chart-type-option:hover {
  border-color: #667eea;
  transform: translateY(-2px);
}

.chart-type-option.active {
  border-color: #667eea;
  background: rgba(102, 126, 234, 0.1);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.1);
}

.type-icon {
  font-size: 24px;
  margin-bottom: 4px;
}

.type-label {
  font-size: 12px;
  font-weight: 600;
  color: #374151;
}

.type-description {
  font-size: 13px;
  color: #6b7280;
  padding: 12px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  margin-top: 8px;
}

/* Advanced Controls */
.advanced-controls {
  border-top: 1px solid #e5e7eb;
  padding-top: 20px;
  margin-top: 20px;
}

.advanced-toggle {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.advanced-toggle:hover {
  background: #f3f4f6;
  border-color: #d1d5db;
}

.toggle-icon {
  font-size: 12px;
  color: #6b7280;
}

.toggle-text {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

.advanced-content {
  margin-top: 16px;
  animation: slideDown 0.3s ease;
}

.advanced-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.advanced-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.advanced-label {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  display: flex;
  align-items: center;
  gap: 8px;
}

.advanced-icon {
  font-size: 16px;
  color: #667eea;
}

.advanced-select {
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  color: #374151;
  background: white;
  cursor: pointer;
}

.color-scheme {
  display: flex;
  gap: 8px;
}

.color-option {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  cursor: pointer;
  border: 2px solid transparent;
  transition: all 0.3s ease;
}

.color-option:hover {
  transform: scale(1.1);
}

.color-option.active {
  border-color: #374151;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Chart Section */
.chart-section {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  background: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.chart-info {
  flex: 1;
}

.chart-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
}

.chart-summary {
  display: flex;
  gap: 16px;
  align-items: center;
}

.summary-item {
  font-size: 14px;
  color: #6b7280;
  background: white;
  padding: 4px 12px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
}

.chart-actions {
  display: flex;
  gap: 8px;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  color: #374151;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.action-btn:hover {
  background: #f3f4f6;
  border-color: #d1d5db;
  transform: translateY(-1px);
}

.btn-icon {
  font-size: 14px;
}

.chart-container {
  position: relative;
  height: 500px;
  padding: 24px;
}

.chart-container.has-data {
  height: 500px;
}

.chart-canvas {
  width: 100% !important;
  height: 100% !important;
}

/* Chart Empty State */
.chart-empty {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  text-align: center;
  background: white;
}

.empty-icon {
  font-size: 64px;
  margin-bottom: 20px;
  opacity: 0.3;
}

.empty-title {
  font-size: 20px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 12px;
}

.empty-description {
  color: #6b7280;
  font-size: 15px;
  max-width: 400px;
  margin: 0 auto 24px;
}

.empty-hints {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-width: 300px;
  margin: 0 auto;
}

.hint-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #f9fafb;
  border-radius: 8px;
  text-align: left;
}

.hint-icon {
  width: 24px;
  height: 24px;
  background: #667eea;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 600;
  flex-shrink: 0;
}

.hint-text {
  font-size: 13px;
  color: #374151;
}

/* Loading State */
.chart-loading {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.9);
  z-index: 10;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #f3f4f6;
  border-top-color: #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

.loading-text {
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* No Data State */
.empty-state {
  text-align: center;
  padding: 60px 40px;
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border-radius: 12px;
  border: 2px dashed #d1d5db;
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
  color: #374151;
  margin-bottom: 8px;
}

.empty-description {
  color: #6b7280;
  font-size: 15px;
  margin-bottom: 24px;
  max-width: 300px;
  margin: 0 auto 24px;
}

.empty-action {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
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
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

/* Responsive Design */
@media (max-width: 1024px) {
  .card-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }

  .header-stats {
    margin-left: 0;
    align-self: stretch;
  }

  .stat-item {
    width: 100%;
  }

  .control-grid {
    grid-template-columns: 1fr;
  }

  .chart-container {
    height: 400px;
  }

  .chart-container.has-data {
    height: 400px;
  }
}

@media (max-width: 768px) {
  .card-header {
    padding: 20px 24px;
  }

  .card-title {
    font-size: 20px;
  }

  .card-body {
    padding: 24px;
  }

  .visualization-controls {
    padding: 20px;
  }

  .chart-type-selector {
    grid-template-columns: repeat(3, 1fr);
  }

  .chart-header {
    flex-direction: column;
    gap: 12px;
    align-items: stretch;
  }

  .chart-actions {
    justify-content: center;
  }

  .chart-container {
    height: 350px;
    padding: 16px;
  }

  .chart-container.has-data {
    height: 350px;
  }

  .empty-state {
    padding: 40px 24px;
  }
}

@media (max-width: 480px) {
  .card-header {
    padding: 16px 20px;
  }

  .stat-item {
    min-width: auto;
    padding: 10px 16px;
  }

  .card-body {
    padding: 20px;
  }

  .visualization-controls {
    padding: 16px;
  }

  .chart-type-selector {
    grid-template-columns: repeat(2, 1fr);
  }

  .chart-container {
    height: 300px;
  }

  .chart-container.has-data {
    height: 300px;
  }

  .empty-state {
    padding: 32px 20px;
  }

  .empty-icon {
    font-size: 48px;
  }
}

.warning-badge {
  background: #fef3c7;
  color: #92400e;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  margin-left: 8px;
  border: 1px solid #fbbf24;
}
</style>