<template>
  <!-- SaaS shell: fixed-width sidebar column + fluid workspace column (see .app grid) -->
  <div class="app">
    <AppSidebar
      :mobile-open="isSidebarOpen"
      @close="isSidebarOpen = false"
      @show-profile-details="showProfileDetails = true"
      @show-tasks="showTasks = true"
    />

    <div class="workspace">
      <header class="topbar">
        <div class="topbar-row">
          <!-- Hamburger is only displayed below 768px, where the sidebar becomes an off-canvas drawer -->
          <button
            type="button"
            class="hamburger"
            aria-controls="app-sidebar"
            :aria-expanded="isSidebarOpen ? 'true' : 'false'"
            :aria-label="t('nav.openMenu')"
            @click="isSidebarOpen = true"
          >
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" aria-hidden="true">
              <path d="M4 6h16" />
              <path d="M4 12h16" />
              <path d="M4 18h16" />
            </svg>
          </button>
          <h1 class="topbar-title">{{ pageTitle }}</h1>
        </div>
        <FilterBar />
      </header>

      <main class="main-content">
        <div class="content-inner">
          <router-view />
        </div>
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
import { ref, onMounted, computed, watch } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import AppSidebar from './components/AppSidebar.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    AppSidebar,
    ProfileDetailsModal,
    TasksModal
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Layout state: mobile drawer visibility (desktop collapse state lives in AppSidebar)
    const route = useRoute()
    const isSidebarOpen = ref(false)

    // Top bar title uses the same nav.* keys as the sidebar so both stay in sync across locales
    const routeTitleKeys = {
      '/': 'nav.overview',
      '/inventory': 'nav.inventory',
      '/orders': 'nav.orders',
      '/spending': 'nav.finance',
      '/demand': 'nav.demandForecast',
      '/reports': 'nav.reports'
    }
    const pageTitle = computed(() => t(routeTitleKeys[route.path] || 'nav.companyName'))

    // Close the mobile drawer after navigating
    watch(() => route.fullPath, () => {
      isSidebarOpen.value = false
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
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
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
      isSidebarOpen,
      pageTitle,
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
:root {
  /* Spacing scale (4px base) */
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-12: 3rem;    /* 48px */

  /* Neutrals (slate) */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-border: #e2e8f0;
  --color-border-strong: #cbd5e1;
  --color-text: #0f172a;
  --color-text-body: #334155;
  --color-text-muted: #64748b;
  --color-hover: #f1f5f9;

  /* Brand and status */
  --color-primary: #2563eb;
  --color-primary-soft: #eff6ff;
  --color-success: #059669;
  --color-warning: #ea580c;
  --color-danger: #dc2626;
  --color-info: #2563eb;

  /* Sidebar (dark slate) */
  --sidebar-bg: #0f172a;
  --sidebar-text: #cbd5e1;
  --sidebar-text-muted: #94a3b8;
  --sidebar-hover: #1e293b;
  --sidebar-active-bg: #1e293b;
  --sidebar-active-text: #ffffff;
  --sidebar-active-indicator: #3b82f6;

  /* Shape */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 4px 12px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 12px 32px rgba(15, 23, 42, 0.12);

  /* Layout */
  --sidebar-w: 248px;
  --sidebar-w-collapsed: 72px;
  --topbar-h: 60px;
  --content-max: 1440px;

  /* Breakpoints (px). Media queries cannot read custom properties, so these literals are
     mirrored in AppSidebar.vue (BREAKPOINTS). Keep both in sync.
     < 768: off-canvas drawer + hamburger | 768-1279: icons-only rail | >= 1280: full sidebar */

  /* Layering: modals already use their own higher z-index; keep them above these */
  --z-topbar: 150;
  --z-sidebar: 200;
  --z-backdrop: 190;

  --transition: 0.2s ease;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* App shell: sidebar column sizes itself (expanded/collapsed width), workspace takes the rest */
.app {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr);
  min-height: 100vh;
}

/* Tablet rail (768px to 1279px): the sidebar is always icons-only, so the column is fixed at rail width.
   An expanded rail overlays the content instead of changing this column, so nothing jumps. */
@media (min-width: 768px) and (max-width: 1279px) {
  .app {
    grid-template-columns: var(--sidebar-w-collapsed) minmax(0, 1fr);
  }
}

/* min-width: 0 lets wide tables scroll inside .table-container instead of widening the grid */
.workspace {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

/* Sticky top bar holds the page title row and the FilterBar row */
.topbar {
  position: sticky;
  top: 0;
  z-index: var(--z-topbar);
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
}

.topbar-row {
  height: var(--topbar-h);
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 0 var(--space-8);
}

.topbar-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: var(--color-text);
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.hamburger {
  display: none;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  margin-left: calc(-1 * var(--space-2));
  background: none;
  border: none;
  border-radius: var(--radius-md);
  color: var(--color-text-body);
  cursor: pointer;
  transition: background var(--transition);
}

.hamburger:hover {
  background: var(--color-hover);
}

.hamburger:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.main-content {
  flex: 1;
  width: 100%;
  padding: var(--space-6) var(--space-8);
}

/* Max-width applies to the inner content only so the gutter stays aligned with the top bar */
.content-inner {
  max-width: var(--content-max);
  margin: 0 auto;
}

/* Below 768px the sidebar is an off-canvas drawer (position: fixed), so the grid is single-column */
@media (max-width: 767px) {
  .app {
    grid-template-columns: minmax(0, 1fr);
  }

  .hamburger {
    display: inline-flex;
  }

  .topbar-row,
  .main-content {
    padding-left: var(--space-4);
    padding-right: var(--space-4);
  }
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-text);
  margin-bottom: var(--space-1);
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--color-text-muted);
  font-size: 0.875rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-6);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  transition: box-shadow var(--transition), border-color var(--transition);
}

.stat-card:hover {
  border-color: var(--color-border-strong);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--color-text);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: var(--color-warning);
}

.stat-card.success .stat-value {
  color: var(--color-success);
}

.stat-card.danger .stat-value {
  color: var(--color-danger);
}

.stat-card.info .stat-value {
  color: var(--color-info);
}

.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-6);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-text);
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
  background: var(--color-bg);
  border-top: 1px solid var(--color-border);
  border-bottom: 1px solid var(--color-border);
}

th {
  text-align: left;
  padding: var(--space-3) var(--space-4);
  font-weight: 600;
  background: var(--color-bg);
  color: var(--color-text-muted);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: var(--space-3) var(--space-4);
  border-top: 1px solid var(--color-hover);
  color: var(--color-text-body);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-bg);
}

/* Pill badges; status palettes unchanged so existing semantics stay recognizable */
.badge {
  display: inline-block;
  padding: var(--space-1) var(--space-3);
  border-radius: 999px;
  font-size: 0.75rem;
  font-weight: 600;
  line-height: 1.25;
  white-space: nowrap;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--color-text-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: 0.938rem;
}
</style>
