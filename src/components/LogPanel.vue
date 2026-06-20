<template>
  <div class="log-panel">
    <h3 class="panel-title">日志</h3>
    <div class="filter-bar">
      <button 
        v-for="filter in filters" 
        :key="filter.value"
        class="filter-btn"
        :class="{ active: activeFilter === filter.value }"
        @click="activeFilter = filter.value"
      >
        <span class="filter-icon">{{ filter.icon }}</span>
        <span class="filter-label">{{ filter.label }}</span>
        <span v-if="getFilterCount(filter.value) > 0" class="filter-count">
          {{ getFilterCount(filter.value) }}
        </span>
      </button>
    </div>
    <div class="log-list" ref="logListRef">
      <div 
        v-for="(log, index) in filteredLogs" 
        :key="index" 
        class="log-item" 
        :class="[log.type, { 'key-node': log.keyNode }]"
      >
        <span v-if="log.keyNode" class="key-node-badge">⭐</span>
        <span class="log-time">[{{ log.timestamp }}]</span>
        <span class="log-message">{{ log.message }}</span>
      </div>
      <div v-if="filteredLogs.length === 0" class="empty-log">
        暂无日志
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, nextTick } from 'vue'

const props = defineProps({
  logs: {
    type: Array,
    default: () => []
  }
})

const activeFilter = ref('all')
const logListRef = ref(null)

const filters = [
  { value: 'all', label: '全部', icon: '📋' },
  { value: 'keyNode', label: '关键', icon: '⭐' },
  { value: 'info', label: '信息', icon: 'ℹ️' },
  { value: 'success', label: '成功', icon: '✅' },
  { value: 'warning', label: '警告', icon: '⚠️' },
  { value: 'danger', label: '危险', icon: '🚨' },
  { value: 'action', label: '行动', icon: '🎯' }
]

const filteredLogs = computed(() => {
  if (activeFilter.value === 'all') {
    return props.logs
  }
  if (activeFilter.value === 'keyNode') {
    return props.logs.filter(log => log.keyNode)
  }
  return props.logs.filter(log => log.type === activeFilter.value)
})

function getFilterCount(filterValue) {
  if (filterValue === 'all') {
    return props.logs.length
  }
  if (filterValue === 'keyNode') {
    return props.logs.filter(log => log.keyNode).length
  }
  return props.logs.filter(log => log.type === filterValue).length
}

watch(() => props.logs.length, () => {
  nextTick(() => {
    if (logListRef.value) {
      logListRef.value.scrollTop = 0
    }
  })
})
</script>

<style scoped>
.log-panel {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
  height: 100%;
  max-height: 400px;
}

.panel-title {
  color: white;
  font-size: 18px;
  margin-bottom: 15px;
  text-align: center;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.filter-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.filter-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 6px 12px;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  color: rgba(255, 255, 255, 0.6);
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.filter-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  color: white;
  transform: translateY(-1px);
}

.filter-btn.active {
  background: linear-gradient(135deg, #3498db, #2980b9);
  border-color: #3498db;
  color: white;
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.4);
}

.filter-icon {
  font-size: 12px;
}

.filter-label {
  font-weight: 500;
}

.filter-count {
  background: rgba(255, 255, 255, 0.2);
  padding: 1px 6px;
  border-radius: 10px;
  font-size: 10px;
  font-weight: bold;
}

.filter-btn.active .filter-count {
  background: rgba(255, 255, 255, 0.3);
}

.log-list {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding-right: 10px;
}

.log-list::-webkit-scrollbar {
  width: 6px;
}

.log-list::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}

.log-list::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.log-item {
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 12px;
  line-height: 1.4;
  animation: fadeIn 0.3s ease;
  display: flex;
  align-items: flex-start;
  gap: 6px;
}

.log-item.key-node {
  animation: keyNodeGlow 2s ease-in-out infinite;
  border-width: 2px;
  position: relative;
}

@keyframes keyNodeGlow {
  0%, 100% {
    box-shadow: 0 0 5px rgba(241, 196, 15, 0.3);
  }
  50% {
    box-shadow: 0 0 15px rgba(241, 196, 15, 0.6);
  }
}

.key-node-badge {
  flex-shrink: 0;
  font-size: 12px;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateX(-10px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.log-item.info {
  background: rgba(52, 152, 219, 0.3);
  border-left: 3px solid #3498db;
  color: #aed6f1;
}

.log-item.success {
  background: rgba(46, 204, 113, 0.3);
  border-left: 3px solid #2ecc71;
  color: #abebc6;
}

.log-item.warning {
  background: rgba(243, 156, 18, 0.3);
  border-left: 3px solid #f39c12;
  color: #fad7a0;
}

.log-item.danger {
  background: rgba(231, 76, 60, 0.3);
  border-left: 3px solid #e74c3c;
  color: #f5b7b1;
}

.log-item.action {
  background: rgba(155, 89, 182, 0.3);
  border-left: 3px solid #9b59b6;
  color: #d2b4de;
}

.log-time {
  color: rgba(255, 255, 255, 0.5);
  margin-right: 8px;
  font-size: 10px;
}

.empty-log {
  text-align: center;
  color: rgba(255, 255, 255, 0.4);
  padding: 20px;
  font-style: italic;
}
</style>
