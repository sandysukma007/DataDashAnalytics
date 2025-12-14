<template>
  <section class="card">
    <div class="card-header">
      <div class="header-content">
        <div class="title-section">
          <h3 class="card-title">
            <span class="title-icon">📋</span>
            Dataset Preview
          </h3>
          <p class="card-subtitle">Explore and analyze your uploaded data</p>
        </div>

        <div class="stats-section" v-if="columns.length">
          <div class="stats-group">
            <div class="stat-item">
              <span class="stat-icon">📊</span>
              <div class="stat-content">
                <div class="stat-value">{{ rows.length }}</div>
                <div class="stat-label">Rows</div>
              </div>
            </div>
            <div class="stat-item">
              <span class="stat-icon">📋</span>
              <div class="stat-content">
                <div class="stat-value">{{ columns.length }}</div>
                <div class="stat-label">Columns</div>
              </div>
            </div>
            <div class="stat-item">
              <span class="stat-icon">🔢</span>
              <div class="stat-content">
                <div class="stat-value">{{ filteredRows.length }}</div>
                <div class="stat-label">Filtered</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="card-body">
      <!-- Filters Section -->
      <div v-if="columns.length && columns.length > 0" class="filters-section">
        <div class="filters-header">
          <h4 class="filters-title">
            <span class="filter-icon">🔍</span>
            Data Filters
          </h4>
          <div class="filters-actions">
            <span class="active-filters">
              {{ activeFilterCount }} active filter{{ activeFilterCount !== 1 ? 's' : '' }}
            </span>
            <button
              class="clear-filters-btn"
              @click="clearFilters"
              :disabled="activeFilterCount === 0"
            >
              <span class="btn-icon">🗑️</span>
              Clear All
            </button>
          </div>
        </div>

        <div class="filters-grid">
          <div
            v-for="col in columns.slice(0, 4)"
            :key="col"
            class="filter-item"
          >
            <label class="filter-label">
              <span class="filter-column">{{ col }}</span>
              <span class="filter-type" :class="getColumnTypeClass(col)">
                {{ categoricalColumns.includes(col) ? 'Category' : 'Number' }}
              </span>
            </label>

            <div class="filter-control">
              <select
                v-if="categoricalColumns.includes(col)"
                v-model="filters[col]"
                class="filter-select"
              >
                <option value="">All values</option>
                <option
                  v-for="val in uniqueValues(col)"
                  :key="val"
                  :value="val"
                >
                  {{ val }}
                </option>
              </select>

              <input
                v-else
                type="number"
                v-model.number="filters[col]"
                :placeholder="`Filter ${col}`"
                class="filter-input"
              />

              <div class="filter-status" v-if="filters[col]">
                <span class="status-indicator active"></span>
                <span class="status-text">Active</span>
              </div>
            </div>
          </div>

          <!-- Show more columns button -->
          <div class="filter-item more-columns" v-if="columns.length > 4">
            <div class="more-columns-content">
              <span class="more-icon">➕</span>
              <p class="more-text">
                {{ columns.length - 4 }} more column{{ columns.length - 4 > 1 ? 's' : '' }} available
              </p>
              <button class="show-more-btn" @click="showAllFilters = !showAllFilters">
                {{ showAllFilters ? 'Show Less' : 'Show All' }}
              </button>
            </div>
          </div>

          <!-- Additional columns (hidden by default) -->
          <div
            v-if="showAllFilters"
            class="additional-filters"
          >
            <div
              v-for="col in columns.slice(4)"
              :key="col"
              class="filter-item"
            >
              <label class="filter-label">
                <span class="filter-column">{{ col }}</span>
                <span class="filter-type" :class="getColumnTypeClass(col)">
                  {{ categoricalColumns.includes(col) ? 'Category' : 'Number' }}
                </span>
              </label>

              <div class="filter-control">
                <select
                  v-if="categoricalColumns.includes(col)"
                  v-model="filters[col]"
                  class="filter-select"
                >
                  <option value="">All values</option>
                  <option
                    v-for="val in uniqueValues(col)"
                    :key="val"
                    :value="val"
                  >
                    {{ val }}
                  </option>
                </select>

                <input
                  v-else
                  type="number"
                  v-model.number="filters[col]"
                  :placeholder="`Filter ${col}`"
                  class="filter-input"
                />
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Data Table -->
      <div v-if="columns.length" class="table-section">
        <div class="table-container">
          <div class="table-header-row">
            <div class="table-info">
              <span class="table-count">
                Showing {{ filteredRows.length }} of {{ rows.length }} rows
              </span>
              <span class="table-scroll-hint">
                <span class="hint-icon">↔️</span>
                Scroll horizontally to view all columns
              </span>
            </div>
            <div class="table-actions">
              <button class="table-action-btn" title="Download CSV">
                <span class="action-icon">📥</span>
                Export
              </button>
              <button class="table-action-btn" title="Copy to clipboard">
                <span class="action-icon">📋</span>
                Copy
              </button>
            </div>
          </div>

          <div class="table-wrapper">
            <table class="data-table">
              <thead>
                <tr>
                  <th
                    v-for="(col, index) in columns"
                    :key="col"
                    class="table-header"
                    :class="{ 'first-column': index === 0 }"
                  >
                    <div class="header-content">
                      <span class="column-number">#{{ index + 1 }}</span>
                      <span class="column-name">{{ col }}</span>
                      <span class="column-type" :class="getColumnTypeClass(col)">
                        {{ categoricalColumns.includes(col) ? 'CAT' : 'NUM' }}
                      </span>
                    </div>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(row, i) in filteredRows.slice(0, 100)"
                  :key="i"
                  :class="{ 'alternate-row': i % 2 === 0 }"
                >
                  <td
                    v-for="col in columns"
                    :key="col"
                    class="table-cell"
                    :class="{ 'first-column': columns.indexOf(col) === 0 }"
                    :title="formatCellValue(row[col])"
                  >
                    {{ formatCellValue(row[col]) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Pagination/Row Info -->
          <div class="table-footer">
            <div class="row-info">
              <span class="row-count">
                Displaying first {{ Math.min(filteredRows.length, 100) }} rows
                <span v-if="filteredRows.length > 100" class="row-more">
                  ({{ filteredRows.length - 100 }} more rows available)
                </span>
              </span>
            </div>
            <div class="scroll-hint">
              <span class="scroll-text">← Scroll → to view all columns</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else class="empty-state">
        <div class="empty-icon">📊</div>
        <h4 class="empty-title">No Data Available</h4>
        <p class="empty-description">Upload a dataset to start exploring your data</p>
        <button class="empty-action" @click="$emit('navigate-upload')">
          <span class="action-icon">📤</span>
          Upload Data
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps([
  'columns',
  'rows',
  'categoricalColumns',
  'filters',
  'filteredRows',
  'uniqueValues',
  'clearFilters'
])

const emit = defineEmits(['navigate-upload'])

const showAllFilters = ref(false)

// Computed properties
const activeFilterCount = computed(() => {
  return Object.values(props.filters).filter(val =>
    val !== '' && val !== null && val !== undefined
  ).length
})

// Methods
const getColumnTypeClass = (col) => {
  return props.categoricalColumns.includes(col) ? 'type-category' : 'type-number'
}

const formatCellValue = (value) => {
  if (value === null || value === undefined || value === '') {
    return '-'
  }

  // Format numbers with thousand separators
  if (typeof value === 'number' || !isNaN(parseFloat(value))) {
    const num = parseFloat(value)
    if (Number.isInteger(num)) {
      return num.toLocaleString()
    } else {
      return num.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      })
    }
  }

  // Return string value (truncate if too long)
  const str = String(value)
  return str.length > 50 ? str.substring(0, 50) + '...' : str
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
  padding: 24px 32px;
  background: linear-gradient(135deg, #667eea15 0%, #764ba215 100%);
  border-bottom: 1px solid #e5e7eb;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 24px;
}

.title-section {
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

/* Stats Section */
.stats-section {
  flex-shrink: 0;
}

.stats-group {
  display: flex;
  gap: 20px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 20px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  min-width: 120px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border-color: #667eea;
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

/* Filters Section */
.filters-section {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 32px;
}

.filters-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 16px;
  border-bottom: 1px solid #e5e7eb;
}

.filters-title {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.filter-icon {
  color: #667eea;
}

.active-filters {
  font-size: 14px;
  color: #6b7280;
  background: rgba(102, 126, 234, 0.1);
  padding: 6px 12px;
  border-radius: 20px;
  border: 1px solid rgba(102, 126, 234, 0.2);
}

.clear-filters-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: white;
  border: 1px solid #dc2626;
  color: #dc2626;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.clear-filters-btn:hover:not(:disabled) {
  background: #dc2626;
  color: white;
  transform: translateY(-1px);
}

.clear-filters-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  border-color: #d1d5db;
  color: #9ca3af;
}

.btn-icon {
  font-size: 16px;
}

.filters-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.filter-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.filter-label {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.filter-column {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.filter-type {
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 12px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.filter-type.type-category {
  background: rgba(139, 92, 246, 0.1);
  color: #8b5cf6;
  border: 1px solid rgba(139, 92, 246, 0.2);
}

.filter-type.type-number {
  background: rgba(59, 130, 246, 0.1);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.filter-control {
  position: relative;
}

.filter-select, .filter-input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  color: #374151;
  background: white;
  transition: all 0.3s ease;
}

.filter-select:focus, .filter-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.filter-input::placeholder {
  color: #9ca3af;
}

.filter-status {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  gap: 4px;
}

.status-indicator {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.status-indicator.active {
  background: #10b981;
  animation: pulse 2s infinite;
}

.status-text {
  font-size: 11px;
  color: #10b981;
  font-weight: 600;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* More Columns */
.more-columns {
  background: white;
  border: 2px dashed #d1d5db;
  border-radius: 10px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.more-columns:hover {
  border-color: #667eea;
  background: rgba(102, 126, 234, 0.02);
}

.more-columns-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  text-align: center;
}

.more-icon {
  font-size: 24px;
  color: #667eea;
}

.more-text {
  font-size: 14px;
  color: #6b7280;
  margin: 0;
}

.show-more-btn {
  padding: 8px 16px;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border: 1px solid rgba(102, 126, 234, 0.2);
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.show-more-btn:hover {
  background: rgba(102, 126, 234, 0.2);
  transform: translateY(-1px);
}

.additional-filters {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding-top: 20px;
  border-top: 1px solid #e5e7eb;
  margin-top: 20px;
  animation: slideDown 0.3s ease;
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

/* Table Section */
.table-section {
  margin-top: 24px;
}

.table-container {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
}

.table-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  background: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.table-info {
  display: flex;
  align-items: center;
  gap: 16px;
}

.table-count {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  background: white;
  padding: 6px 12px;
  border-radius: 6px;
  border: 1px solid #e5e7eb;
}

.table-scroll-hint {
  font-size: 13px;
  color: #6b7280;
  display: flex;
  align-items: center;
  gap: 6px;
}

.hint-icon {
  font-size: 14px;
}

.table-actions {
  display: flex;
  gap: 8px;
}

.table-action-btn {
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

.table-action-btn:hover {
  background: #f3f4f6;
  border-color: #d1d5db;
  transform: translateY(-1px);
}

.action-icon {
  font-size: 14px;
}

/* Table Wrapper */
.table-wrapper {
  width: 100%;
  overflow-x: auto;
  max-height: 500px;
  overflow-y: auto;
}

.data-table {
  width: 100%;
  min-width: max-content;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 13px;
}

/* Table Header */
.table-header {
  padding: 16px 12px;
  text-align: left;
  font-weight: 600;
  color: #374151;
  background: #f3f4f6;
  border-bottom: 2px solid #e5e7eb;
  border-right: 1px solid #e5e7eb;
  position: sticky;
  top: 0;
  z-index: 10;
  min-width: 180px;
  white-space: nowrap;
}

.table-header.first-column {
  position: sticky;
  left: 0;
  z-index: 20;
  background: #e5e7eb;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
}

.header-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.column-number {
  font-size: 11px;
  color: #6b7280;
  font-weight: 500;
  background: rgba(0, 0, 0, 0.05);
  padding: 2px 6px;
  border-radius: 4px;
}

.column-name {
  flex: 1;
  font-weight: 600;
  color: #1f2937;
  overflow: hidden;
  text-overflow: ellipsis;
}

.column-type {
  font-size: 10px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 4px;
  text-transform: uppercase;
}

.column-type.type-category {
  background: #f3e8ff;
  color: #8b5cf6;
}

.column-type.type-number {
  background: #dbeafe;
  color: #3b82f6;
}

/* Table Body */
.table-cell {
  padding: 14px 12px;
  border-bottom: 1px solid #f3f4f6;
  border-right: 1px solid #f3f4f6;
  color: #4b5563;
  background: white;
  max-width: 300px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.table-cell.first-column {
  position: sticky;
  left: 0;
  z-index: 5;
  background: white;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.05);
}

.alternate-row .table-cell {
  background: #fafbfd;
}

.alternate-row .table-cell.first-column {
  background: #fafbfd;
}

.data-table tbody tr:hover .table-cell {
  background: #f8fafc !important;
}

.data-table tbody tr:hover .table-cell.first-column {
  background: #f8fafc !important;
}

/* Table Footer */
.table-footer {
  padding: 16px 20px;
  background: #f9fafb;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.row-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.row-count {
  font-size: 14px;
  color: #374151;
  font-weight: 500;
}

.row-more {
  font-size: 12px;
  color: #6b7280;
  margin-left: 8px;
}

.scroll-hint {
  font-size: 12px;
  color: #6b7280;
  display: flex;
  align-items: center;
  gap: 6px;
}

.scroll-text {
  background: white;
  padding: 4px 12px;
  border-radius: 20px;
  border: 1px solid #e5e7eb;
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

/* Scrollbar Styling */
.table-wrapper::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

.table-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 5px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 5px;
  border: 2px solid #f1f5f9;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

/* Tooltip for cells */
.table-cell[title]:hover::after {
  content: attr(title);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: #1f2937;
  color: white;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 12px;
  white-space: normal;
  max-width: 300px;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  pointer-events: none;
}

.table-cell[title]:hover::before {
  content: '';
  position: absolute;
  left: 50%;
  bottom: calc(100% - 6px);
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top-color: #1f2937;
  z-index: 1001;
  pointer-events: none;
}

/* Responsive Design */
@media (max-width: 1024px) {
  .header-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }

  .stats-group {
    flex-wrap: wrap;
  }

  .filters-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .table-header-row {
    flex-direction: column;
    gap: 12px;
    align-items: stretch;
  }

  .table-info {
    justify-content: space-between;
  }

  .table-actions {
    justify-content: center;
  }
}

@media (max-width: 768px) {
  .card-header {
    padding: 20px 24px;
  }

  .card-title {
    font-size: 20px;
  }

  .stats-group {
    gap: 12px;
  }

  .stat-item {
    min-width: 100px;
    padding: 10px 16px;
  }

  .card-body {
    padding: 24px;
  }

  .filters-grid {
    grid-template-columns: 1fr;
  }

  .table-wrapper {
    max-height: 400px;
  }

  .table-cell {
    max-width: 200px;
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
    min-width: 80px;
    padding: 8px 12px;
  }

  .stat-icon {
    width: 32px;
    height: 32px;
    font-size: 18px;
  }

  .stat-value {
    font-size: 18px;
  }

  .card-body {
    padding: 20px;
  }

  .filters-section {
    padding: 16px;
  }

  .table-header-row {
    padding: 12px 16px;
  }

  .empty-state {
    padding: 32px 20px;
  }

  .empty-icon {
    font-size: 48px;
  }
}
</style>