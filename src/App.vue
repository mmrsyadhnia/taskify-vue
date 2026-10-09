<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// ==========================================
// STATE APLIKASI UTAMA (TUGAS & NAVIGASI)
// ==========================================
const activeTab = ref('home')
const filter = ref('all')
const newTaskText = ref('')
const newTaskPriority = ref('medium')
const isNotificationOpen = ref(false)
const modalMessage = ref('')
const isModalOpen = ref(false)

// State Pengaturan Glassmorphism & UI
const settings = ref({
  blurIntensity: 16,
  ambientGlow: true,
  soundAlerts: true
})

// Data Tugas
const tasks = ref([
  { id: 1, title: 'Desain Komponen Spatial UI', priority: 'high', completed: false },
  { id: 2, title: 'Integrasi Tailwind CSS v3', priority: 'medium', completed: true },
  { id: 3, title: 'Review Kode Vue 3 SFC', priority: 'low', completed: false }
])

// ==========================================
// STATE DESKTOP & WIDGETS
// ==========================================
const currentTime = ref('')
const currentDate = ref('')

// Pomodoro Timer (25 Menit = 1500d)
const totalTimerSeconds = 1500
const timerSeconds = ref(1500)
const isTimerRunning = ref(false)
let timerInterval = null

// Quick Sticky Notes
const quickNotes = ref([
  { id: 1, text: 'Review UI/UX mockup spatial', color: 'from-amber-500/20 to-orange-500/20 border-amber-400/30' },
  { id: 2, text: 'Meeting sync tim jam 4 sore', color: 'from-pink-500/20 to-rose-500/20 border-pink-400/30' }
])
const newNoteText = ref('')

// Interactive Mouse Light Source (Spotlight)
const mouseX = ref(0)
const mouseY = ref(0)

const handleMouseMove = (e) => {
  mouseX.value = e.clientX
  mouseY.value = e.clientY
}

// ==========================================
// COMPUTED PROPERTIES
// ==========================================
const filteredTasks = computed(() => {
  if (filter.value === 'active') return tasks.value.filter(t => !t.completed)
  if (filter.value === 'completed') return tasks.value.filter(t => t.completed)
  return tasks.value
})

const completedCount = computed(() => tasks.value.filter(t => t.completed).length)
const pendingCount = computed(() => tasks.value.filter(t => !t.completed).length)
const totalCount = computed(() => tasks.value.length)
const completionRate = computed(() => totalCount.value ? Math.round((completedCount.value / totalCount.value) * 100) : 0)

const highPriorityCount = computed(() => tasks.value.filter(t => t.priority === 'high' && !t.completed).length)
const mediumPriorityCount = computed(() => tasks.value.filter(t => t.priority === 'medium' && !t.completed).length)
const lowPriorityCount = computed(() => tasks.value.filter(t => t.priority === 'low' && !t.completed).length)

const dashboardProgressOffset = computed(() => {
  const circumference = 2 * Math.PI * 36
  const progress = completionRate.value / 100
  return circumference * (1 - progress)
})

const formattedTimer = computed(() => {
  const m = Math.floor(timerSeconds.value / 60).toString().padStart(2, '0')
  const s = (timerSeconds.value % 60).toString().padStart(2, '0')
  return `${m}:${s}`
})

const timerProgressOffset = computed(() => {
  const circumference = 2 * Math.PI * 42
  const progress = (totalTimerSeconds - timerSeconds.value) / totalTimerSeconds
  return circumference * progress
})

// ==========================================
// METHODS & HANDLERS
// ==========================================
const updateClock = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })
  currentDate.value = now.toLocaleDateString('id-ID', { weekday: 'long', day: 'numeric', month: 'short' })
}

const toggleTimer = () => {
  if (isTimerRunning.value) {
    clearInterval(timerInterval)
    isTimerRunning.value = false
  } else {
    isTimerRunning.value = true
    timerInterval = setInterval(() => {
      if (timerSeconds.value > 0) {
        timerSeconds.value--
      } else {
        clearInterval(timerInterval)
        isTimerRunning.value = false
        showPopup('Waktu Fokus Selesai! 🎉')
      }
    }, 1000)
  }
}

const resetTimer = () => {
  clearInterval(timerInterval)
  isTimerRunning.value = false
  timerSeconds.value = totalTimerSeconds
}

const addTask = () => {
  if (!newTaskText.value.trim()) return showPopup('Tuliskan judul tugas terlebih dahulu!')
  tasks.value.unshift({ id: Date.now(), title: newTaskText.value.trim(), priority: newTaskPriority.value, completed: false })
  newTaskText.value = ''
  showPopup('Tugas baru ditambahkan!')
}

const toggleTask = (task) => {
  task.completed = !task.completed
  showPopup(task.completed ? 'Tugas Selesai! 🎉' : 'Tugas Dikembalikan')
}

const deleteTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id)
  showPopup('Tugas Dihapus')
}

const addNote = () => {
  if (!newNoteText.value.trim()) return
  const colors = [
    'from-amber-500/20 to-orange-500/20 border-amber-400/30',
    'from-cyan-500/20 to-blue-500/20 border-cyan-400/30',
    'from-emerald-500/20 to-teal-500/20 border-emerald-400/30',
    'from-purple-500/20 to-pink-500/20 border-purple-400/30'
  ]
  const randomColor = colors[Math.floor(Math.random() * colors.length)]
  quickNotes.value.push({ id: Date.now(), text: newNoteText.value.trim(), color: randomColor })
  newNoteText.value = ''
}

const deleteNote = (id) => {
  quickNotes.value = quickNotes.value.filter(n => n.id !== id)
}

const showPopup = (msg) => {
  modalMessage.value = msg
  isModalOpen.value = true
  setTimeout(() => { isModalOpen.value = false }, 2000)
}

const getPriorityBadge = (p) => {
  if (p === 'high') return 'bg-rose-500/20 text-rose-300 border-rose-500/30'
  if (p === 'medium') return 'bg-amber-500/20 text-amber-300 border-amber-500/30'
  return 'bg-emerald-500/20 text-emerald-300 border-emerald-500/30'
}

onMounted(() => {
  updateClock()
  const clockInterval = setInterval(updateClock, 1000)
  window.addEventListener('mousemove', handleMouseMove)
  onUnmounted(() => {
    clearInterval(clockInterval)
    window.removeEventListener('mousemove', handleMouseMove)
  })
})
</script>

<template>
  <div class="relative flex min-h-screen items-center justify-center bg-slate-950 p-4 font-sans text-slate-100 overflow-hidden select-none">
    
    <!-- BACKGROUND SPOTLIGHT & ORBS -->
    <div class="absolute inset-0 overflow-hidden pointer-events-none">
      <div 
        class="absolute -top-32 -left-32 w-[500px] h-[500px] rounded-full bg-gradient-to-tr from-indigo-600/40 via-purple-600/30 to-pink-500/30 blur-3xl animate-blob transition-opacity duration-500"
        :class="{ 'opacity-100': settings.ambientGlow, 'opacity-10': !settings.ambientGlow }"
      ></div>
      <div 
        class="absolute top-1/2 -right-32 w-[600px] h-[600px] rounded-full bg-gradient-to-br from-cyan-500/30 via-blue-600/40 to-emerald-500/20 blur-3xl animate-blob animation-delay-2000 transition-opacity duration-500"
        :class="{ 'opacity-100': settings.ambientGlow, 'opacity-10': !settings.ambientGlow }"
      ></div>

      <div 
        class="absolute w-[600px] h-[600px] rounded-full bg-cyan-500/10 blur-[120px] transition-transform duration-100 ease-out"
        :style="{ transform: `translate(${mouseX - 300}px, ${mouseY - 300}px)` }"
      ></div>
      <div class="absolute inset-0 bg-[radial-gradient(#ffffff_1px,transparent_1px)] [background-size:24px_24px] opacity-10"></div>
    </div>

    <!-- MAIN CONTAINER DESKTOP -->
    <div class="relative w-full max-w-6xl flex items-center justify-center lg:justify-between gap-6 z-10 py-6">
      
      <!-- SISI KIRI (WIDGETS) -->
      <aside class="hidden lg:flex flex-col gap-5 w-80 flex-shrink-0">
        <div class="p-5 rounded-3xl bg-white/5 border border-white/10 backdrop-blur-xl shadow-2xl hover:border-cyan-400/40 transition-all duration-300">
          <div class="flex items-center justify-between">
            <div>
              <p class="text-[10px] uppercase font-bold text-cyan-400 tracking-wider">Spatial Workspace</p>
              <h2 class="text-3xl font-black text-white tracking-tight">{{ currentTime }}</h2>
            </div>
            <div class="p-3 rounded-2xl bg-cyan-500/10 border border-cyan-400/20 text-cyan-300 font-bold text-xs shadow-inner">
              ☁️ 28°C
            </div>
          </div>
          <p class="text-xs text-slate-300 capitalize mt-2 font-medium">{{ currentDate }}</p>
        </div>

        <div class="p-6 rounded-3xl bg-gradient-to-b from-white/10 to-white/5 border border-white/15 backdrop-blur-xl shadow-2xl hover:border-indigo-400/40 transition-all duration-300 flex flex-col items-center">
          <div class="w-full flex justify-between items-center mb-4">
            <span class="text-xs font-bold text-white flex items-center gap-2">
              <span class="p-1 rounded-lg bg-indigo-500/20 text-indigo-300">⏱️</span> Focus Timer
            </span>
            <span class="text-[9px] px-2 py-0.5 rounded-full bg-indigo-500/20 text-indigo-300 border border-indigo-400/30 font-semibold">Pomodoro</span>
          </div>

          <div class="relative w-32 h-32 flex items-center justify-center my-1">
            <svg class="w-full h-full -rotate-90" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="42" stroke="currentColor" stroke-width="6" class="text-slate-800" fill="transparent" />
              <circle 
                cx="50" cy="50" r="42" stroke="currentColor" stroke-width="6" 
                class="text-cyan-400 transition-all duration-500 ease-linear" 
                fill="transparent" 
                stroke-dasharray="263.89" 
                :stroke-dashoffset="timerProgressOffset"
                stroke-linecap="round"
              />
            </svg>
            <span class="absolute text-2xl font-black text-white tracking-wider font-mono">{{ formattedTimer }}</span>
          </div>

          <div class="flex gap-2 w-full mt-4">
            <button 
              @click="toggleTimer"
              class="flex-1 py-2 rounded-xl text-xs font-bold bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-400 hover:to-blue-500 text-white shadow-lg active:scale-95 transition-all cursor-pointer border border-white/20"
            >
              {{ isTimerRunning ? 'PAUSE' : 'START' }}
            </button>
            <button 
              @click="resetTimer"
              class="px-3 py-2 rounded-xl text-xs font-semibold bg-white/10 text-slate-300 border border-white/15 hover:bg-white/20 active:scale-95 transition-all cursor-pointer"
            >
              RESET
            </button>
          </div>
        </div>
      </aside>

      <!-- PANEL UTAMA (DESKTOP GLASS PANEL, NON-HP) -->
      <main 
        class="relative w-full max-w-md h-[720px] bg-white/5 border border-white/15 rounded-3xl flex flex-col overflow-hidden transition-all duration-300 shadow-2xl backdrop-blur-2xl"
        :style="{ backdropFilter: `blur(${settings.blurIntensity}px)` }"
      >
        <!-- Top Navigation Bar -->
        <header class="flex items-center justify-between p-4 bg-white/5 border-b border-white/10">
          <div class="flex items-center space-x-3">
            <div class="w-9 h-9 rounded-2xl bg-gradient-to-tr from-cyan-400 to-indigo-500 border border-white/30 flex items-center justify-center shadow-md">
              <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
              </svg>
            </div>
            <div>
              <p class="text-[10px] tracking-wider uppercase font-semibold text-cyan-400">Spatial Task Manager</p>
              <h1 class="text-sm font-bold text-white">Kelola Tugas Hari Ini</h1>
            </div>
          </div>

          <!-- Tab Selector Header -->
          <div class="flex bg-white/5 p-1 rounded-xl border border-white/10">
            <button 
              @click="activeTab = 'home'"
              :class="activeTab === 'home' ? 'bg-cyan-500/30 text-cyan-300 font-bold' : 'text-slate-400'"
              class="px-2.5 py-1 rounded-lg text-xs transition-all cursor-pointer"
            >Tugas</button>
            <button 
              @click="activeTab = 'dashboard'"
              :class="activeTab === 'dashboard' ? 'bg-cyan-500/30 text-cyan-300 font-bold' : 'text-slate-400'"
              class="px-2.5 py-1 rounded-lg text-xs transition-all cursor-pointer"
            >Statistik</button>
            <button 
              @click="activeTab = 'settings'"
              :class="activeTab === 'settings' ? 'bg-cyan-500/30 text-cyan-300 font-bold' : 'text-slate-400'"
              class="px-2.5 py-1 rounded-lg text-xs transition-all cursor-pointer"
            >Settings</button>
          </div>
        </header>

        <!-- Area Konten Utama -->
        <div class="flex-1 overflow-y-auto p-4 space-y-4 no-scrollbar">
          <Transition name="fade" mode="out-in">
            
            <!-- TAB HOME -->
            <div v-if="activeTab === 'home'" key="home" class="space-y-4">
              <!-- Progres Card -->
              <div class="p-4 rounded-2xl bg-gradient-to-br from-indigo-900/40 via-purple-900/30 to-slate-900/50 border border-indigo-500/30 backdrop-blur-xl shadow-lg">
                <div class="flex justify-between items-center mb-2">
                  <span class="text-[10px] font-semibold uppercase tracking-wider text-indigo-300">Progres Harian</span>
                  <span class="text-xs font-bold text-cyan-300">{{ completionRate }}% Selesai</span>
                </div>
                <div class="w-full h-2 bg-slate-800 rounded-full overflow-hidden border border-white/10">
                  <div class="h-full bg-gradient-to-r from-cyan-400 to-indigo-500 transition-all duration-500" :style="{ width: `${completionRate}%` }"></div>
                </div>
              </div>

              <!-- Input Form -->
              <div class="p-3.5 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md space-y-2.5 hover:border-white/20 transition-all">
                <input 
                  type="text" v-model="newTaskText" @keyup.enter="addTask" placeholder="Ketik tugas baru di sini..." 
                  class="w-full px-3 py-2 text-xs rounded-xl bg-slate-900/60 border border-white/15 text-white placeholder-slate-400 focus:outline-none focus:border-cyan-400"
                />
                <div class="flex gap-2">
                  <select v-model="newTaskPriority" class="flex-1 px-3 py-1.5 text-xs rounded-xl bg-slate-900/60 border border-white/15 text-slate-200">
                    <option value="low" class="bg-slate-900">Prioritas Rendah</option>
                    <option value="medium" class="bg-slate-900">Prioritas Sedang</option>
                    <option value="high" class="bg-slate-900">Prioritas Tinggi</option>
                  </select>
                  <button @click="addTask" class="px-4 py-1.5 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 text-white font-medium text-xs border border-white/20 cursor-pointer">+ Tambah</button>
                </div>
              </div>

              <!-- Task List -->
              <div class="space-y-2.5">
                <div class="flex justify-between items-center px-1">
                  <h3 class="text-xs font-semibold text-slate-400 uppercase">Daftar Tugas</h3>
                  <div class="flex gap-1 bg-white/5 p-1 rounded-xl border border-white/10">
                    <button @click="filter = 'all'" :class="filter === 'all' ? 'bg-cyan-500/30 text-cyan-300' : 'text-slate-400'" class="px-2 py-0.5 text-[9px] rounded cursor-pointer">Semua</button>
                    <button @click="filter = 'active'" :class="filter === 'active' ? 'bg-cyan-500/30 text-cyan-300' : 'text-slate-400'" class="px-2 py-0.5 text-[9px] rounded cursor-pointer">Aktif</button>
                    <button @click="filter = 'completed'" :class="filter === 'completed' ? 'bg-cyan-500/30 text-cyan-300' : 'text-slate-400'" class="px-2 py-0.5 text-[9px] rounded cursor-pointer">Selesai</button>
                  </div>
                </div>

                <div v-for="task in filteredTasks" :key="task.id" class="flex items-center justify-between p-3 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md hover:bg-white/10 transition-all" :class="{ 'opacity-60': task.completed }">
                  <div class="flex items-center space-x-3 flex-1 min-w-0 pr-2">
                    <button @click="toggleTask(task)" :class="task.completed ? 'bg-cyan-500 border-cyan-400 text-slate-950' : 'border-white/30'" class="w-5 h-5 rounded-lg border flex items-center justify-center cursor-pointer">
                      <svg v-if="task.completed" class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" /></svg>
                    </button>
                    <span :class="{ 'line-through text-slate-400': task.completed }" class="text-xs text-slate-100 truncate">{{ task.title }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <span class="text-[9px] px-2 py-0.5 rounded-full border capitalize font-semibold" :class="getPriorityBadge(task.priority)">{{ task.priority }}</span>
                    <button @click="deleteTask(task.id)" class="text-slate-400 hover:text-rose-400 cursor-pointer px-1">✕</button>
                  </div>
                </div>
              </div>
            </div>

            <!-- TAB METRICS / STATISTIK -->
            <div v-else-if="activeTab === 'dashboard'" key="dashboard" class="space-y-4 pb-2">
              <h2 class="text-xs font-bold text-white px-1">Ringkasan Produktivitas</h2>

              <div class="p-4 rounded-2xl bg-gradient-to-br from-indigo-900/40 via-purple-900/30 to-slate-900/50 border border-indigo-500/30 backdrop-blur-xl shadow-lg flex items-center justify-between">
                <div>
                  <p class="text-[10px] font-bold text-cyan-400 uppercase tracking-wider">Status Tugas</p>
                  <h3 class="text-base font-extrabold text-white mt-0.5">{{ completionRate }}% Selesai</h3>
                  <p class="text-[9px] text-slate-300 mt-1">{{ completedCount }} dari {{ totalCount }} tugas diselesaikan</p>
                </div>

                <div class="relative w-20 h-20 flex items-center justify-center flex-shrink-0">
                  <svg class="w-full h-full -rotate-90" viewBox="0 0 80 80">
                    <circle cx="40" cy="40" r="32" stroke="currentColor" stroke-width="6" class="text-slate-800" fill="transparent" />
                    <circle 
                      cx="40" cy="40" r="32" stroke="currentColor" stroke-width="6" 
                      class="text-cyan-400 transition-all duration-500" 
                      fill="transparent" 
                      stroke-dasharray="201.06" 
                      :stroke-dashoffset="dashboardProgressOffset"
                      stroke-linecap="round"
                    />
                  </svg>
                  <span class="absolute text-xs font-bold text-cyan-300">{{ completionRate }}%</span>
                </div>
              </div>

              <div class="grid grid-cols-2 gap-2.5">
                <div class="p-3 rounded-2xl bg-emerald-500/10 border border-emerald-500/20 backdrop-blur-md">
                  <p class="text-[9px] font-medium text-emerald-300 uppercase">Tugas Selesai</p>
                  <p class="text-xl font-black text-emerald-400 mt-1">{{ completedCount }}</p>
                </div>
                <div class="p-3 rounded-2xl bg-amber-500/10 border border-amber-500/20 backdrop-blur-md">
                  <p class="text-[9px] font-medium text-amber-300 uppercase">Tugas Pending</p>
                  <p class="text-xl font-black text-amber-400 mt-1">{{ pendingCount }}</p>
                </div>
                <div class="p-3 rounded-2xl bg-rose-500/10 border border-rose-500/20 backdrop-blur-md">
                  <p class="text-[9px] font-medium text-rose-300 uppercase">Prioritas Tinggi</p>
                  <p class="text-xl font-black text-rose-400 mt-1">{{ highPriorityCount }}</p>
                </div>
                <div class="p-3 rounded-2xl bg-cyan-500/10 border border-cyan-500/20 backdrop-blur-md">
                  <p class="text-[9px] font-medium text-cyan-300 uppercase">Total Tugas</p>
                  <p class="text-xl font-black text-cyan-400 mt-1">{{ totalCount }}</p>
                </div>
              </div>

              <div class="p-3.5 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md space-y-2">
                <p class="text-[10px] font-bold text-slate-300 uppercase tracking-wider">Breakdown Prioritas Pending</p>
                
                <div class="space-y-1.5 text-[10px]">
                  <div class="flex justify-between text-slate-300">
                    <span>Tinggi</span>
                    <span class="font-bold text-rose-400">{{ highPriorityCount }} Item</span>
                  </div>
                  <div class="w-full h-1 bg-slate-800 rounded-full overflow-hidden">
                    <div class="h-full bg-rose-500" :style="{ width: `${totalCount ? (highPriorityCount/totalCount)*100 : 0}%` }"></div>
                  </div>

                  <div class="flex justify-between text-slate-300 pt-1">
                    <span>Sedang</span>
                    <span class="font-bold text-amber-400">{{ mediumPriorityCount }} Item</span>
                  </div>
                  <div class="w-full h-1 bg-slate-800 rounded-full overflow-hidden">
                    <div class="h-full bg-amber-500" :style="{ width: `${totalCount ? (mediumPriorityCount/totalCount)*100 : 0}%` }"></div>
                  </div>

                  <div class="flex justify-between text-slate-300 pt-1">
                    <span>Rendah</span>
                    <span class="font-bold text-emerald-400">{{ lowPriorityCount }} Item</span>
                  </div>
                  <div class="w-full h-1 bg-slate-800 rounded-full overflow-hidden">
                    <div class="h-full bg-emerald-500" :style="{ width: `${totalCount ? (lowPriorityCount/totalCount)*100 : 0}%` }"></div>
                  </div>
                </div>
              </div>
            </div>

            <!-- TAB SETTINGS -->
            <div v-else-if="activeTab === 'settings'" key="settings" class="space-y-3.5 pb-2">
              <h2 class="text-xs font-bold text-white px-1">Pengaturan Spatial UI</h2>

              <div class="p-3.5 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md space-y-3">
                <p class="text-[10px] font-bold text-cyan-400 uppercase tracking-wider">Kustomisasi Tampilan</p>
                <div class="space-y-1">
                  <div class="flex justify-between text-xs">
                    <span class="text-slate-200 font-medium">Intensitas Blur</span>
                    <span class="text-cyan-400 font-bold">{{ settings.blurIntensity }}px</span>
                  </div>
                  <input type="range" min="4" max="32" v-model.number="settings.blurIntensity" class="w-full accent-cyan-400 bg-slate-800 rounded-lg h-1.5 cursor-pointer" />
                </div>

                <hr class="border-white/10" />

                <div class="flex items-center justify-between">
                  <div>
                    <p class="text-xs font-medium text-slate-200">Ambient Glow</p>
                    <p class="text-[9px] text-slate-400">Pendaran cahaya latar belakang</p>
                  </div>
                  <button 
                    @click="settings.ambientGlow = !settings.ambientGlow"
                    class="w-9 h-5 p-0.5 rounded-full flex items-center transition-colors duration-300 border border-white/20 cursor-pointer"
                    :class="settings.ambientGlow ? 'bg-cyan-500 justify-end' : 'bg-slate-700 justify-start'"
                  >
                    <div class="w-4 h-4 rounded-full bg-white shadow-md"></div>
                  </button>
                </div>
              </div>

              <div class="p-3.5 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md space-y-3">
                <p class="text-[10px] font-bold text-cyan-400 uppercase tracking-wider">Notifikasi & Suara</p>
                <div class="flex items-center justify-between">
                  <div>
                    <p class="text-xs font-medium text-slate-200">Suara Pengingat</p>
                    <p class="text-[9px] text-slate-400">Bunyi saat timer fokus selesai</p>
                  </div>
                  <button 
                    @click="settings.soundAlerts = !settings.soundAlerts; showPopup(settings.soundAlerts ? 'Suara pengingat aktif' : 'Suara dimatikan')"
                    class="w-9 h-5 p-0.5 rounded-full flex items-center transition-colors duration-300 border border-white/20 cursor-pointer"
                    :class="settings.soundAlerts ? 'bg-cyan-500 justify-end' : 'bg-slate-700 justify-start'"
                  >
                    <div class="w-4 h-4 rounded-full bg-white shadow-md"></div>
                  </button>
                </div>
              </div>

              <div class="p-3.5 rounded-2xl bg-white/5 border border-white/10 backdrop-blur-md space-y-2">
                <p class="text-[10px] font-bold text-cyan-400 uppercase tracking-wider">Manajemen Data</p>
                <div class="flex gap-2 pt-1">
                  <button @click="showPopup('Data berhasil disinkronkan!')" class="flex-1 py-1.5 rounded-xl bg-white/10 hover:bg-white/20 border border-white/15 text-[10px] font-semibold text-slate-200 transition-all cursor-pointer">
                    Sync Cloud
                  </button>
                  <button @click="tasks = []; showPopup('Semua tugas dibersihkan!')" class="flex-1 py-1.5 rounded-xl bg-rose-500/20 hover:bg-rose-500/30 border border-rose-500/30 text-[10px] font-semibold text-rose-300 transition-all cursor-pointer">
                    Reset Tugas
                  </button>
                </div>
              </div>
            </div>

          </Transition>
        </div>
      </main>

      <!-- SISI KANAN (WIDGETS) -->
      <aside class="hidden lg:flex flex-col gap-5 w-80 flex-shrink-0">
        <div class="p-6 rounded-3xl bg-gradient-to-b from-white/10 to-white/5 border border-white/15 backdrop-blur-xl shadow-2xl hover:border-cyan-400/40 transition-all duration-300 space-y-3">
          <div class="flex justify-between items-center">
            <span class="text-xs font-bold text-white flex items-center gap-2">
              <span class="p-1 rounded-lg bg-cyan-500/20 text-cyan-300">📊</span> Produktivitas
            </span>
            <span class="text-[9px] px-2 py-0.5 rounded-full bg-emerald-500/20 text-emerald-300 border border-emerald-400/30 font-semibold">+18% WEEKLIES</span>
          </div>

          <div class="relative h-24 w-full pt-2">
            <svg class="w-full h-full overflow-visible" viewBox="0 0 200 80" fill="none">
              <defs>
                <linearGradient id="waveGradient" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="0%" stop-color="#06b6d4" stop-opacity="0.4" />
                  <stop offset="100%" stop-color="#06b6d4" stop-opacity="0" />
                </linearGradient>
              </defs>
              <path d="M 0 60 Q 40 20 80 45 T 160 15 T 200 30 L 200 80 L 0 80 Z" fill="url(#waveGradient)" />
              <path d="M 0 60 Q 40 20 80 45 T 160 15 T 200 30" stroke="#06b6d4" stroke-width="3" stroke-linecap="round" fill="none" />
              <circle cx="160" cy="15" r="4" class="fill-cyan-300 animate-ping" />
              <circle cx="160" cy="15" r="4" class="fill-cyan-400" />
            </svg>
          </div>

          <div class="flex justify-between text-[9px] text-slate-400 px-1 font-semibold">
            <span>SEN</span><span>SEL</span><span>RAB</span><span>KAM</span><span>JUM</span><span>SAB</span>
          </div>
        </div>

        <div class="p-6 rounded-3xl bg-white/5 border border-white/10 backdrop-blur-xl shadow-2xl space-y-3 hover:border-pink-400/40 transition-all duration-300">
          <div class="flex justify-between items-center">
            <span class="text-xs font-bold text-white flex items-center gap-2">
              <span class="p-1 rounded-lg bg-pink-500/20 text-pink-300">📝</span> Sticky Notes
            </span>
          </div>

          <div class="flex gap-2">
            <input 
              type="text" v-model="newNoteText" @keyup.enter="addNote" placeholder="Tulis catatan cepat..." 
              class="flex-1 px-3 py-1.5 text-xs rounded-xl bg-slate-900/60 border border-white/15 text-white placeholder-slate-400 focus:outline-none focus:border-cyan-400"
            />
            <button @click="addNote" class="px-3 py-1.5 rounded-xl bg-cyan-500 text-slate-950 font-extrabold text-xs cursor-pointer shadow-md active:scale-95 transition-all">+</button>
          </div>

          <div class="space-y-2 max-h-36 overflow-y-auto no-scrollbar pt-1">
            <div 
              v-for="note in quickNotes" 
              :key="note.id" 
              class="flex justify-between items-center p-3 rounded-2xl bg-gradient-to-r border backdrop-blur-md text-xs text-slate-100 shadow-sm"
              :class="note.color"
            >
              <span class="truncate pr-2 font-medium">{{ note.text }}</span>
              <button @click="deleteNote(note.id)" class="text-slate-400 hover:text-rose-400 text-xs font-bold cursor-pointer">✕</button>
            </div>
          </div>
        </div>
      </aside>

    </div>

    <!-- Toast Modal -->
    <Transition name="fade">
      <div v-if="isModalOpen" class="fixed bottom-8 p-3 rounded-2xl bg-cyan-500/90 text-slate-950 font-bold text-xs text-center shadow-2xl z-50 backdrop-blur-md border border-white/40">
        {{ modalMessage }}
      </div>
    </Transition>

  </div>
</template>

<style scoped>
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(4px); }

@keyframes blob {
  0% { transform: translate(0px, 0px) scale(1); }
  33% { transform: translate(30px, -50px) scale(1.1); }
  66% { transform: translate(-20px, 20px) scale(0.9); }
  100% { transform: translate(0px, 0px) scale(1); }
}

.animate-blob {
  animation: blob 8s infinite ease-in-out;
}

.animation-delay-2000 {
  animation-delay: 2s;
}
</style>