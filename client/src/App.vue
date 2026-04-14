<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-logo">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" exact-active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="7" height="7" rx="1.5" />
            <rect x="14" y="3" width="7" height="7" rx="1.5" />
            <rect x="3" y="14" width="7" height="7" rx="1.5" />
            <rect x="14" y="14" width="7" height="7" rx="1.5" />
          </svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z" />
            <polyline points="3.27 6.96 12 12.01 20.73 6.96" />
            <line x1="12" y1="22.08" x2="12" y2="12" />
          </svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z" />
            <line x1="3" y1="6" x2="21" y2="6" />
            <path d="M16 10a4 4 0 0 1-8 0" />
          </svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10" />
            <line x1="12" y1="6" x2="12" y2="18" />
            <path d="M15 9H10.5a2.5 2.5 0 0 0 0 5h3a2.5 2.5 0 0 1 0 5H9" />
          </svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="23 6 13.5 15.5 8.5 10.5 1 18" />
            <polyline points="17 6 23 6 23 12" />
          </svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" active-class="active" class="nav-link">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="20" x2="18" y2="10" />
            <line x1="12" y1="20" x2="12" y2="4" />
            <line x1="6" y1="20" x2="6" y2="14" />
          </svg>
          <span class="nav-label">{{ t('nav.reports') }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="app-main">
      <header class="topbar">
        <h2 class="page-title">{{ currentPageTitle }}</h2>
        <div class="topbar-actions">
          <LanguageSwitcher />
        </div>
      </header>

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
import { useRoute } from 'vue-router'
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
    const route = useRoute()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const currentPageTitle = computed(() => {
      const titles = {
        '/': t('nav.overview'),
        '/inventory': t('nav.inventory'),
        '/orders': t('nav.orders'),
        '/spending': t('nav.finance'),
        '/demand': t('nav.demandForecast'),
        '/reports': t('nav.reports')
      }
      return titles[route.path] || ''
    })

    // Merge mock tasks from currentUser with API tasks
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
          mockTask.completed = !mockTask.completed
        } else {
          const updated = await api.toggleTask(taskId)
          const idx = apiTasks.value.findIndex(t => t.id === taskId)
          if (idx !== -1) {
            apiTasks.value[idx] = updated
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      currentPageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
/* unscoped so tokens cascade to every child component */
:root {
  --bg-app: #f6f7f9;
  --bg-surface: #ffffff;
  --bg-sidebar: #0f172a;
  --bg-subtle: #f1f5f9;
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  --text-on-dark: #e2e8f0;
  --text-on-dark-muted: #94a3b8;
  --border: #e2e8f0;
  --border-strong: #cbd5e1;
  --accent: #4f46e5;
  --accent-hover: #4338ca;
  --accent-soft: #eef2ff;
  --accent-on: #ffffff;
  --success: #16a34a;
  --success-soft: #dcfce7;
  --warning: #d97706;
  --warning-soft: #fef3c7;
  --danger: #dc2626;
  --danger-soft: #fee2e2;
  --info: #2563eb;
  --info-soft: #dbeafe;
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 1px 3px rgba(15, 23, 42, 0.06), 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-lg: 0 10px 15px -3px rgba(15, 23, 42, 0.08), 0 4px 6px -4px rgba(15, 23, 42, 0.05);
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --radius-pill: 999px;
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;
  --sidebar-w: 240px;
  --sidebar-w-collapsed: 68px;
  --header-h: 64px;
  --font-sans: -apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, sans-serif;
  --font-mono: "SF Mono", Menlo, Consolas, monospace;
  --text-xs: 12px;
  --text-sm: 13px;
  --text-base: 14px;
  --text-md: 15px;
  --text-lg: 17px;
  --text-xl: 20px;
  --text-2xl: 24px;
  --text-3xl: 30px;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --line-tight: 1.25;
  --line-normal: 1.5;
}
html, body, #app { height: 100%; }
body {
  margin: 0;
  background: var(--bg-app);
  color: var(--text-primary);
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--line-normal);
  -webkit-font-smoothing: antialiased;
}
* { box-sizing: border-box; }
a { color: inherit; text-decoration: none; }

/* Global table styling — views use bare <table> so these rules cascade */
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: var(--text-sm);
  color: var(--text-primary);
}
thead th {
  background: var(--bg-subtle);
  color: var(--text-muted);
  font-weight: var(--weight-semibold);
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  text-align: left;
  padding: var(--space-3) var(--space-4);
  border-bottom: 1px solid var(--border);
  white-space: nowrap;
}
thead th:first-child { border-top-left-radius: var(--radius-md); }
thead th:last-child  { border-top-right-radius: var(--radius-md); }
tbody td {
  padding: var(--space-3) var(--space-4);
  border-bottom: 1px solid var(--border);
  vertical-align: middle;
  color: var(--text-primary);
}
tbody tr:last-child td { border-bottom: none; }
tbody tr:hover td { background: var(--bg-subtle); }

/* Generic status/metric badges used across views */
.badge {
  display: inline-block;
  padding: 2px var(--space-2);
  border-radius: var(--radius-pill);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  line-height: 1.4;
  letter-spacing: 0;
  text-transform: none;
}
.badge.success    { background: var(--success-soft); color: var(--success); }
.badge.warning    { background: var(--warning-soft); color: var(--warning); }
.badge.danger     { background: var(--danger-soft);  color: var(--danger); }
.badge.info       { background: var(--info-soft);    color: var(--info); }
.badge.delivered  { background: var(--success-soft); color: var(--success); }
.badge.shipped    { background: var(--info-soft);    color: var(--info); }
.badge.processing { background: var(--warning-soft); color: var(--warning); }
.badge.backordered{ background: var(--danger-soft);  color: var(--danger); }
.badge.increasing { background: var(--success-soft); color: var(--success); }
.badge.stable     { background: var(--info-soft);    color: var(--info); }
.badge.decreasing { background: var(--danger-soft);  color: var(--danger); }

/* Card wrapper used by views */
.card {
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
}
</style>

<style scoped>
.app {
  min-height: 100vh;
  background: var(--bg-app);
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: var(--sidebar-w);
  background: var(--bg-sidebar);
  color: var(--text-on-dark);
  display: flex;
  flex-direction: column;
  z-index: 50;
}

.sidebar-logo {
  padding: var(--space-6);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}
.sidebar-logo h1 {
  margin: 0;
  font-size: var(--text-lg);
  font-weight: var(--weight-semibold);
  color: var(--text-on-dark);
  letter-spacing: -0.01em;
}
.sidebar-subtitle {
  display: block;
  margin-top: var(--space-1);
  font-size: var(--text-xs);
  color: var(--text-on-dark-muted);
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-2) 0;
  overflow-y: auto;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-4);
  margin: 2px var(--space-3);
  border-radius: var(--radius-md);
  color: var(--text-on-dark-muted);
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  transition: background 120ms ease, color 120ms ease;
}
.nav-link:hover {
  background: rgba(255, 255, 255, 0.06);
  color: var(--text-on-dark);
}
.nav-link.active {
  background: var(--accent);
  color: var(--accent-on);
}
.nav-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.sidebar-footer {
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  padding: var(--space-3);
}

.app-main {
  margin-left: var(--sidebar-w);
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.topbar {
  position: sticky;
  top: 0;
  z-index: 40;
  height: var(--header-h);
  background: var(--bg-surface);
  border-bottom: 1px solid var(--border);
  padding: 0 var(--space-8);
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.page-title {
  margin: 0;
  font-size: var(--text-xl);
  font-weight: var(--weight-semibold);
  color: var(--text-primary);
  letter-spacing: -0.01em;
}
.topbar-actions {
  display: flex;
  align-items: center;
  gap: var(--space-3);
}

.main-content {
  padding: var(--space-8);
  max-width: 1400px;
  width: 100%;
  margin: 0 auto;
}

@media (max-width: 1024px) {
  .sidebar {
    width: var(--sidebar-w-collapsed);
  }
  .sidebar-logo h1,
  .sidebar-subtitle,
  .nav-label {
    display: none;
  }
  .nav-link {
    justify-content: center;
    padding: var(--space-3);
    margin: 2px var(--space-2);
  }
  .sidebar-logo {
    padding: var(--space-4);
    display: flex;
    justify-content: center;
  }
  .app-main {
    margin-left: var(--sidebar-w-collapsed);
  }
  .topbar {
    padding: 0 var(--space-4);
  }
  .main-content {
    padding: var(--space-4);
  }
}
</style>
