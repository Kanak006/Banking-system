<template>
  <div class="min-h-screen bg-gray-50">
    <!-- Header -->
    <div class="bg-white shadow-sm px-6 py-4 flex items-center justify-between">
      <div class="flex items-center space-x-3">
        <button @click="$router.push('/customer/dashboard')"
          class="text-gray-500 hover:text-gray-700">
          <ArrowLeft class="h-5 w-5" />
        </button>
        <h1 class="text-xl font-semibold text-gray-900">Transaction History</h1>
      </div>
      <button @click="showExport = true"
        class="flex items-center px-4 py-2 bg-blue-600 text-white rounded-md text-sm hover:bg-blue-700">
        <Download class="h-4 w-4 mr-2" />
        Export
      </button>
    </div>

    <!-- Content -->
    <div class="max-w-4xl mx-auto px-4 py-6">
      <!-- Loading -->
      <div v-if="loading" class="flex justify-center py-12">
        <Loader2 class="h-8 w-8 animate-spin text-blue-600" />
      </div>

      <!-- Transactions List -->
      <div v-else class="bg-white rounded-lg shadow overflow-hidden">
        <div v-if="transactions.length === 0" class="text-center py-12 text-gray-500">
          No transactions found.
        </div>
        <ul v-else class="divide-y divide-gray-200">
          <li v-for="t in transactions" :key="t.id"
            class="px-6 py-4 flex items-center justify-between hover:bg-gray-50">
            <div class="flex items-center space-x-4">
              <div :class="[
                'rounded-full p-2',
                t.type === 'deposit' || t.type === 'transfer_in' ? 'bg-green-50' :
                t.type === 'transfer' ? 'bg-blue-50' : 'bg-red-50'
              ]">
                <ArrowDownLeft v-if="t.type === 'deposit' || t.type === 'transfer_in'"
                  class="h-5 w-5 text-green-500" />
                <Send v-else-if="t.type === 'transfer'"
                  class="h-5 w-5 text-blue-500" />
                <ArrowUpRight v-else class="h-5 w-5 text-red-500" />
              </div>
              <div>
                <p class="text-sm font-medium text-gray-900 capitalize">{{ formatType(t.type) }}</p>
                <p class="text-xs text-gray-500">{{ formatDate(t.created_at) }}</p>
                <p v-if="t.description" class="text-xs text-gray-400">{{ t.description }}</p>
              </div>
            </div>
            <div class="text-right">
              <p :class="[
                'text-sm font-semibold',
                t.type === 'deposit' || t.type === 'transfer_in' ? 'text-green-600' : 'text-red-600'
              ]">
                {{ t.type === 'deposit' || t.type === 'transfer_in' ? '+' : '-' }}
                {{ formatCurrency(t.amount) }}
              </p>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- Export Modal -->
    <ExportReportModal
      v-model="showExport"
      :transactions="transactions"
      @export="handleExport"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { ArrowLeft, ArrowDownLeft, ArrowUpRight, Send, Download, Loader2 } from 'lucide-vue-next';
import { useToast } from 'vue-toastification';
import ExportReportModal from '../components/ExportReportModal.vue';
import axios from 'axios';

const toast = useToast();
const transactions = ref([]);
const loading = ref(true);
const showExport = ref(false);

onMounted(async () => {
  try {
    const res = await axios.get('/api/customers/transactions');
    transactions.value = res.data.data || [];
  } catch (e) {
    toast.error('Failed to load transactions');
  } finally {
    loading.value = false;
  }
});

const formatType = (type) => ({
  deposit: 'Deposit', withdrawal: 'Withdrawal', withdraw: 'Withdrawal',
  transfer: 'Transfer', transfer_in: 'Money Received', transfer_out: 'Money Sent'
}[type] || type);

const formatCurrency = (amount) =>
  new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' }).format(amount);

const formatDate = (d) => new Intl.DateTimeFormat('en-IN', {
  day: 'numeric', month: 'short', year: 'numeric',
  hour: '2-digit', minute: '2-digit'
}).format(new Date(d));

const handleExport = ({ data, format, filename }) => {
  try {
    const headers = Object.keys(data[0]).join(',');
    const rows = data.map(row =>
      Object.values(row).map(v => `"${String(v ?? '').replace(/"/g, '""')}"`).join(',')
    );
    const blob = new Blob([[headers, ...rows].join('\n')], { type: 'text/csv;charset=utf-8;' });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = `${filename}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(a.href);
    toast.success('Transactions exported!');
  } catch {
    toast.error('Failed to export');
  }
};
</script>