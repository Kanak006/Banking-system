<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Generate Report</h1>
        <p class="text-sm text-gray-500">Export transaction and customer reports</p>
      </div>
    </div>
    <div class="max-w-3xl mx-auto p-6 space-y-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <h2 class="text-base font-semibold text-gray-700 mb-5">Report Settings</h2>
        <div class="space-y-5">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Report Type</label>
            <div class="grid grid-cols-2 gap-3">
              <div v-for="type in reportTypes" :key="type.value" @click="reportForm.type = type.value"
                class="p-4 rounded-xl border-2 cursor-pointer transition"
                :class="reportForm.type === type.value ? 'border-blue-500 bg-blue-50' : 'border-gray-100 hover:border-gray-300'">
                <p class="text-sm font-medium" :class="reportForm.type === type.value ? 'text-blue-700' : 'text-gray-700'">{{ type.label }}</p>
                <p class="text-xs text-gray-400 mt-1">{{ type.desc }}</p>
              </div>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">From Date</label>
              <input v-model="reportForm.startDate" type="date"
                class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">To Date</label>
              <input v-model="reportForm.endDate" type="date"
                class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
            </div>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Export Format</label>
            <div class="flex gap-3">
              <button v-for="fmt in ['CSV', 'JSON']" :key="fmt" @click="reportForm.format = fmt"
                class="flex-1 py-3 rounded-xl border-2 font-medium text-sm transition"
                :class="reportForm.format === fmt ? 'border-blue-500 bg-blue-50 text-blue-700' : 'border-gray-200 text-gray-600 hover:border-gray-300'">
                {{ fmt }}
              </button>
            </div>
          </div>
          <button @click="generateReport" :disabled="loading"
            class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-blue-300 text-white font-semibold py-3 rounded-xl transition">
            {{ loading ? 'Generating...' : 'Generate and Download ' + reportForm.format + ' Report' }}
          </button>
        </div>
      </div>

      <div v-if="previewData.length > 0" class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-base font-semibold text-gray-700">Preview ({{ previewData.length }} records)</h2>
          <span class="text-xs text-gray-400">Showing first 10 rows</span>
        </div>
        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead>
              <tr class="border-b border-gray-100">
                <th v-for="col in previewColumns" :key="col" class="text-left py-2 px-3 text-xs font-semibold text-gray-500 uppercase">{{ col }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, i) in previewData.slice(0, 10)" :key="i" class="border-b border-gray-50 hover:bg-gray-50">
                <td v-for="col in previewColumns" :key="col" class="py-2 px-3 text-gray-700">{{ row[col] !== undefined ? row[col] : '-' }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';
const loading = ref(false);
const previewData = ref([]);
const previewColumns = ref([]);

const reportTypes = [
  { value: 'transactions', label: 'Transactions', desc: 'All transaction records' },
  { value: 'customers', label: 'Customers', desc: 'Customer account data' },
  { value: 'deposits', label: 'Deposits', desc: 'All deposit records' },
  { value: 'summary', label: 'Summary', desc: 'Bank-wide overview' },
];

const today = new Date().toISOString().split('T')[0];
const monthAgo = new Date(Date.now() - 30 * 86400000).toISOString().split('T')[0];
const reportForm = ref({ type: 'transactions', startDate: monthAgo, endDate: today, format: 'CSV' });

const generateReport = async () => {
  loading.value = true; previewData.value = [];
  try {
    let data = [];
    if (reportForm.value.type === 'transactions') {
      const res = await axios.get(`${API_URL}/api/banker/transactions?startDate=${reportForm.value.startDate}&endDate=${reportForm.value.endDate}&limit=1000`, { withCredentials: true });
      data = res.data.transactions || [];
    } else if (reportForm.value.type === 'customers') {
      const res = await axios.get(`${API_URL}/api/banker/customers`, { withCredentials: true });
      data = res.data.customers || res.data || [];
    } else if (reportForm.value.type === 'deposits') {
      const res = await axios.get(`${API_URL}/api/deposits?limit=1000`, { withCredentials: true });
      data = res.data.deposits || res.data || [];
    }
    if (data.length > 0) {
      previewColumns.value = Object.keys(data[0]).slice(0, 6);
      previewData.value = data;
      downloadFile(data);
    }
  } catch (e) { console.error(e); }
  finally { loading.value = false; }
};

const downloadFile = (data) => {
  let content, filename, type;
  if (reportForm.value.format === 'CSV') {
    const keys = Object.keys(data[0]);
    const rows = [keys.join(','), ...data.map(r => keys.map(k => `"${r[k] ?? ''}"`).join(','))];
    content = rows.join('\n'); filename = `${reportForm.value.type}_report.csv`; type = 'text/csv';
  } else {
    content = JSON.stringify(data, null, 2); filename = `${reportForm.value.type}_report.json`; type = 'application/json';
  }
  const blob = new Blob([content], { type });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = filename; a.click();
  URL.revokeObjectURL(url);
};
</script>