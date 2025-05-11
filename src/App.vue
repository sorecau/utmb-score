<template>
  <div class="column">
    <div class="text-center mb-8">
      <p class="text-lg text-gray-100 mt-2">
        Upload a file containing a list of runner names. Each line in the file should contain one runner's full name.
      </p>
      <p class="text-sm text-gray-400 mt-1">
        Example:
        <br>Jim Walmsley
        <br>Remi Bonet
        <br>Toni McCann
      </p>

      <input
          type="file"
          accept="text/plain,.csv"
          @change="handleFileUpload"
          class="block mx-auto mt-4 text-sm text-gray-200 file:mr-4 file:py-2 file:px-4 file:rounded file:border-0 file:bg-gray-100 file:text-gray-700 hover:file:bg-gray-200"
      />
    </div>
  </div>

  <template v-if="results.length">
    <div class="column flex flex-col items-center min-h-[400px]">
      <h2 class="text-2xl font-semibold text-gray-100 mb-4">Processed Results</h2>

      <div class="overflow-x-auto">
        <table class="min-w-full text-sm text-left border border-gray-200">
          <thead class="bg-gray-100 text-gray-600 uppercase tracking-wider">
          <tr>
            <th class="px-4 py-3 border-b min-w-[150px]">Name</th>
            <th class="px-4 py-3 border-b min-w-[150px]">Score</th>
            <th class="px-4 py-3 border-b min-w-[150px]">Nationality</th>
            <th class="px-4 py-3 border-b min-w-[150px]">Gender</th>
          </tr>
          </thead>
          <tbody class="divide-y divide-gray-200">
          <tr
              v-for="(item, index) in results"
              :key="index"
          >
            <td class="px-4 py-3">{{ item.fullname }}</td>
            <td class="px-4 py-3">{{ item.ip }}</td>
            <td class="px-4 py-3">{{ item.nationality }}</td>
            <td class="px-4 py-3">{{ item.sex === 'H' ? 'Men' : item.sex === 'F' ? 'Women' : '' }}</td>
          </tr>
          </tbody>
        </table>
      </div>

      <button
          class="mt-6 inline-flex items-center px-5 py-2 bg-gray-600 hover:bg-gray-700 text-white text-sm font-medium rounded-lg shadow"
          @click="downloadCSV"
      >
        Download Results
      </button>
    </div>
  </template>
  <template v-else>
    <div class="column flex flex-col items-center min-h-[400px]">
      <svg
          class="w-24 h-24 mx-auto mb-4"
          fill="none"
          stroke="currentColor"
          stroke-width="1.5"
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
      >
        <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M12 15v2m0 4h.01M19.5 12A7.5 7.5 0 005.25 6.75a.75.75 0 00-.75.75v.75a.75.75 0 00.75.75A5.25 5.25 0 0112 12.75v0a5.25 5.25 0 016.75-5.25.75.75 0 00.75-.75V7.5a.75.75 0 00-.75-.75A7.5 7.5 0 0019.5 12z"
        ></path>
      </svg>
      <p class="text-lg font-medium">No results yet</p>
      <p class="text-sm">Upload a text file to see scores.</p>
    </div>
  </template>
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
    complete: async ({ data }) => {
      const runners = data.map(row => ({ name: row[0] }));

      results.value = await fetchUTMBScore(runners);
    }
  });
}

const fetchUTMBScore = async (runners) => {
  const url = "https://utmb-score-ws.onrender.com/api/score"

  await new Promise(resolve => setTimeout(resolve, 300))

  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },

    body: JSON.stringify(runners)
  })

  return await response.json()
}

const downloadCSV = () => {
  const headers = ['Name', 'Score', 'Nationality', 'Gender'];
  const csvRows = [
    headers.join(';'),
    ...results.value.map(item => {
      const gender = item.sex === 'H' ? 'Men' : item.sex === 'F' ? 'Woman' : '';
      return [item.fullname, item.ip, item.nationality, gender].join(';');
    })
  ];
  const blob = new Blob([csvRows.join('\n')], {
    type: 'text/csv;charset=utf-8;',
  });
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.setAttribute('download', 'runners-scores.csv')
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
