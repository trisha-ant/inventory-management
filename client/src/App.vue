<template>
  <div class="app" :class="{ 'sidebar-collapsed': isSidebarCollapsed }">
    <aside class="sidebar">
      <div class="sidebar-header">
        <div class="logo">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <button class="sidebar-toggle" @click="toggleSidebar" :aria-label="isSidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M10 4L6 8L10 12" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        </button>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }" :data-label="t('nav.overview')">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }" :data-label="t('nav.inventory')">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }" :data-label="t('nav.orders')">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1"/></svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }" :data-label="t('nav.finance')">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }" :data-label="t('nav.demandForecast')">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }" data-label="Reports">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu @show-profile-details="showProfileDetails = true" @show-tasks="showTasks = true" />
      </div>
    </aside>

    <div class="content-area">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const isSidebarCollapsed = ref(localStorage.getItem('sidebarCollapsed') === 'true')
    const toggleSidebar = () => {
      isSidebarCollapsed.value = !isSidebarCollapsed.value
      localStorage.setItem('sidebarCollapsed', isSidebarCollapsed.value)
    }

    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isSidebarCollapsed,
      toggleSidebar
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Geist:wght@400;500;600&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Geist', -apple-system, sans-serif;
  background: #fafafa;
  color: #18181b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

:root {
  --sidebar-bg: #18181b;
  --sidebar-border: #27272a;
  --sidebar-text: #a1a1aa;
  --sidebar-text-hover: #fafafa;
  --sidebar-active-bg: #27272a;

  --content-bg: #fafafa;
  --surface: #ffffff;
  --border: #e4e4e7;
  --border-hover: #d4d4d8;

  --text-primary: #18181b;
  --text-secondary: #71717a;
  --text-tertiary: #a1a1aa;

  --accent: #6366f1;
  --accent-bg: #eef2ff;

  --success: #10b981;
  --warning: #f59e0b;
  --danger: #ef4444;
  --info: #6366f1;
}

.app {
  display: grid;
  grid-template-columns: 240px 1fr;
  height: 100vh;
  overflow: hidden;
  transition: grid-template-columns 0.2s ease;
}

.app.sidebar-collapsed {
  grid-template-columns: 64px 1fr;
}

.sidebar {
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-header {
  padding: 1.25rem 1rem;
  border-bottom: 1px solid var(--sidebar-border);
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
}

.sidebar-header .logo h1 {
  font-size: 0.9375rem;
  font-weight: 600;
  color: #fafafa;
  letter-spacing: -0.01em;
}

.sidebar-header .logo .subtitle {
  font-size: 0.75rem;
  color: var(--sidebar-text);
  display: block;
  margin-top: 0.125rem;
}

.sidebar-toggle {
  background: transparent;
  border: none;
  color: var(--sidebar-text);
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 4px;
  transition: color 0.15s ease, background 0.15s ease, transform 0.2s ease;
  flex-shrink: 0;
  margin-top: 0.125rem;
}

.sidebar-toggle:hover {
  color: var(--sidebar-text-hover);
  background: var(--sidebar-active-bg);
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  color: var(--sidebar-text);
  font-size: 0.8125rem;
  font-weight: 500;
  text-decoration: none;
  transition: color 0.15s ease, background 0.15s ease;
  position: relative;
}

.sidebar-nav a:hover {
  color: var(--sidebar-text-hover);
  background: var(--sidebar-active-bg);
}

.sidebar-nav a.active {
  color: var(--sidebar-text-hover);
  background: var(--sidebar-active-bg);
}

.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: -0.5rem;
  top: 0.25rem;
  bottom: 0.25rem;
  width: 2px;
  background: var(--accent);
  border-radius: 1px;
}

.nav-icon {
  flex-shrink: 0;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  transition: opacity 0.2s ease, width 0.2s ease;
}

.sidebar-footer {
  padding: 0.75rem 0.5rem;
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

/* === COLLAPSED STATE === */

.sidebar-collapsed .sidebar-header {
  justify-content: center;
  padding: 1.25rem 0.5rem;
}

.sidebar-collapsed .logo h1,
.sidebar-collapsed .logo .subtitle,
.sidebar-collapsed .nav-label {
  opacity: 0;
  width: 0;
  overflow: hidden;
}

.sidebar-collapsed .sidebar-toggle {
  transform: rotate(180deg);
}

.sidebar-collapsed .sidebar-nav {
  padding: 0.75rem 0.5rem;
}

.sidebar-collapsed .sidebar-nav a {
  justify-content: center;
  padding: 0.625rem;
}

.sidebar-collapsed .sidebar-nav a.active::before {
  left: -0.5rem;
}

.sidebar-collapsed .sidebar-footer {
  padding: 0.75rem 0.25rem;
  align-items: center;
}

/* CSS-only tooltips when collapsed */
.sidebar-collapsed .sidebar-nav a::after {
  content: attr(data-label);
  position: absolute;
  left: calc(100% + 0.5rem);
  top: 50%;
  transform: translateY(-50%);
  background: #27272a;
  color: #fafafa;
  padding: 0.375rem 0.625rem;
  border-radius: 6px;
  font-size: 0.75rem;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.15s ease;
  z-index: 50;
}

.sidebar-collapsed .sidebar-nav a:hover::after {
  opacity: 1;
}

/* Auto-collapse below 1024px */
@media (max-width: 1024px) {
  .app {
    grid-template-columns: 64px 1fr;
  }

  .logo h1,
  .logo .subtitle,
  .nav-label {
    opacity: 0;
    width: 0;
    overflow: hidden;
  }

  .sidebar-header {
    justify-content: center;
    padding: 1.25rem 0.5rem;
  }

  .sidebar-nav {
    padding: 0.75rem 0.5rem;
  }

  .sidebar-nav a {
    justify-content: center;
    padding: 0.625rem;
  }

  .sidebar-footer {
    padding: 0.75rem 0.25rem;
    align-items: center;
  }

  .sidebar-toggle {
    display: none;
  }

  .sidebar-nav a {
    position: relative;
  }

  .sidebar-nav a::after {
    content: attr(data-label);
    position: absolute;
    left: calc(100% + 0.5rem);
    top: 50%;
    transform: translateY(-50%);
    background: #27272a;
    color: #fafafa;
    padding: 0.375rem 0.625rem;
    border-radius: 6px;
    font-size: 0.75rem;
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.15s ease;
    z-index: 50;
  }

  .sidebar-nav a:hover::after {
    opacity: 1;
  }
}

.content-area {
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  background: var(--content-bg);
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
}

.page-header {
  margin-bottom: 1.25rem;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: -0.02em;
  margin-bottom: 0.25rem;
}

.page-header p {
  color: var(--text-secondary);
  font-size: 0.875rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.stat-card,
.card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 1rem;
  transition: border-color 0.15s ease;
}

.stat-card:hover,
.card:hover {
  border-color: var(--border-hover);
}

.card {
  margin-bottom: 1rem;
}

.stat-label {
  color: var(--text-secondary);
  font-size: 0.75rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 1.875rem;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

.stat-card.warning .stat-value { color: var(--warning); }
.stat-card.success .stat-value { color: var(--success); }
.stat-card.danger .stat-value  { color: var(--danger); }
.stat-card.info .stat-value    { color: var(--info); }

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.875rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 0.9375rem;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: -0.01em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  border-bottom: 1px solid var(--border);
}

th {
  text-align: left;
  padding: 0.625rem 0.75rem;
  font-weight: 500;
  color: var(--text-secondary);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

td {
  padding: 0.625rem 0.75rem;
  border-top: 1px solid var(--border);
  color: var(--text-primary);
  font-size: 0.8125rem;
}

tbody tr {
  transition: background 0.15s ease;
}

tbody tr:hover {
  background: #fafafa;
}

.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #fecaca; color: #991b1b; }
.badge.info       { background: var(--accent-bg); color: var(--accent); }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable     { background: var(--accent-bg); color: var(--accent); }
.badge.high       { background: #fecaca; color: #991b1b; }
.badge.medium     { background: #fef3c7; color: #92400e; }
.badge.low        { background: var(--accent-bg); color: var(--accent); }

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--text-secondary);
  font-size: 0.875rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.875rem;
}
</style>
