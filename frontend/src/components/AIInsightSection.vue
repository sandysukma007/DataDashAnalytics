<template>
  <section class="card">
    <div class="card-header">
      <div class="header-content">
        <h3 class="card-title">
          <span class="title-icon">🧠</span>
          AI Insights
        </h3>
        <p class="card-subtitle">Intelligent analysis from your data patterns</p>
      </div>

      <div class="header-actions" v-if="insights.length">
        <button class="action-btn" @click="refreshInsights" :disabled="isGenerating">
          <span class="btn-icon">🔄</span>
          {{ isGenerating ? 'Generating...' : 'Refresh' }}
        </button>
      </div>
    </div>

    <div class="card-body">
      <!-- Configuration Section -->
      <div v-if="showConfiguration && columns && columns.length" class="configuration-section">
        <h4 class="section-title">
          <span class="section-icon">⚙️</span>
          Configure AI Analysis
        </h4>

        <div class="config-form">
          <div class="form-group">
            <label class="form-label">
              <span class="label-icon">📍</span>
              Select X Column
              <span class="label-hint">(Category or Numeric)</span>
            </label>
            <select v-model="config.xColumn" class="form-select">
              <option value="">Choose X column</option>
              <option v-for="col in columns" :key="col" :value="col">{{ col }}</option>
            </select>
          </div>

          <div class="form-group">
            <label class="form-label">
              <span class="label-icon">📈</span>
              Select Y Column
              <span class="label-hint">(Numeric values for analysis)</span>
            </label>
            <select v-model="config.yColumn" class="form-select">
              <option value="">Choose Y column</option>
              <option v-for="col in availableNumericColumns" :key="col" :value="col">{{ col }}</option>
            </select>
          </div>

          <div class="form-actions">
            <button
              class="form-btn primary"
              @click="generateInsights"
              :disabled="!config.xColumn || !config.yColumn || isGenerating || !hasValidData"
            >
              <span class="btn-icon">✨</span>
              Generate AI Insights
            </button>
            <button class="form-btn" @click="showConfiguration = false" v-if="insights.length">
              Cancel
            </button>
          </div>

          <div v-if="!hasValidData" class="data-warning">
            <span class="warning-icon">⚠️</span>
            <p>Selected columns don't have enough numeric data for analysis.</p>
          </div>
        </div>
      </div>

      <!-- Generated Insights -->
      <div v-if="insights.length && !showConfiguration">
        <!-- Insights Overview -->
        <div class="insights-overview">
          <div class="overview-content">
            <h4 class="overview-title">
              <span class="overview-icon">📊</span>
              Analysis Results
              <span class="overview-subtitle">for {{ config.yColumn }} vs {{ config.xColumn }}</span>
            </h4>

            <div class="analysis-meta">
              <span class="meta-item">
                <span class="meta-icon">🕒</span>
                Generated just now
              </span>
              <span class="meta-item">
                <span class="meta-icon">🔍</span>
                {{ analysisType }} analysis
              </span>
              <span class="meta-item">
                <span class="meta-icon">📊</span>
                {{ filteredRows.length }} data points
              </span>
            </div>
          </div>

          <button class="action-btn" @click="showConfiguration = true">
            <span class="btn-icon">⚙️</span>
            Change Parameters
          </button>
        </div>

        <!-- Insights Grid -->
        <div class="insights-grid">
          <div
            v-for="(insight, index) in insights"
            :key="index"
            class="insight-card"
            :class="getInsightType(insight)"
          >
            <div class="insight-header">
              <div class="insight-icon">{{ getInsightIcon(index) }}</div>
              <div class="insight-title-section">
                <h4 class="insight-title">{{ formatInsightTitle(insight, index) }}</h4>
                <div class="insight-meta">
                  <span class="meta-item">
                    <span class="meta-icon">#</span>
                    Insight {{ index + 1 }}
                  </span>
                  <span class="meta-item">
                    <span class="meta-icon">🎯</span>
                    {{ getInsightCategory(insight) }}
                  </span>
                </div>
              </div>
              <span class="insight-badge" :class="getInsightType(insight)">
                {{ getInsightType(insight).toUpperCase() }}
              </span>
            </div>

            <div class="insight-content">
              <p class="insight-text">{{ insight }}</p>

              <!-- Extract numbers from insight -->
              <div v-if="extractNumbers(insight).length > 0" class="insight-metrics">
                <div
                  v-for="(number, idx) in extractNumbers(insight)"
                  :key="idx"
                  class="metric-highlight"
                >
                  <span class="metric-label">{{ getNumberLabel(number, insight) }}</span>
                  <span class="metric-value">{{ formatNumber(number) }}</span>
                </div>
              </div>

              <!-- Highlight keywords -->
              <div class="insight-keywords">
                <span
                  v-for="keyword in extractKeywords(insight)"
                  :key="keyword"
                  class="keyword-tag"
                  @click="focusOnKeyword(keyword)"
                >
                  {{ keyword }}
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- Statistical Summary -->
        <div class="statistical-summary">
          <h4 class="section-title">
            <span class="section-icon">📈</span>
            Statistical Summary
          </h4>

          <div class="stats-grid">
            <div class="stat-card">
              <div class="stat-icon">📊</div>
              <div class="stat-content">
                <div class="stat-value">{{ calculateAverage() }}</div>
                <div class="stat-label">Average Value</div>
                <div class="stat-hint">Mean of {{ config.yColumn }}</div>
              </div>
            </div>

            <div class="stat-card">
              <div class="stat-icon">⬆️</div>
              <div class="stat-content">
                <div class="stat-value">{{ calculateMaxValue() }}</div>
                <div class="stat-label">Maximum Value</div>
                <div class="stat-hint">Peak performance</div>
              </div>
            </div>

            <div class="stat-card">
              <div class="stat-icon">⬇️</div>
              <div class="stat-content">
                <div class="stat-value">{{ calculateMinValue() }}</div>
                <div class="stat-label">Minimum Value</div>
                <div class="stat-hint">Lowest point</div>
              </div>
            </div>

            <div class="stat-card">
              <div class="stat-icon">📉</div>
              <div class="stat-content">
                <div class="stat-value">{{ detectTrend() }}</div>
                <div class="stat-label">Overall Trend</div>
                <div class="stat-hint">Direction analysis</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Recommendations -->
        <div class="recommendations-section" v-if="recommendations.length > 0">
          <h4 class="section-title">
            <span class="section-icon">🎯</span>
            Actionable Recommendations
          </h4>

          <div class="recommendations-grid">
            <div class="recommendation-card" v-for="(rec, index) in recommendations" :key="index">
              <div class="rec-icon">{{ getRecIcon(index) }}</div>
              <div class="rec-content">
                <h5 class="rec-title">{{ rec.title }}</h5>
                <p class="rec-description">{{ rec.description }}</p>
                <div class="rec-actions">
                  <button class="rec-action-btn" @click="applyRecommendation(rec)">
                    Apply
                  </button>
                  <button class="rec-action-btn outline" @click="learnMore(rec)">
                    Learn More
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- No Insights State -->
      <div v-else-if="!showConfiguration" class="empty-state">
        <div class="empty-content">
          <div class="empty-icon">🧠</div>
          <h4 class="empty-title">Generate AI Insights</h4>
          <p class="empty-description">
            Use AI to analyze relationships between columns and discover hidden patterns in your data
          </p>

          <div class="empty-features">
            <div class="feature-item">
              <span class="feature-icon">📊</span>
              <span class="feature-text">Statistical analysis and trends</span>
            </div>
            <div class="feature-item">
              <span class="feature-icon">🔍</span>
              <span class="feature-text">Pattern recognition and insights</span>
            </div>
            <div class="feature-item">
              <span class="feature-icon">🎯</span>
              <span class="feature-text">Actionable recommendations</span>
            </div>
          </div>

          <button
            class="empty-action-btn primary"
            @click="showConfiguration = true"
            :disabled="!columns || columns.length === 0"
          >
            <span class="btn-icon">✨</span>
            {{ columns && columns.length > 0 ? 'Start AI Analysis' : 'No Data Available' }}
          </button>

          <div v-if="!columns || columns.length === 0" class="no-data-warning">
            <p>Please upload data first to generate insights.</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// Define props dengan default values
const props = defineProps({
  columns: {
    type: Array,
    default: () => []
  },
  rows: {
    type: Array,
    default: () => []
  },
  numericColumns: {
    type: Array,
    default: () => []
  },
  insights: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['generate-insights'])

// State
const localInsights = ref([])
const isGenerating = ref(false)
const showConfiguration = ref(false)
const analysisType = ref('')
const recommendations = ref([])

// Configuration
const config = ref({
  xColumn: '',
  yColumn: ''
})

// Computed properties dengan safe access
const availableNumericColumns = computed(() => {
  return props.numericColumns || []
})

const columns = computed(() => {
  return props.columns || []
})

const rows = computed(() => {
  return props.rows || []
})

const filteredRows = computed(() => {
  if (!config.value.xColumn || !config.value.yColumn || !rows.value.length) return []

  return rows.value.filter(row => {
    const xVal = row[config.value.xColumn]
    const yVal = row[config.value.yColumn]

    return xVal !== undefined &&
           xVal !== null &&
           xVal !== '' &&
           yVal !== undefined &&
           yVal !== null &&
           yVal !== '' &&
           !isNaN(parseFloat(yVal))
  })
})

const hasValidData = computed(() => {
  return filteredRows.value.length >= 2 // Minimal 2 data points untuk analysis
})

const yValues = computed(() => {
  return filteredRows.value.map(row => parseFloat(row[config.value.yColumn]) || 0)
})

const xValues = computed(() => {
  return filteredRows.value.map(row => row[config.value.xColumn])
})

// Methods
const generateInsights = async () => {
  if (!config.value.xColumn || !config.value.yColumn || !hasValidData.value) {
    alert('Please select valid columns with at least 2 numeric data points')
    return
  }

  isGenerating.value = true

  try {
    // Prepare data for backend
    const payload = {
      x: xValues.value,
      y: yValues.value,
      x_name: config.value.xColumn,
      y_name: config.value.yColumn
    }

    // Call parent to generate insights (will trigger API call)
    emit('generate-insights', payload)

    // For now, generate local insights as fallback
    generateLocalInsights()

  } catch (error) {
    console.error('Error generating insights:', error)
    generateLocalInsights() // Fallback to local insights
  } finally {
    isGenerating.value = false
    showConfiguration.value = false
  }
}

const generateLocalInsights = () => {
  if (yValues.value.length === 0) {
    localInsights.value = []
    return
  }

  const avg = yValues.value.reduce((a, b) => a + b, 0) / yValues.value.length
  const maxVal = Math.max(...yValues.value)
  const minVal = Math.min(...yValues.value)
  const maxIndex = yValues.value.indexOf(maxVal)

  // Simple trend detection
  let trend = "stabil"
  if (yValues.value.length > 1) {
    const firstHalf = yValues.value.slice(0, Math.floor(yValues.value.length / 2))
    const secondHalf = yValues.value.slice(Math.floor(yValues.value.length / 2))
    const avgFirst = firstHalf.reduce((a, b) => a + b, 0) / firstHalf.length
    const avgSecond = secondHalf.reduce((a, b) => a + b, 0) / secondHalf.length

    if (avgSecond > avgFirst * 1.1) trend = "meningkat"
    else if (avgSecond < avgFirst * 0.9) trend = "menurun"
  }

  localInsights.value = [
    `Rata-rata ${config.value.yColumn} adalah ${avg.toFixed(2)}`,
    `Nilai tertinggi ${config.value.yColumn} sebesar ${maxVal.toFixed(2)} pada ${config.value.xColumn} = ${xValues.value[maxIndex]}`,
    `Tren ${config.value.yColumn} cenderung ${trend}`,
    `Rentang nilai berada antara ${minVal.toFixed(2)} hingga ${maxVal.toFixed(2)}`
  ]

  analysisType.value = 'Statistical'
  generateRecommendations()
}

const refreshInsights = () => {
  if (config.value.xColumn && config.value.yColumn) {
    generateInsights()
  } else {
    showConfiguration.value = true
  }
}

const getInsightType = (insight) => {
  if (!insight) return 'insight'

  const text = insight.toLowerCase()
  if (text.includes('rata-rata') || text.includes('average')) return 'statistic'
  if (text.includes('tertinggi') || text.includes('maximum') || text.includes('peak')) return 'highlight'
  if (text.includes('tren') || text.includes('trend') || text.includes('cenderung')) return 'trend'
  if (text.includes('rentang') || text.includes('range') || text.includes('antara')) return 'range'
  return 'insight'
}

const getInsightIcon = (index) => {
  const icons = ['📊', '⬆️', '📈', '📏']
  return icons[index % icons.length]
}

const formatInsightTitle = (insight, index) => {
  const titles = [
    'Average Analysis',
    'Peak Performance',
    'Trend Detection',
    'Value Range'
  ]

  const defaultTitle = titles[index] || 'Data Insight'

  if (!insight) return defaultTitle

  // Extract key phrase from insight
  if (insight.includes('Rata-rata')) return 'Average Value Analysis'
  if (insight.includes('Nilai tertinggi')) return 'Maximum Value Found'
  if (insight.includes('Tren')) return 'Trend Analysis'
  if (insight.includes('Rentang')) return 'Value Range Summary'

  return defaultTitle
}

const getInsightCategory = (insight) => {
  const type = getInsightType(insight)
  const categories = {
    statistic: 'Statistical',
    highlight: 'Highlight',
    trend: 'Trend',
    range: 'Range',
    insight: 'General'
  }
  return categories[type] || 'Analysis'
}

const extractNumbers = (insight) => {
  if (!insight) return []

  const numbers = insight.match(/\d+\.?\d*/g)
  return numbers ? numbers.map(n => parseFloat(n)).filter(n => !isNaN(n)) : []
}

const extractKeywords = (insight) => {
  if (!insight) return []

  const commonWords = new Set(['adalah', 'pada', 'sebesar', 'cenderung', 'antara', 'hingga', 'nilai', 'rata'])
  const words = insight.toLowerCase()
    .replace(/[^\w\s]/g, ' ')
    .split(/\s+/)
    .filter(word =>
      word.length > 3 &&
      !commonWords.has(word) &&
      !/\d/.test(word)
    )

  return [...new Set(words)].slice(0, 3)
}

const getNumberLabel = (number, insight) => {
  if (!insight) return 'Value'

  if (insight.includes('rata-rata') || insight.includes('average')) return 'Average'
  if (insight.includes('tertinggi') || insight.includes('maximum')) return 'Maximum'
  if (insight.includes('terendah') || insight.includes('minimum')) return 'Minimum'
  if (insight.includes('rentang') || insight.includes('range')) return 'Range Value'
  return 'Value'
}

const formatNumber = (number) => {
  if (Number.isInteger(number)) {
    return number.toLocaleString()
  } else {
    return number.toFixed(2)
  }
}

const focusOnKeyword = (keyword) => {
  console.log('Focusing on keyword:', keyword)
  alert(`Focusing analysis on: ${keyword}`)
}

// Statistical calculations
const calculateAverage = () => {
  if (yValues.value.length === 0) return 'N/A'

  const avg = yValues.value.reduce((a, b) => a + b, 0) / yValues.value.length
  return avg.toFixed(2)
}

const calculateMaxValue = () => {
  if (yValues.value.length === 0) return 'N/A'
  return Math.max(...yValues.value).toFixed(2)
}

const calculateMinValue = () => {
  if (yValues.value.length === 0) return 'N/A'
  return Math.min(...yValues.value).toFixed(2)
}

const detectTrend = () => {
  if (yValues.value.length < 2) return 'Insufficient Data'

  // Simple trend detection
  const firstHalf = yValues.value.slice(0, Math.floor(yValues.value.length / 2))
  const secondHalf = yValues.value.slice(Math.floor(yValues.value.length / 2))
  const avgFirst = firstHalf.reduce((a, b) => a + b, 0) / firstHalf.length
  const avgSecond = secondHalf.reduce((a, b) => a + b, 0) / secondHalf.length

  if (avgSecond > avgFirst * 1.1) return 'Increasing ↗️'
  if (avgSecond < avgFirst * 0.9) return 'Decreasing ↘️'
  return 'Stable →'
}

const generateRecommendations = () => {
  const recs = []

  if (yValues.value.length === 0) {
    recommendations.value = []
    return
  }

  const avg = yValues.value.reduce((a, b) => a + b, 0) / yValues.value.length
  const maxVal = Math.max(...yValues.value)
  const minVal = Math.min(...yValues.value)

  // Recommendation based on data spread
  const spread = maxVal - minVal
  if (spread > avg * 2) {
    recs.push({
      title: 'High Variability Detected',
      description: `Large spread (${spread.toFixed(2)}) suggests inconsistent performance. Consider standardizing processes.`
    })
  } else if (spread < avg * 0.5) {
    recs.push({
      title: 'Consistent Performance',
      description: 'Low variability indicates stable operations. Focus on incremental improvements.'
    })
  }

  // Recommendation based on trend
  const trend = detectTrend()
  if (trend.includes('Increasing')) {
    recs.push({
      title: 'Growth Opportunity',
      description: 'Positive trend detected. Consider scaling successful strategies.'
    })
  } else if (trend.includes('Decreasing')) {
    recs.push({
      title: 'Attention Required',
      description: 'Negative trend detected. Investigate causes and implement corrective actions.'
    })
  }

  // General recommendations
  recs.push({
    title: 'Data Monitoring',
    description: `Monitor ${config.value.yColumn} regularly. Set target range: ${minVal.toFixed(2)} - ${maxVal.toFixed(2)}`
  })

  recommendations.value = recs.slice(0, 3)
}

const getRecIcon = (index) => {
  const icons = ['🚀', '🎯', '📊', '💡', '🔧', '⭐']
  return icons[index % icons.length]
}

const applyRecommendation = (recommendation) => {
  console.log('Applying recommendation:', recommendation.title)
  alert(`Applying: ${recommendation.title}`)
}

const learnMore = (recommendation) => {
  console.log('Learning more about:', recommendation.title)
  alert(`${recommendation.title}\n\n${recommendation.description}`)
}

// Initialize with default values when data is available
watch(() => columns.value, (newColumns) => {
  if (newColumns && newColumns.length > 0 && !config.value.xColumn) {
    config.value.xColumn = newColumns[0]
  }
}, { immediate: true })

watch(() => availableNumericColumns.value, (newNumericCols) => {
  if (newNumericCols && newNumericCols.length > 0 && !config.value.yColumn) {
    config.value.yColumn = newNumericCols[0]
  }
}, { immediate: true })

// Watch for insights from parent (from backend)
watch(() => props.insights, (newInsights) => {
  if (newInsights && newInsights.length > 0) {
    localInsights.value = newInsights
    showConfiguration.value = false
    generateRecommendations()
  }
}, { immediate: true })

// Get the insights to display (either from props or local)
const insights = computed(() => {
  return localInsights.value.length > 0 ? localInsights.value : []
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

.action-btn:hover:not(:disabled) {
  background: #f3f4f6;
  border-color: #d1d5db;
  transform: translateY(-1px);
}

.action-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-icon {
  font-size: 14px;
}

/* Card Body */
.card-body {
  padding: 32px;
}

/* Configuration Section */
.configuration-section {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
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

.config-form {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-label {
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
  font-size: 13px;
  color: #6b7280;
  font-weight: 400;
  margin-left: auto;
}

.form-select {
  padding: 12px 16px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  color: #374151;
  background: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.form-select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-actions {
  grid-column: 1 / -1;
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding-top: 16px;
  border-top: 1px solid #e5e7eb;
}

.form-btn {
  padding: 12px 24px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  color: #374151;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.form-btn:hover:not(:disabled) {
  background: #f3f4f6;
  border-color: #d1d5db;
  transform: translateY(-1px);
}

.form-btn.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
}

.form-btn.primary:hover:not(:disabled) {
  background: linear-gradient(135deg, #5a6fd9 0%, #6a4199 100%);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.form-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.data-warning {
  grid-column: 1 / -1;
  background: #fef3c7;
  border: 1px solid #fbbf24;
  border-radius: 8px;
  padding: 12px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 8px;
}

.warning-icon {
  color: #d97706;
  font-size: 18px;
}

.data-warning p {
  margin: 0;
  color: #92400e;
  font-size: 14px;
}

/* Insights Overview */
.insights-overview {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.overview-content {
  flex: 1;
}

.overview-title {
  font-size: 20px;
  font-weight: 700;
  color: #1f2937;
  margin: 0 0 8px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.overview-icon {
  color: #667eea;
}

.overview-subtitle {
  font-size: 16px;
  color: #6b7280;
  font-weight: 500;
  margin-left: 12px;
}

.analysis-meta {
  display: flex;
  gap: 16px;
  align-items: center;
  flex-wrap: wrap;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 13px;
  color: #6b7280;
  background: white;
  padding: 4px 12px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
}

.meta-icon {
  font-size: 12px;
}

/* Insights Grid */
.insights-grid {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 32px;
}

.insight-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  transition: all 0.3s ease;
}

.insight-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.insight-card.statistic { border-left: 4px solid #3b82f6; }
.insight-card.highlight { border-left: 4px solid #10b981; }
.insight-card.trend { border-left: 4px solid #f59e0b; }
.insight-card.range { border-left: 4px solid #8b5cf6; }
.insight-card.insight { border-left: 4px solid #ec4899; }

.insight-header {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 16px;
}

.insight-icon {
  font-size: 24px;
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.insight-card.statistic .insight-icon { background: rgba(59, 130, 246, 0.1); color: #3b82f6; }
.insight-card.highlight .insight-icon { background: rgba(16, 185, 129, 0.1); color: #10b981; }
.insight-card.trend .insight-icon { background: rgba(245, 158, 11, 0.1); color: #f59e0b; }
.insight-card.range .insight-icon { background: rgba(139, 92, 246, 0.1); color: #8b5cf6; }
.insight-card.insight .insight-icon { background: rgba(236, 72, 153, 0.1); color: #ec4899; }

.insight-title-section {
  flex: 1;
}

.insight-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
  line-height: 1.4;
}

.insight-meta {
  display: flex;
  gap: 16px;
  align-items: center;
}

.insight-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  flex-shrink: 0;
}

.insight-card.statistic .insight-badge { background: rgba(59, 130, 246, 0.1); color: #3b82f6; border: 1px solid rgba(59, 130, 246, 0.2); }
.insight-card.highlight .insight-badge { background: rgba(16, 185, 129, 0.1); color: #10b981; border: 1px solid rgba(16, 185, 129, 0.2); }
.insight-card.trend .insight-badge { background: rgba(245, 158, 11, 0.1); color: #f59e0b; border: 1px solid rgba(245, 158, 11, 0.2); }
.insight-card.range .insight-badge { background: rgba(139, 92, 246, 0.1); color: #8b5cf6; border: 1px solid rgba(139, 92, 246, 0.2); }
.insight-card.insight .insight-badge { background: rgba(236, 72, 153, 0.1); color: #ec4899; border: 1px solid rgba(236, 72, 153, 0.2); }

.insight-content {
  margin-bottom: 20px;
}

.insight-text {
  font-size: 15px;
  color: #4b5563;
  line-height: 1.6;
  margin: 0 0 16px 0;
}

.insight-metrics {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.metric-highlight {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  min-width: 120px;
}

.metric-label {
  font-size: 12px;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.metric-value {
  font-size: 18px;
  font-weight: 700;
  color: #1f2937;
}

.insight-keywords {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.keyword-tag {
  display: inline-flex;
  align-items: center;
  padding: 6px 12px;
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #e5e7eb;
  border-radius: 16px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.keyword-tag:hover {
  background: #e5e7eb;
  transform: translateY(-1px);
}

/* Statistical Summary */
.statistical-summary {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 32px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.stat-card {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  transition: all 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.stat-icon {
  font-size: 24px;
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
}

.stat-content {
  flex: 1;
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
  line-height: 1;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: #6b7280;
  font-weight: 600;
  margin-bottom: 2px;
}

.stat-hint {
  font-size: 12px;
  color: #9ca3af;
}

/* Recommendations Section */
.recommendations-section {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
}

.recommendations-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.recommendation-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  gap: 16px;
  transition: all 0.3s ease;
}

.recommendation-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  border-color: #667eea;
}

.rec-icon {
  font-size: 24px;
  width: 48px;
  height: 48px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.rec-content {
  flex: 1;
}

.rec-title {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 8px 0;
}

.rec-description {
  font-size: 14px;
  color: #6b7280;
  line-height: 1.5;
  margin: 0 0 12px 0;
}

.rec-actions {
  display: flex;
  gap: 8px;
}

.rec-action-btn {
  padding: 6px 12px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border: 1px solid rgba(102, 126, 234, 0.2);
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.rec-action-btn:hover {
  background: rgba(102, 126, 234, 0.2);
}

.rec-action-btn.outline {
  background: transparent;
  color: #6b7280;
  border-color: #e5e7eb;
}

.rec-action-btn.outline:hover {
  background: #f3f4f6;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 60px 40px;
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border-radius: 12px;
  border: 2px dashed #d1d5db;
}

.empty-content {
  max-width: 500px;
  margin: 0 auto;
}

.empty-icon {
  font-size: 80px;
  margin-bottom: 24px;
  opacity: 0.3;
  display: block;
}

.empty-title {
  font-size: 24px;
  font-weight: 700;
  color: #374151;
  margin-bottom: 12px;
}

.empty-description {
  font-size: 16px;
  color: #6b7280;
  line-height: 1.6;
  margin-bottom: 32px;
}

.empty-features {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 32px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}

.feature-icon {
  font-size: 18px;
  color: #667eea;
}

.feature-text {
  font-size: 14px;
  color: #4b5563;
  font-weight: 500;
}

.empty-action-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  color: #374151;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.empty-action-btn:hover:not(:disabled) {
  background: #f3f4f6;
  border-color: #d1d5db;
  transform: translateY(-2px);
}

.empty-action-btn.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
}

.empty-action-btn.primary:hover:not(:disabled) {
  background: linear-gradient(135deg, #5a6fd9 0%, #6a4199 100%);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.empty-action-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.no-data-warning {
  margin-top: 16px;
  padding: 12px;
  background: #fef3c7;
  border: 1px solid #fbbf24;
  border-radius: 8px;
  color: #92400e;
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

  .insights-overview {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .config-form {
    grid-template-columns: 1fr;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .recommendations-grid {
    grid-template-columns: 1fr;
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

  .overview-title {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .overview-subtitle {
    margin-left: 0;
    font-size: 14px;
  }

  .analysis-meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .insight-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .insight-badge {
    align-self: flex-start;
  }

  .stats-grid {
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

  .insight-card {
    padding: 16px;
  }

  .insight-metrics {
    flex-direction: column;
  }

  .metric-highlight {
    width: 100%;
  }

  .empty-state {
    padding: 32px 20px;
  }

  .empty-icon {
    font-size: 64px;
  }

  .empty-title {
    font-size: 20px;
  }
}
</style>