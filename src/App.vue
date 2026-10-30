<script setup>
import { ref, computed, watch } from 'vue'

const STORAGE_KEY = 'taskify-vue-tasks'

const loadTasks = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY)
    return saved ? JSON.parse(saved) : []
  } catch {
    return []
  }
}

const tasks = ref(loadTasks())
const inputTask = ref('')
const activeFilter = ref('all')
const searchQuery = ref('')

watch(
  tasks,
  (value) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
  },
  { deep: true }
)

const totalTasks = computed(() => tasks.value.length)

const completedTasks = computed(
  () => tasks.value.filter((task) => task.status).length
)

const pendingTasks = computed(
  () => tasks.value.filter((task) => !task.status).length
)

const progress = computed(() => {
  if (totalTasks.value === 0) return 0

  return Math.round(
    (completedTasks.value / totalTasks.value) * 100
  )
})

const filteredTasks = computed(() => {
  let result = tasks.value

  if (activeFilter.value === 'active') {
    result = result.filter((task) => !task.status)
  }

  if (activeFilter.value === 'completed') {
    result = result.filter((task) => task.status)
  }

  const keyword = searchQuery.value.trim().toLowerCase()

  if (keyword) {
    result = result.filter((task) =>
      task.title.toLowerCase().includes(keyword)
    )
  }

  return result
})

const addTask = () => {
  const title = inputTask.value.trim()

  if (!title) return

  tasks.value.unshift({
    id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
    title,
    status: false,
    createdAt: new Date().toISOString()
  })

  inputTask.value = ''
}

const deleteTask = (id) => {
  tasks.value = tasks.value.filter((task) => task.id !== id)
}

const clearCompleted = () => {
  tasks.value = tasks.value.filter((task) => !task.status)
}

const formatDate = () => {
  return new Intl.DateTimeFormat('id-ID', {
    weekday: 'long',
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(new Date())
}
</script>

<template>
  <main class="app-shell">
    <div class="ambient ambient-one"></div>
    <div class="ambient ambient-two"></div>

    <!-- NAVBAR -->
    <header class="navbar">
      <a href="#" class="brand">
        <span class="brand-icon">
          <svg viewBox="0 0 24 24" fill="none">
            <path
              d="M5 12.5L9.2 16.5L19 6.5"
              stroke="currentColor"
              stroke-width="2.8"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
            <path
              d="M19 12V18C19 19.1 18.1 20 17 20H6C4.9 20 4 19.1 4 18V7C4 5.9 4.9 5 6 5H14"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </span>

        <span class="brand-name">
          Taskify<span class="brand-accent"> Vue</span>
        </span>
      </a>

      <nav class="nav-links">
        <a href="#home" class="nav-link active">Home</a>
        <a href="#tasks" class="nav-link">Tasks</a>
        <a href="#about" class="nav-link">About</a>
      </nav>

      <div class="nav-status">
        <span class="status-dot"></span>
        <span>Workspace Active</span>
      </div>
    </header>

    <!-- HERO -->
    <section id="home" class="hero">
      <div class="hero-copy">
        <div class="eyebrow">
          <span class="eyebrow-line"></span>
          YOUR PERSONAL PRODUCTIVITY SPACE
        </div>

        <h1>
          Taskify<span class="gradient-text"> Vue.</span>
        </h1>

        <p class="hero-description">
          Atur tugasmu, fokus pada prioritasmu,<br class="desktop-break" />
          dan capai lebih banyak setiap hari.
        </p>

        <div class="hero-meta">
          <span class="meta-icon">✦</span>
          {{ formatDate() }}
        </div>
      </div>

      <div class="hero-visual" aria-hidden="true">
        <div class="orbit orbit-one"></div>
        <div class="orbit orbit-two"></div>

        <div class="visual-card">
          <svg viewBox="0 0 100 100" fill="none">
            <path
              d="M23 52L42 70L79 30"
              stroke="currentColor"
              stroke-width="9"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </div>

        <span class="visual-spark spark-one">✦</span>
        <span class="visual-spark spark-two">✧</span>
        <span class="visual-spark spark-three">+</span>
      </div>
    </section>

    <!-- STATISTICS -->
    <section class="stats-grid">
      <article class="stat-card">
        <div class="stat-top">
          <span class="stat-label">Total Tugas</span>
          <span class="stat-icon green-icon">▤</span>
        </div>

        <div class="stat-value">{{ totalTasks }}</div>
        <div class="stat-caption">Semua tugasmu</div>

        <div class="stat-bottom-line"></div>
      </article>

      <article class="stat-card">
        <div class="stat-top">
          <span class="stat-label">Belum Selesai</span>
          <span class="stat-icon orange-icon">◷</span>
        </div>

        <div class="stat-value">{{ pendingTasks }}</div>
        <div class="stat-caption">Tetap semangat!</div>

        <div class="stat-bottom-line orange-line"></div>
      </article>

      <article class="stat-card">
        <div class="stat-top">
          <span class="stat-label">Selesai</span>
          <span class="stat-icon green-icon">✓</span>
        </div>

        <div class="stat-value">{{ completedTasks }}</div>
        <div class="stat-caption">Satu langkah lebih maju</div>

        <div class="stat-bottom-line"></div>
      </article>
    </section>

    <!-- TASK MANAGER -->
    <section id="tasks" class="task-panel">
      <div class="panel-header">
        <div>
          <div class="section-kicker">
            <span class="live-dot"></span>
            YOUR WORKSPACE
          </div>

          <h2>Daftar Tugas<span class="title-period">.</span></h2>

          <p class="panel-description">
            Rencanakan hari ini, selesaikan satu per satu.
          </p>
        </div>

        <div class="progress-ring">
          <svg viewBox="0 0 80 80">
            <circle
              class="progress-track"
              cx="40"
              cy="40"
              r="31"
            />

            <circle
              class="progress-value"
              cx="40"
              cy="40"
              r="31"
              :style="{
                strokeDasharray: `${progress * 1.948} 194.8`
              }"
            />
          </svg>

          <span>{{ progress }}%</span>
        </div>
      </div>

      <!-- INPUT -->
      <form class="task-form" @submit.prevent="addTask">
        <span class="input-icon">✎</span>

        <input
          v-model="inputTask"
          type="text"
          maxlength="200"
          placeholder="Apa yang ingin kamu selesaikan hari ini?"
          aria-label="Masukkan tugas baru"
        />

        <button type="submit" class="add-button">
          <span class="plus-icon">+</span>
          <span>Tambah Tugas</span>
        </button>
      </form>

      <!-- TOOLBAR -->
      <div class="task-toolbar">
        <div class="filter-group">
          <button
            type="button"
            class="filter-button"
            :class="{ selected: activeFilter === 'all' }"
            @click="activeFilter = 'all'"
          >
            Semua
            <span class="filter-count">{{ totalTasks }}</span>
          </button>

          <button
            type="button"
            class="filter-button"
            :class="{ selected: activeFilter === 'active' }"
            @click="activeFilter = 'active'"
          >
            Aktif
          </button>

          <button
            type="button"
            class="filter-button"
            :class="{ selected: activeFilter === 'completed' }"
            @click="activeFilter = 'completed'"
          >
            Selesai
          </button>
        </div>

        <label class="search-box">
          <span>⌕</span>

          <input
            v-model="searchQuery"
            type="search"
            placeholder="Cari tugas..."
            aria-label="Cari tugas"
          />
        </label>
      </div>

      <!-- TASK LIST -->
      <div class="task-list">
        <TransitionGroup name="task" tag="div" class="task-items">
          <article
            v-for="(task, index) in filteredTasks"
            :key="task.id"
            class="task-item"
            :class="{ 'task-completed': task.status }"
          >
            <button
              type="button"
              class="check-button"
              :class="{ checked: task.status }"
              :aria-label="
                task.status
                  ? 'Tandai belum selesai'
                  : 'Tandai selesai'
              "
              :aria-pressed="task.status"
              @click="task.status = !task.status"
            >
              <svg
                v-if="task.status"
                viewBox="0 0 24 24"
                fill="none"
              >
                <path
                  d="M5 12.5L10 17L19 7"
                  stroke="currentColor"
                  stroke-width="2.5"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </button>

            <div class="task-content">
              <span class="task-title">{{ task.title }}</span>

              <span class="task-subtitle">
                {{ task.status ? 'Berhasil diselesaikan' : 'Tugas' }}
              </span>
            </div>

            <span
              class="task-badge"
              :class="task.status ? 'badge-done' : 'badge-pending'"
            >
              {{ task.status ? 'Selesai' : 'Dalam proses' }}
            </span>

            <button
              type="button"
              class="delete-button"
              aria-label="Hapus tugas"
              title="Hapus tugas"
              @click="deleteTask(task.id)"
            >
              <svg viewBox="0 0 24 24" fill="none">
                <path
                  d="M4 7H20M10 11V17M14 11V17M5.5 7L6.5 20H17.5L18.5 7M9 7V4H15V7"
                  stroke="currentColor"
                  stroke-width="1.7"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </button>
          </article>
        </TransitionGroup>

        <div
          v-if="filteredTasks.length === 0"
          class="empty-state"
        >
          <div class="empty-icon">
            <svg viewBox="0 0 48 48" fill="none">
              <rect
                x="10"
                y="8"
                width="28"
                height="34"
                rx="6"
                stroke="currentColor"
                stroke-width="1.8"
              />
              <path
                d="M17 19H31M17 25H31M17 31H26"
                stroke="currentColor"
                stroke-width="1.8"
                stroke-linecap="round"
              />
            </svg>
          </div>

          <h3>
            {{
              searchQuery
                ? 'Tugas tidak ditemukan'
                : activeFilter === 'completed'
                  ? 'Belum ada tugas selesai'
                  : activeFilter === 'active'
                    ? 'Semua tugas sudah selesai!'
                    : 'Mulai dari satu tugas kecil'
            }}
          </h3>

          <p>
            {{
              searchQuery
                ? 'Coba gunakan kata kunci yang berbeda.'
                : 'Tambahkan tugas pertamamu dan mulai produktif hari ini.'
            }}
          </p>
        </div>
      </div>

      <!-- FOOTER PANEL -->
      <div class="panel-footer">
        <div class="footer-message">
          <span class="footer-leaf">✳</span>
          <span>Pelan-pelan, yang penting terus berjalan.</span>
        </div>

        <button
          v-if="completedTasks > 0"
          type="button"
          class="clear-button"
          @click="clearCompleted"
        >
          Bersihkan tugas selesai
        </button>
      </div>
    </section>

    <!-- PAGE FOOTER -->
    <footer id="about" class="page-footer">
      <div class="footer-brand">
        <span class="footer-brand-dot"></span>
        Taskify Vue
      </div>

      <p>Built for focus. Designed for progress.</p>

      <span class="version-label">VERSION 1.0.0</span>
    </footer>
  </main>
</template>