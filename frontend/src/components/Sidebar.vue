<!-- Sidebar.vue -->
<template>
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
        <li :class="{ active: activeMenu === 'upload' }" @click="$emit('update-menu', 'upload')">
          <span class="nav-icon">📤</span>
          <span class="nav-text">Upload Data</span>
        </li>
        <li :class="{ active: activeMenu === 'table' }" @click="$emit('update-menu', 'table')">
          <span class="nav-icon">📋</span>
          <span class="nav-text">Data Table</span>
        </li>
        <li :class="{ active: activeMenu === 'chart' }" @click="$emit('update-menu', 'chart')">
          <span class="nav-icon">📈</span>
          <span class="nav-text">Visualization</span>
        </li>
        <li :class="{ active: activeMenu === 'eda' }" @click="$emit('update-menu', 'eda')">
          <span class="nav-icon">🔍</span>
          <span class="nav-text">Auto Profiling</span>
          <span class="nav-badge">EDA</span>
        </li>
        <li :class="{ active: activeMenu === 'insight' }" @click="$emit('update-menu', 'insight')">
          <span class="nav-icon">🧠</span>
          <span class="nav-text">AI Insight</span>
          <span class="nav-badge">AI</span>
        </li>

        <li :class="{ active: activeMenu === 'ExportMenu' }" @click="$emit('update-menu', 'ExportMenu')">
          <span class="nav-icon">🧠</span>
          <span class="nav-text">Export Menu</span>
          <span class="nav-badge">Export</span>
        </li>
      </ul>
    </nav>
  </aside>
</template>

<script setup>
import { defineProps } from 'vue'
defineProps({ activeMenu: String })
defineEmits(['update-menu'])
</script>

<style scoped>
/* Sidebar Styles */
.sidebar {
  width: 280px;
  min-width: 280px;
  height: 100vh;
  background: linear-gradient(180deg, #1a1f36 0%, #111827 100%);
  color: white;
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 1000;
  box-shadow: 4px 0 20px rgba(0, 0, 0, 0.2);
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
  transition: all 0.3s ease;
}

.sidebar-header {
  padding: 28px 24px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.03);
}

.logo-container {
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: flex-start;
}

.logo-icon {
  width: 48px;
  height: 48px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  margin-bottom: 12px;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.logo-text {
  font-size: 26px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffffff 0%, #a5b4fc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: -0.5px;
  margin: 0;
  line-height: 1.2;
}

.logo-subtitle {
  font-size: 13px;
  color: #94a3b8;
  font-weight: 500;
  letter-spacing: 1px;
  text-transform: uppercase;
  opacity: 0.8;
}

.sidebar-nav {
  flex: 1;
  padding: 24px 16px;
  overflow-y: auto;
}

.sidebar-nav ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.sidebar-nav li {
  padding: 14px 16px;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 15px;
  font-weight: 500;
  color: #cbd5e1;
  display: flex;
  align-items: center;
  gap: 12px;
  position: relative;
  border: 1px solid transparent;
  user-select: none;
}

.sidebar-nav li:hover {
  background: rgba(255, 255, 255, 0.05);
  color: #ffffff;
  transform: translateX(4px);
  border-color: rgba(255, 255, 255, 0.1);
}

.sidebar-nav li.active {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.15) 0%, rgba(118, 75, 162, 0.15) 100%);
  color: #ffffff;
  border-color: rgba(102, 126, 234, 0.3);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
  position: relative;
  transform: translateX(4px);
}

.sidebar-nav li.active::before {
  content: '';
  position: absolute;
  left: -16px;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 60%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 0 4px 4px 0;
  animation: slideIn 0.3s ease;
}

.nav-icon {
  font-size: 18px;
  width: 24px;
  text-align: center;
  transition: transform 0.3s ease;
}

.nav-text {
  flex: 1;
}

.nav-badge {
  background: rgba(255, 255, 255, 0.1);
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.sidebar-nav li.active .nav-badge {
  background: rgba(255, 255, 255, 0.2);
  color: white;
}

/* Responsive */
@media (max-width: 1024px) {
  .sidebar {
    width: 240px;
    min-width: 240px;
  }
}

@media (max-width: 768px) {
  .sidebar {
    width: 100%;
    height: 60px;
    flex-direction: row;
    align-items: center;
    padding: 0;
  }

  .sidebar-header {
    padding: 0 16px;
    border-bottom: none;
    border-right: 1px solid rgba(255, 255, 255, 0.1);
    height: 60px;
    display: flex;
    align-items: center;
  }

  .logo-container {
    flex-direction: row;
    align-items: center;
    gap: 12px;
  }

  .logo-icon {
    width: 36px;
    height: 36px;
    font-size: 18px;
    margin-bottom: 0;
    margin-right: 8px;
  }

  .logo-text {
    font-size: 18px;
  }

  .logo-subtitle {
    display: none;
  }

  .sidebar-nav {
    padding: 0;
    flex: 1;
    overflow-x: auto;
  }

  .sidebar-nav ul {
    flex-direction: row;
    height: 60px;
    padding: 0 12px;
    align-items: center;
  }

  .sidebar-nav li {
    padding: 8px 12px;
    white-space: nowrap;
    height: 40px;
    display: flex;
    align-items: center;
  }

  .sidebar-nav li.active::before {
    left: 50%;
    top: -8px;
    transform: translateX(-50%);
    width: 60%;
    height: 3px;
    border-radius: 3px 3px 0 0;
  }

  .nav-text {
    display: none;
  }

  .nav-badge {
    display: none;
  }
}

@media (max-width: 480px) {
  .sidebar {
    height: 56px;
  }

  .sidebar-header {
    padding: 0 12px;
  }

  .logo-icon {
    width: 32px;
    height: 32px;
    font-size: 16px;
  }

  .logo-text {
    font-size: 16px;
  }

  .sidebar-nav li {
    padding: 6px 10px;
  }

  .nav-icon {
    font-size: 16px;
  }
}
</style>