<script setup>
import { ref } from 'vue'

// State untuk menyimpan daftar tugas
const tasks = ref([])

// State untuk menampung nilai input dari user
const inputTask = ref('')

// Action untuk menambahkan tugas baru
const addTask = () => {
  // Validasi agar tidak memasukkan tugas kosong
  if (!inputTask.value.trim()) return

  const newTask = {
    id: new Date().getTime(),
    title: inputTask.value,
    status: false
  }

  tasks.value.push(newTask)
  
  // Reset input setelah tugas ditambahkan
  inputTask.value = ''
}

// Action untuk menghapus tugas berdasarkan id
const deleteTask = (id) => {

  const task = tasks.value.find(item => item.id == id)
  if (!task) {
    alert("Task tidak ditemukan!")
    return
  }
  const confirmation = confirm("apakah anda yakin ingin menghapus?")

  if (confirmation) {
     tasks.value = tasks.value.filter(item => item.id =! id)
  }


}
</script>

<template>
  <section class="mx-auto mt-12 max-w-xl px-4">
    <div class="rounded-xl bg-auto-600 p-6 shadow">
      <h1 class="mb-2 text-center text-2xl font-bold">Taskify Vue</h1>
      
      <form 
        @submit.prevent="addTask"
        class="flex gap-3 mb-6"
      >
        <input
          v-model="inputTask"
          type="text"
          placeholder="masukan tugas baru"
          aria-label="Tugas baru"
          class="min-w-0 flex-1 rounded-lg border px-4 py-3 focus:border-indigo-600 focus:outline-none"
        />
        <button
          type="submit"
          class="rounded-lg bg-red-600 px-6 py-3 font-semibold text-white transition hover:bg-indigo-700"
        >
          Tambah
        </button>
      </form>

       <!-- daftar tugas -->
      <ul class="mt-6 space-y-4" v-auto-animate>

        <li 
          v-for="task in tasks" 
          :key="task.id" 
          class="flex items-center justify-between rounded-lg bg-gray-50 border p-3"
        >
          <div class="flex items-center gap-3">
            <!-- Checkbox dihubungkan dengan v-model ke task.status -->
            <input 
              type="checkbox" 
              v-model="task.status"
              class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
            />
            <!-- Teks tugas otomatis tercoret jika task.status bernilai true -->
            <span :class="{ 'line-through text-gray-700': task.status }" class="text-gray-700">
              {{ task.title }}
            </span>
          </div>

          <!-- Tombol Hapus dengan event @click -->
          <button 
            @click="deleteTask(task.id)" 
            class="text-sm font-medium text-red-500 hover:text-red-700"
          >
            Hapus
          </button>
        </li>
      </ul>
    </div>
  </section>
</template>