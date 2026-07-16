<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-brand">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>

      <nav class="sidebar-nav" aria-label="Main navigation">
        <!-- Root link uses exact path match so it isn't marked active on every route -->
        <router-link
          to="/"
          :class="{ active: $route.path === '/' }"
          :aria-current="$route.path === '/' ? 'page' : undefined"
        >
          {{ t('nav.overview') }}
        </router-link>
        <router-link
          to="/inventory"
          :class="{ active: $route.path === '/inventory' }"
          :aria-current="$route.path === '/inventory' ? 'page' : undefined"
        >
          {{ t('nav.inventory') }}
        </router-link>
        <router-link
          to="/orders"
          :class="{ active: $route.path === '/orders' }"
          :aria-current="$route.path === '/orders' ? 'page' : undefined"
        >
          {{ t('nav.orders') }}
        </router-link>
        <router-link
          to="/spending"
          :class="{ active: $route.path === '/spending' }"
          :aria-current="$route.path === '/spending' ? 'page' : undefined"
        >
          {{ t('nav.finance') }}
        </router-link>
        <router-link
          to="/demand"
          :class="{ active: $route.path === '/demand' }"
          :aria-current="$route.path === '/demand' ? 'page' : undefined"
        >
          {{ t('nav.demandForecast') }}
        </router-link>
        <router-link
          to="/reports"
          :class="{ active: $route.path === '/reports' }"
          :aria-current="$route.path === '/reports' ? 'page' : undefined"
        >
          {{ t('nav.reports') }}
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <!-- Static info only: drop-down widgets placed here would be clipped
             by the sidebar's overflow-y -->
        <span>{{ t('nav.companyName') }} v1.0.0</span>
      </div>
    </aside>

    <div class="content">
      <header class="content-header">
        <div class="content-header-spacer"></div>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </header>

      <FilterBar />

      <main class="main-content">
        <router-view />
      </main>

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
  /* Colors — promoted from the app's existing slate/blue palette, not invented */
  --color-primary: #2563eb;
  --color-primary-light: #3b82f6;
  --color-primary-tint: #eff6ff;   /* active-nav / info tint used app-wide */
  --color-text: #0f172a;
  --color-text-body: #1e293b;
  --color-text-secondary: #64748b;
  --color-border: #e2e8f0;
  --color-surface: #ffffff;
  --color-bg: #f8fafc;
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #dc2626;

  /* Spacing — 4/8px-based scale */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Radii & shadows — the app's most common existing values */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 10px;
  --shadow-card: 0 4px 12px rgba(0, 0, 0, 0.06);
  --shadow-dropdown: 0 10px 25px rgba(0, 0, 0, 0.1);

  /* Layout. 240px sidebar fits the longest nav label in both locales
     ("Demand Forecast" / 需要予測) on one line with room for the accent bar. */
  --sidebar-width: 240px;
  --header-height: 56px;

  /* Z-index scale, mirroring the audited stacking order:
     sidebar (50) < sticky filter bar (90) < content header (100) < dropdown
     menus (1000, hardcoded in ProfileMenu/LanguageSwitcher) < modals (1000/2000).
     The content header must beat the filter bar: sticky + z-index makes the
     header a stacking context, so the dropdowns it hosts can only paint above
     the filter bar if the header itself does (the old 70px top-nav was 100). */
  --z-sidebar: 50;
  --z-sticky: 90;
  --z-header: 100;
  --z-dropdown: 1000;
  --z-modal: 2000;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-bg);
  color: var(--color-text-body);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Two-column app shell. Grid (not flex) so the sidebar column is rigid
   and wide content can't push it around. */
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}

.sidebar {
  /* sticky + 100vh keeps the sidebar pinned while the content column scrolls */
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  /* below the sticky header/filter bars so their dropdowns/shadows win overlaps */
  z-index: var(--z-sidebar);
}

.sidebar-brand {
  padding: var(--space-5) var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.sidebar-brand h1 {
  font-size: 1.125rem;
  font-weight: 700;
  letter-spacing: -0.025em;
  color: var(--color-text);
}

.sidebar-subtitle {
  font-size: 0.75rem;
  color: var(--color-text-secondary);
}

.sidebar-nav {
  flex: 1; /* pushes the footer to the bottom */
  padding: var(--space-4) var(--space-3);
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-sm);
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-text-secondary);
  text-decoration: none;
  /* transparent accent reserves the space so the active state doesn't shift text */
  border-left: 3px solid transparent;
  transition: background 0.15s, color 0.15s;
}

.sidebar-nav a:hover {
  background: var(--color-bg);
  color: var(--color-text);
}

.sidebar-nav a.active {
  background: var(--color-primary-tint);
  border-left-color: var(--color-primary);
  color: var(--color-primary);
  font-weight: 600;
}

.sidebar-nav a:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: -2px;
}

.sidebar-footer {
  padding: var(--space-4) var(--space-5);
  border-top: 1px solid var(--color-border);
  font-size: 0.75rem;
  color: var(--color-text-secondary);
}

.content {
  /* min-width: 0 lets this grid child shrink; without it, wide tables/charts
     force horizontal page overflow */
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.content-header {
  height: var(--header-height);
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding: 0 var(--space-8);
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  /* above the filter bar so the ProfileMenu/LanguageSwitcher dropdowns
     (children of this stacking context) are not painted under it */
  z-index: var(--z-header);
}

.content-header-spacer {
  flex: 1; /* pushes the widgets to the right edge */
}

.main-content {
  flex: 1;
  max-width: 1600px;
  width: 100%;
  margin: 0 auto;
  padding: var(--space-6) var(--space-8);
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: white;
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
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
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
