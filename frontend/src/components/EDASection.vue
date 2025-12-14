<template>
  <section class="card">
    <div class="card-header">
      <div class="header-content">
        <h3 class="card-title">
          <span class="title-icon">🔍</span>
          Exploratory Data Analysis
        </h3>
        <p class="card-subtitle">Automated insights and data profiling</p>
      </div>

      <div class="header-actions" v-if="columns.length">
        <button class="action-btn" @click="exportReport" title="Export Report">
          <span class="btn-icon">📥</span>
          Export
        </button>
        <button class="action-btn" @click="refreshAnalysis" title="Refresh Analysis">
          <span class="btn-icon">🔄</span>
          Refresh
        </button>
      </div>
    </div>

    <div class="card-body">
      <div v-if="columns.length && rows.length">
        <!-- Overview Stats -->
        <div class="overview-section">
          <h4 class="section-title">
            <span class="section-icon">📊</span>
            Dataset Overview
          </h4>

          <div class="stats-grid">
            <div class="stat-card primary">
              <div class="stat-icon">📋</div>
              <div class="stat-content">
                <div class="stat-value">{{ rows.length }}</div>
                <div class="stat-label">Total Rows</div>
                <div class="stat-change" v-if="rowStats">
                  <span class="change-icon">📈</span>
                  <span class="change-text">{{ rowStats.uniquePercentage }}% unique</span>
                </div>
              </div>
            </div>

            <div class="stat-card secondary">
              <div class="stat-icon">📊</div>
              <div class="stat-content">
                <div class="stat-value">{{ columns.length }}</div>
                <div class="stat-label">Total Columns</div>
                <div class="stat-breakdown">
                  <span class="breakdown-item numeric">{{ numericCount }}</span>
                  <span class="breakdown-item categorical">{{ categoricalCount }}</span>
                </div>
              </div>
            </div>

            <div class="stat-card success">
              <div class="stat-icon">✅</div>
              <div class="stat-content">
                <div class="stat-value">{{ completenessPercentage }}%</div>
                <div class="stat-label">Data Completeness</div>
                <div class="stat-change">
                  <span class="change-icon">{{ completenessIcon }}</span>
                  <span class="change-text">{{ missingValues }} missing values</span>
                </div>
              </div>
            </div>

            <div class="stat-card warning">
              <div class="stat-icon">⚠️</div>
              <div class="stat-content">
                <div class="stat-value">{{ duplicateRows }}</div>
                <div class="stat-label">Duplicate Rows</div>
                <div class="stat-change" v-if="duplicateRows > 0">
                  <span class="change-icon">🔍</span>
                  <span class="change-text">{{ duplicatePercentage }}% of data</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Column Analysis -->
        <div class="analysis-section">
          <div class="section-header">
            <h4 class="section-title">
              <span class="section-icon">🔢</span>
              Column Analysis
            </h4>
            <div class="section-filter">
              <span class="filter-label">Show:</span>
              <div class="filter-buttons">
                <button
                  :class="['filter-btn', { active: columnFilter === 'all' }]"
                  @click="columnFilter = 'all'"
                >
                  All
                </button>
                <button
                  :class="['filter-btn', { active: columnFilter === 'numeric' }]"
                  @click="columnFilter = 'numeric'"
                >
                  Numeric
                </button>
                <button
                  :class="['filter-btn', { active: columnFilter === 'categorical' }]"
                  @click="columnFilter = 'categorical'"
                >
                  Categorical
                </button>
              </div>
            </div>
          </div>

          <div class="columns-grid">
            <div
              v-for="col in filteredColumns"
              :key="col.name"
              class="column-card"
              :class="{ 'numeric': col.type === 'numeric', 'categorical': col.type === 'categorical' }"
            >
              <div class="column-header">
                <div class="column-info">
                  <h5 class="column-name">{{ col.name }}</h5>
                  <span class="column-type" :class="col.type">{{ col.type }}</span>
                </div>
                <span class="column-index">#{{ col.index }}</span>
              </div>

              <div class="column-stats">
                <div class="stat-row">
                  <div class="stat-item">
                    <span class="stat-label">Type</span>
                    <span class="stat-value">{{ col.dataType }}</span>
                  </div>
                  <div class="stat-item">
                    <span class="stat-label">Unique</span>
                    <span class="stat-value">{{ col.uniqueCount }}</span>
                  </div>
                </div>

                <div class="stat-row">
                  <div class="stat-item">
                    <span class="stat-label">Missing</span>
                    <span class="stat-value">{{ col.missingCount }}</span>
                  </div>
                  <div class="stat-item">
                    <span class="stat-label">% Complete</span>
                    <span class="stat-value" :class="getCompletenessClass(col.completeness)">
                      {{ col.completeness }}%
                    </span>
                  </div>
                </div>

                <div v-if="col.type === 'numeric'" class="numeric-stats">
                  <div class="range-bar">
                    <div class="range-labels">
                      <span class="range-min">{{ formatNumber(col.min) }}</span>
                      <span class="range-avg">Avg: {{ formatNumber(col.avg) }}</span>
                      <span class="range-max">{{ formatNumber(col.max) }}</span>
                    </div>
                    <div class="range-track">
                      <div
                        class="range-progress"
                        :style="{ width: col.rangePercentage + '%' }"
                      ></div>
                    </div>
                  </div>
                </div>

                <div v-if="col.type === 'categorical' && col.topValues" class="categorical-stats">
                  <div class="top-values">
                    <div
                      v-for="(value, idx) in col.topValues.slice(0, 3)"
                      :key="idx"
                      class="value-tag"
                      :title="`${value.value}: ${value.percentage}%`"
                    >
                      <span class="value-text">{{ value.value }}</span>
                      <span class="value-percentage">{{ value.percentage }}%</span>
                    </div>
                    <div v-if="col.topValues.length > 3" class="more-values">
                      +{{ col.topValues.length - 3 }} more
                    </div>
                  </div>
                </div>
              </div>

              <div class="column-actions">
                <button class="action-btn small" @click="visualizeColumn(col)" title="Visualize">
                  <span class="btn-icon">📈</span>
                </button>
                <button class="action-btn small" @click="analyzeColumn(col)" title="Analyze">
                  <span class="btn-icon">🔍</span>
                </button>
              </div>
            </div>
          </div>

          <div v-if="filteredColumns.length === 0" class="empty-columns">
            <div class="empty-icon">📊</div>
            <p class="empty-text">No columns match the selected filter</p>
          </div>
        </div>

        <!-- Data Quality Insights -->
        <div class="insights-section">
          <h4 class="section-title">
            <span class="section-icon">💡</span>
            Data Quality Insights
          </h4>

          <div class="insights-grid">
            <div class="insight-card" :class="getInsightClass(insight.type)"
                 v-for="insight in dataInsights" :key="insight.id">
              <div class="insight-icon">{{ insight.icon }}</div>
              <div class="insight-content">
                <h5 class="insight-title">{{ insight.title }}</h5>
                <p class="insight-description">{{ insight.description }}</p>
                <!-- <div class="insight-actions">
                  <button class="insight-btn" @click="handleInsightAction(insight)">
                    {{ insight.actionText }}
                  </button>
                </div> -->
              </div>
            </div>
          </div>
        </div>

        <!-- Summary Statistics -->
        <div class="summary-section">
          <h4 class="section-title">
            <span class="section-icon">📋</span>
            Summary Statistics
          </h4>

          <div class="summary-content">
            <div class="summary-stats">
              <div class="summary-item">
                <span class="summary-label">Dataset Size</span>
                <span class="summary-value">{{ formatBytes(datasetSize) }}</span>
              </div>
              <div class="summary-item">
                <span class="summary-label">Memory Usage</span>
                <span class="summary-value">{{ formatBytes(memoryUsage) }}</span>
              </div>
              <div class="summary-item">
                <span class="summary-label">Analysis Time</span>
                <span class="summary-value">{{ analysisTime }}ms</span>
              </div>
            </div>

            <div class="summary-notes">
              <h5 class="notes-title">Key Findings</h5>
              <ul class="notes-list">
                <li v-for="(note, index) in keyFindings" :key="index">{{ note }}</li>
              </ul>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else class="empty-state">
        <div class="empty-icon">🔍</div>
        <h4 class="empty-title">No Data for Analysis</h4>
        <p class="empty-description">Upload a dataset to perform exploratory data analysis</p>
        <button class="empty-action" @click="$emit('navigate-upload')">
          <span class="action-icon">📤</span>
          Upload Data
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const props = defineProps(['columns', 'rows'])
const emit = defineEmits(['navigate-upload', 'visualize-column', 'analyze-column'])

// State
const columnFilter = ref('all')
const analysisTime = ref(0)
const dataInsights = ref([])
const keyFindings = ref([])

// Computed properties
const numericColumns = computed(() => {
  if (!props.columns.length || !props.rows.length) return []

  return props.columns.filter(col => {
    // Check if column contains mostly numeric values
    const numericCount = props.rows.filter(row => {
      const value = row[col]
      return !isNaN(parseFloat(value)) && isFinite(value)
    }).length

    return numericCount > props.rows.length * 0.7 // At least 70% numeric
  })
})

const categoricalColumns = computed(() => {
  if (!props.columns.length || !props.rows.length) return []

  return props.columns.filter(col => !numericColumns.value.includes(col))
})

const numericCount = computed(() => numericColumns.value.length)
const categoricalCount = computed(() => categoricalColumns.value.length)

const missingValues = computed(() => {
  if (!props.columns.length || !props.rows.length) return 0

  let missing = 0
  props.columns.forEach(col => {
    props.rows.forEach(row => {
      if (row[col] === null || row[col] === undefined || row[col] === '') {
        missing++
      }
    })
  })
  return missing
})

const completenessPercentage = computed(() => {
  if (!props.columns.length || !props.rows.length) return 100

  const totalCells = props.columns.length * props.rows.length
  const completeCells = totalCells - missingValues.value
  return Math.round((completeCells / totalCells) * 100)
})

const completenessIcon = computed(() => {
  if (completenessPercentage.value >= 95) return '✅'
  if (completenessPercentage.value >= 80) return '⚠️'
  return '❌'
})

const duplicateRows = computed(() => {
  if (!props.columns.length || !props.rows.length) return 0

  const seen = new Set()
  let duplicates = 0

  props.rows.forEach(row => {
    const key = JSON.stringify(row)
    if (seen.has(key)) {
      duplicates++
    } else {
      seen.add(key)
    }
  })

  return duplicates
})

const duplicatePercentage = computed(() => {
  if (!props.rows.length) return 0
  return Math.round((duplicateRows.value / props.rows.length) * 100)
})

const rowStats = computed(() => {
  if (!props.columns.length || !props.rows.length) return null

  const uniqueRows = new Set(props.rows.map(row => JSON.stringify(row))).size
  const uniquePercentage = Math.round((uniqueRows / props.rows.length) * 100)

  return {
    uniqueRows,
    uniquePercentage
  }
})

const datasetSize = computed(() => {
  if (!props.columns.length || !props.rows.length) return 0
  // Rough estimate: each character ~ 1 byte
  return JSON.stringify(props.rows).length
})

const memoryUsage = computed(() => {
  // Rough memory usage estimate
  return datasetSize.value * 1.5
})

const filteredColumns = computed(() => {
  if (!props.columns.length || !props.rows.length) return []

  const columns = props.columns.map((col, index) => {
    const columnData = props.rows.map(row => row[col])
    const isNumeric = numericColumns.value.includes(col)
    const uniqueValues = [...new Set(columnData.filter(v => v != null))]
    const missingCount = columnData.filter(v => v == null || v === '').length
    const completeness = Math.round(((columnData.length - missingCount) / columnData.length) * 100)

    const columnInfo = {
      name: col,
      index: index + 1,
      type: isNumeric ? 'numeric' : 'categorical',
      dataType: isNumeric ? 'number' : 'string',
      uniqueCount: uniqueValues.length,
      missingCount,
      completeness
    }

    if (isNumeric) {
      const numericValues = columnData.filter(v => !isNaN(parseFloat(v))).map(v => parseFloat(v))
      if (numericValues.length > 0) {
        const min = Math.min(...numericValues)
        const max = Math.max(...numericValues)
        const avg = numericValues.reduce((a, b) => a + b, 0) / numericValues.length
        const range = max - min

        columnInfo.min = min
        columnInfo.max = max
        columnInfo.avg = avg
        columnInfo.range = range
        columnInfo.rangePercentage = range > 0 ? Math.min(((avg - min) / range) * 100, 100) : 0
      }
    } else {
      // For categorical columns, find top values
      const valueCounts = {}
      columnData.filter(v => v != null).forEach(v => {
        valueCounts[v] = (valueCounts[v] || 0) + 1
      })

      const topValues = Object.entries(valueCounts)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 5)
        .map(([value, count]) => ({
          value: value.length > 15 ? value.substring(0, 15) + '...' : value,
          count,
          percentage: Math.round((count / columnData.length) * 100)
        }))

      columnInfo.topValues = topValues
    }

    return columnInfo
  })

  // Apply filter
  return columns.filter(col => {
    if (columnFilter.value === 'all') return true
    if (columnFilter.value === 'numeric') return col.type === 'numeric'
    if (columnFilter.value === 'categorical') return col.type === 'categorical'
    return true
  })
})

// Methods
const formatNumber = (num) => {
  if (num === undefined || num === null) return '-'
  if (typeof num === 'number') {
    if (Number.isInteger(num)) {
      return num.toLocaleString()
    } else {
      return num.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      })
    }
  }
  return num
}

const formatBytes = (bytes) => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

const getCompletenessClass = (percentage) => {
  if (percentage >= 95) return 'excellent'
  if (percentage >= 80) return 'good'
  if (percentage >= 60) return 'fair'
  return 'poor'
}

const getInsightClass = (type) => {
  const classes = {
    info: 'info',
    warning: 'warning',
    error: 'error',
    success: 'success'
  }
  return classes[type] || 'info'
}

const visualizeColumn = (column) => {
  emit('visualize-column', column)
}

const analyzeColumn = (column) => {
  emit('analyze-column', column)
}

const exportReport = () => {
  const report = {
    timestamp: new Date().toISOString(),
    dataset: {
      rows: props.rows.length,
      columns: props.columns.length,
      numericColumns: numericCount.value,
      categoricalColumns: categoricalCount.value
    },
    quality: {
      completeness: completenessPercentage.value,
      missingValues: missingValues.value,
      duplicateRows: duplicateRows.value
    },
    columns: filteredColumns.value
  }

  const dataStr = JSON.stringify(report, null, 2)
  const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr)

  const exportFileDefaultName = `eda-report-${new Date().getTime()}.json`

  const linkElement = document.createElement('a')
  linkElement.setAttribute('href', dataUri)
  linkElement.setAttribute('download', exportFileDefaultName)
  linkElement.click()
}

const refreshAnalysis = () => {
  // Trigger recomputation of insights
  generateInsights()
}

const handleInsightAction = (insight) => {
  switch (insight.action) {
    case 'clean':
      console.log('Cleaning action triggered')
      break
    case 'visualize':
      console.log('Visualization action triggered')
      break
    case 'analyze':
      console.log('Analysis action triggered')
      break
  }
}

const generateInsights = () => {
  const insights = []

  // Data quality insights
  if (completenessPercentage.value < 80) {
    insights.push({
      id: 'missing-data',
      type: 'warning',
      icon: '⚠️',
      title: 'Missing Data Detected',
      description: `${missingValues.value} missing values found. Consider data imputation or cleaning.`,
      action: 'clean',
      actionText: 'Clean Data'
    })
  }

  if (duplicateRows.value > 0) {
    insights.push({
      id: 'duplicates',
      type: 'warning',
      icon: '🔍',
      title: 'Duplicate Rows Found',
      description: `${duplicateRows.value} duplicate rows (${duplicatePercentage.value}% of data) detected.`,
      action: 'clean',
      actionText: 'Remove Duplicates'
    })
  }

  // Column insights
  filteredColumns.value.forEach(col => {
    if (col.completeness < 60) {
      insights.push({
        id: `column-${col.name}-missing`,
        type: 'error',
        icon: '❌',
        title: `High Missing Values in ${col.name}`,
        description: `${col.missingCount} missing values (${100 - col.completeness}% missing).`,
        action: 'analyze',
        actionText: 'Analyze Column'
      })
    }

    if (col.type === 'categorical' && col.uniqueCount > 50) {
      insights.push({
        id: `column-${col.name}-high-cardinality`,
        type: 'warning',
        icon: '📊',
        title: `High Cardinality in ${col.name}`,
        description: `${col.uniqueCount} unique values. Consider grouping or encoding.`,
        action: 'analyze',
        actionText: 'Explore Values'
      })
    }
  })

  // Positive insights
  if (completenessPercentage.value >= 95) {
    insights.push({
      id: 'excellent-quality',
      type: 'success',
      icon: '✅',
      title: 'Excellent Data Quality',
      description: 'Your dataset has minimal missing values and good completeness.',
      action: 'visualize',
      // actionText: 'Create Visualizations'
    })
  }

  if (numericCount.value > 0) {
    insights.push({
      id: 'numeric-analysis',
      type: 'info',
      icon: '📈',
      title: 'Numeric Columns Available',
      description: `${numericCount.value} numeric columns ready for statistical analysis and visualization.`,
      action: 'visualize',
      // actionText: 'Create Charts'
    })
  }

  dataInsights.value = insights.slice(0, 4) // Show top 4 insights

  // Generate key findings
  generateKeyFindings()
}

const generateKeyFindings = () => {
  const findings = []

  findings.push(`Dataset contains ${props.rows.length} rows and ${props.columns.length} columns`)
  findings.push(`${numericCount.value} numeric columns and ${categoricalCount.value} categorical columns identified`)

  if (completenessPercentage.value >= 90) {
    findings.push('Data completeness is excellent (above 90%)')
  } else if (completenessPercentage.value >= 70) {
    findings.push('Data completeness is acceptable (above 70%)')
  } else {
    findings.push('Data completeness needs improvement')
  }

  if (duplicateRows.value === 0) {
    findings.push('No duplicate rows detected')
  } else {
    findings.push(`${duplicateRows.value} duplicate rows found`)
  }

  // Find column with most unique values
  if (filteredColumns.value.length > 0) {
    const mostUnique = [...filteredColumns.value].sort((a, b) => b.uniqueCount - a.uniqueCount)[0]
    findings.push(`${mostUnique.name} has the highest unique values (${mostUnique.uniqueCount})`)
  }

  keyFindings.value = findings
}

// Lifecycle
onMounted(() => {
  if (props.columns.length && props.rows.length) {
    const startTime = performance.now()
    generateInsights()
    const endTime = performance.now()
    analysisTime.value = Math.round(endTime - startTime)
  }
})

watch(() => props.columns, () => {
  if (props.columns.length && props.rows.length) {
    generateInsights()
  }
}, { deep: true })
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

.header-actions {
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

.action-btn.small {
  padding: 6px 10px;
  font-size: 12px;
}

.btn-icon {
  font-size: 14px;
}

/* Card Body */
.card-body {
  padding: 32px;
}

/* Overview Section */
.overview-section {
  margin-bottom: 32px;
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 20px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.section-icon {
  color: #667eea;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
}

.stat-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.stat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
}

.stat-card.primary::before { background: linear-gradient(90deg, #667eea, #764ba2); }
.stat-card.secondary::before { background: linear-gradient(90deg, #3b82f6, #1d4ed8); }
.stat-card.success::before { background: linear-gradient(90deg, #10b981, #059669); }
.stat-card.warning::before { background: linear-gradient(90deg, #f59e0b, #d97706); }

.stat-icon {
  font-size: 32px;
  width: 56px;
  height: 56px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.stat-card.primary .stat-icon { background: rgba(102, 126, 234, 0.1); color: #667eea; }
.stat-card.secondary .stat-icon { background: rgba(59, 130, 246, 0.1); color: #3b82f6; }
.stat-card.success .stat-icon { background: rgba(16, 185, 129, 0.1); color: #10b981; }
.stat-card.warning .stat-icon { background: rgba(245, 158, 11, 0.1); color: #f59e0b; }

.stat-content {
  flex: 1;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #1f2937;
  line-height: 1;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
  margin-bottom: 6px;
}

.stat-change, .stat-breakdown {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
}

.change-icon {
  font-size: 12px;
}

.change-text {
  color: #6b7280;
}

.breakdown-item {
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
}

.breakdown-item.numeric {
  background: rgba(59, 130, 246, 0.1);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.breakdown-item.categorical {
  background: rgba(139, 92, 246, 0.1);
  color: #8b5cf6;
  border: 1px solid rgba(139, 92, 246, 0.2);
}

/* Analysis Section */
.analysis-section {
  margin-bottom: 32px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.section-filter {
  display: flex;
  align-items: center;
  gap: 12px;
}

.filter-label {
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
}

.filter-buttons {
  display: flex;
  gap: 4px;
  background: #f3f4f6;
  padding: 4px;
  border-radius: 8px;
}

.filter-btn {
  padding: 6px 12px;
  background: transparent;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  color: #6b7280;
  cursor: pointer;
  transition: all 0.3s ease;
}

.filter-btn:hover {
  background: #e5e7eb;
}

.filter-btn.active {
  background: white;
  color: #667eea;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.columns-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.column-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  transition: all 0.3s ease;
  position: relative;
}

.column-card:hover {
  border-color: #d1d5db;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  transform: translateY(-2px);
}

.column-card.numeric {
  border-left: 4px solid #3b82f6;
}

.column-card.categorical {
  border-left: 4px solid #8b5cf6;
}

.column-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 16px;
}

.column-info {
  flex: 1;
}

.column-name {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 4px 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.column-type {
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 10px;
  text-transform: uppercase;
  display: inline-block;
}

.column-type.numeric {
  background: rgba(59, 130, 246, 0.1);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.column-type.categorical {
  background: rgba(139, 92, 246, 0.1);
  color: #8b5cf6;
  border: 1px solid rgba(139, 92, 246, 0.2);
}

.column-index {
  font-size: 12px;
  color: #9ca3af;
  font-weight: 600;
  background: #f3f4f6;
  padding: 2px 8px;
  border-radius: 10px;
}

.column-stats {
  margin-bottom: 16px;
}

.stat-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 12px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.stat-label {
  font-size: 11px;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.stat-value {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

.stat-value.excellent { color: #10b981; }
.stat-value.good { color: #3b82f6; }
.stat-value.fair { color: #f59e0b; }
.stat-value.poor { color: #ef4444; }

.numeric-stats, .categorical-stats {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #f3f4f6;
}

.range-bar {
  background: #f9fafb;
  padding: 8px;
  border-radius: 6px;
}

.range-labels {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
  font-size: 11px;
  color: #6b7280;
}

.range-track {
  height: 4px;
  background: #e5e7eb;
  border-radius: 2px;
  overflow: hidden;
}

.range-progress {
  height: 100%;
  background: linear-gradient(90deg, #3b82f6, #1d4ed8);
  border-radius: 2px;
}

.top-values {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.value-tag {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  background: #f3f4f6;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  font-size: 11px;
  cursor: help;
}

.value-text {
  color: #374151;
  font-weight: 500;
  max-width: 80px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.value-percentage {
  color: #667eea;
  font-weight: 600;
}

.more-values {
  font-size: 11px;
  color: #9ca3af;
  padding: 4px 8px;
}

.column-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  padding-top: 12px;
  border-top: 1px solid #f3f4f6;
}

.empty-columns {
  text-align: center;
  padding: 40px 20px;
  background: #f9fafb;
  border: 2px dashed #e5e7eb;
  border-radius: 12px;
}

.empty-icon {
  font-size: 32px;
  margin-bottom: 12px;
  opacity: 0.3;
}

.empty-text {
  color: #6b7280;
  font-size: 14px;
  margin: 0;
}

/* Insights Section */
.insights-section {
  margin-bottom: 32px;
}

.insights-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.insight-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  gap: 16px;
  transition: all 0.3s ease;
}

.insight-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.insight-card.info { border-left: 4px solid #3b82f6; }
.insight-card.warning { border-left: 4px solid #f59e0b; }
.insight-card.error { border-left: 4px solid #ef4444; }
.insight-card.success { border-left: 4px solid #10b981; }

.insight-icon {
  font-size: 24px;
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.insight-card.info .insight-icon { background: rgba(59, 130, 246, 0.1); color: #3b82f6; }
.insight-card.warning .insight-icon { background: rgba(245, 158, 11, 0.1); color: #f59e0b; }
.insight-card.error .insight-icon { background: rgba(239, 68, 68, 0.1); color: #ef4444; }
.insight-card.success .insight-icon { background: rgba(16, 185, 129, 0.1); color: #10b981; }

.insight-content {
  flex: 1;
}

.insight-title {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
}

.insight-description {
  font-size: 14px;
  color: #6b7280;
  margin: 0 0 12px 0;
  line-height: 1.5;
}

.insight-actions {
  display: flex;
}

.insight-btn {
  padding: 6px 12px;
  background: #f3f4f6;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  color: #374151;
  cursor: pointer;
  transition: all 0.3s ease;
}

.insight-btn:hover {
  background: #e5e7eb;
}

/* Summary Section */
.summary-section {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
}

.summary-content {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 32px;
}

.summary-stats {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 12px;
  border-bottom: 1px solid #e5e7eb;
}

.summary-label {
  font-size: 14px;
  color: #6b7280;
  font-weight: 500;
}

.summary-value {
  font-size: 15px;
  font-weight: 600;
  color: #1f2937;
}

.summary-notes {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
}

.notes-title {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 12px 0;
}

.notes-list {
  margin: 0;
  padding-left: 20px;
}

.notes-list li {
  font-size: 14px;
  color: #6b7280;
  margin-bottom: 8px;
  line-height: 1.5;
}

.notes-list li:last-child {
  margin-bottom: 0;
}

/* Empty State */
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

  .header-actions {
    align-self: stretch;
    justify-content: flex-end;
  }

  .summary-content {
    grid-template-columns: 1fr;
    gap: 24px;
  }

  .columns-grid {
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
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

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .section-filter {
    width: 100%;
    justify-content: space-between;
  }

  .insights-grid {
    grid-template-columns: 1fr;
  }

  .empty-state {
    padding: 40px 24px;
  }
}

@media (max-width: 480px) {
  .card-header {
    padding: 16px 20px;
  }

  .card-body {
    padding: 20px;
  }

  .column-card {
    padding: 16px;
  }

  .insight-card {
    padding: 16px;
  }

  .summary-section {
    padding: 20px;
  }

  .empty-state {
    padding: 32px 20px;
  }

  .empty-icon {
    font-size: 48px;
  }
}
</style>