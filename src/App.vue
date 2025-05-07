<template>
  <div class="p-4">
    <h1 class="text-2xl mb-4">CSV Processor</h1>

    <input type="file" accept=".csv" @change="handleFileUpload"/>

    <div v-if="results.length" class="mt-4">
      <h2 class="text-xl mb-2">Processed Results</h2>
      <ul>
        <li v-for="(item, index) in results" :key="index">{{ item }}</li>
      </ul>
      <button class="mt-4 px-4 py-2 bg-blue-500 text-white rounded" @click="downloadCSV">
        Download Results
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import Papa from 'papaparse'
import {ref} from 'vue'

const results = ref([])

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  Papa.parse(file, {
    header: false,
    skipEmptyLines: true,
    complete: async (result) => {
      const data = result.data
      const processed = []

      for (const row of data) {
        const value = row[0]
        const response = await fetchUTMBScore(value)

        processed.push(response)
      }

      results.value = processed
    }
  })
}

const fetchUTMBScore = async (runnerName) => {
  const url = `https://api.utmb.world/search/runners?` +
      `search=${encodeURIComponent(runnerName)}`

  // await new Promise(resolve => setTimeout(resolve, 300))

  const response = await fetch(url)
  const data = await response.json()

  return data.runners?.[0]?.ip || '0'
}

const downloadCSV = () => {
  const blob = new Blob([results.value.join('\n')], {type: 'text/csv;charset=utf-8;'})
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.setAttribute('download', 'results.csv')
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}
</script>

<style scoped>
body {
  font-family: sans-serif;
}
</style>
