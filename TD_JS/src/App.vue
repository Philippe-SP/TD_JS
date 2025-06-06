<template>
  <main>
    <div>
      <h1>Vue.js Example</h1>
      <hr />
      <button @click="fetchUsers">Fetch Users</button>
      <button @click="toggleSort">{{ sortButtonLabel }}</button>
      <input v-model="searchTerm" type="text" placeholder="Rechercher par nom..." />
      <select v-model="genderFilter">
        <option value="All">Tous</option>
        <option value="Male">Hommes</option>
        <option value="Female">Femmes</option>
      </select>
    </div>

    <table>
      <thead>
        <tr>
          <th>Photo</th>
          <th>Nom</th>
          <th>Email</th>
          <th>Tel</th>
          <th>Age</th>
          <th>Genre</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in displayedUsers" :key="user.login.uuid">
          <td><img :src="user.picture.thumbnail" /></td>
          <td>{{ user.name.first }} {{ user.name.last }}</td>
          <td>{{ user.email }}</td>
          <td>{{ user.cell }}</td>
          <td>{{ user.dob.age }}</td>
          <td v-html="user.gender === 'male' ? '&#9794;' : '&#9792;'"></td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue'

const usersData = ref([])
const genderFilter = ref('All')
const searchTerm = ref('')
const sortState = ref('no-sort') // 'no-sort' | 'ascended' | 'descended'

const fetchUsers = async () => {
  const response = await fetch('https://randomuser.me/api/?results=20')
  const data = await response.json()
  usersData.value = data.results
}

// Gère le tri et filtre combinés
const displayedUsers = computed(() => {
  let filtered = usersData.value

  // Filtrer par genre
  if (genderFilter.value === 'Male') {
    filtered = filtered.filter(u => u.gender === 'male')
  } else if (genderFilter.value === 'Female') {
    filtered = filtered.filter(u => u.gender === 'female')
  }

  // Filtrer par nom
  const term = searchTerm.value.toLowerCase()
  filtered = filtered.filter(u => {
    const fullName = `${u.name.first} ${u.name.last}`.toLowerCase()
    return fullName.includes(term)
  })

  // Trier selon l'âge
  if (sortState.value === 'ascended') {
    return [...filtered].sort((a, b) => a.dob.age - b.dob.age)
  }
  if (sortState.value === 'descended') {
    return [...filtered].sort((a, b) => b.dob.age - a.dob.age)
  }

  return filtered
})

// Gère le texte du bouton de tri
const sortButtonLabel = computed(() => {
  if (sortState.value === 'ascended') return 'Sort users ↑'
  if (sortState.value === 'descended') return 'Sort users ↓'
  return 'Sort users'
})

// Changement d'état du tri
const toggleSort = () => {
  if (sortState.value === 'no-sort') sortState.value = 'ascended'
  else if (sortState.value === 'ascended') sortState.value = 'descended'
  else sortState.value = 'no-sort'
}
</script>

<style>
table img {
  border-radius: 50%;
}
</style>
