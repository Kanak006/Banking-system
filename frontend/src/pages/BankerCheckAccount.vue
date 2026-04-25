<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Check Account</h1>
        <p class="text-sm text-gray-500">View customer account details and transactions</p>
      </div>
    </div>
    <div class="max-w-4xl mx-auto p-6 space-y-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="flex gap-3">
          <input v-model="searchQuery" type="text" placeholder="Search by name, email or account number"
            class="flex-1 px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm"
            @keyup.enter="searchCustomer" />
          <button @click="searchCustomer" :disabled="loading"
            class="bg-blue-600 hover:bg-blue-700 text-white px-5 py-3 rounded-xl font-medium text-sm transition disabled:bg-blue-300">
            {{ loading ? 'Searching...' : 'Search' }}
          </button>
        </div>
        <div v-if="customers.length > 0" class="mt-4 space-y-2">
          <div v-for="c in customers" :key="c.id" @click="selectCustomer(c)"
            class="flex items-center justify-between p-4 rounded-xl border-2 cursor-pointer transition"
            :class="selected && selected.id === c.id ? 'border-blue-500 bg-blue-50' : 'border-gray-100 hover:border-gray-200'">
            <div>
              <p class="font-medium text-gray-800 text-sm">{{ c.name }}</p>
              <p class="text-xs text-gray-500">{{ c.email }}</p>
            </div>
            <span class="px-2 py-1 rounded-full text-xs font-medium"
              :class="c.status === 'active' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">
              {{ c.status }}
            </span>
          </div>
        </div>
        <p v-if="searchDone && customers.length === 0" class="mt-3 text-sm text-gray-500 text-center py-4">No customers found</p>
      </div>

      <div v-if="selected" class="space-y-4">
        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
          <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm">
            <p class="text-xs text-gray-500 mb-1">Balance</p>
            <p class="text-lg font-bold text-blue-600">Rs. {{ Number(selected.balance || 0).toLocaleString('en-IN') }}</p>
          </div>
          <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm">
            <p class="text-xs text-gray-500 mb-1">Account Type</p>
            <p class="text-sm font-semibold text-gray-800 capitalize">{{ selected.account_type || 'Savings' }}</p>
          </div>
          <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm">
            <p class="text-xs text-gray-500 mb-1">Status</p>
            <span class="px-2 py-1 rounded-full text-xs font-medium"
              :class="selected.status === 'active' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">
              {{ selected.status }}
            </span>
          </div>
          <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm">
            <p class="text-xs text-gray-500 mb-1">Customer ID</p>
            <p class="text-sm font-semibold text-gray-800">{{ selected.customer_id }}</p>
          </div>
        </div>

        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
          <h3 class="font-semibold text-gray-800 mb-4">Account Details</h3>
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div><p class="text-xs text-gray-500">Full Name</p><p class="text-sm font-medium text-gray-800">{{ selected.name }}</p></div>
            <div><p class="text-xs text-gray-500">Email</p><p class="text-sm font-medium text-gray-800">{{ selected.email }}</p></div>
            <div><p class="text-xs text-gray-500">Phone</p><p class="text-sm font-medium text-gray-800">{{ selected.phone || '-' }}</p></div>
            <div><p class="text-xs text-gray-500">Account Number</p><p class="text-sm font-medium text-gray-800">{{ selected.account_number }}</p></div>
            <div><p class="text-xs text-gray-500">Address</p><p class="text-sm font-medium text-gray-800">{{ selected.address || '-' }}</p></div>
            <div><p class="text-xs text-gray-500">Member Since</p><p class="text-sm font-medium text-gray-800">{{ selected.created_at ? new Date(selected.created_at).toLocaleDateString('en-IN') : '-' }}</p></div>
          </div>
        </div>

        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
          <h3 class="font-semibold text-gray-800 mb-4">Recent Transactions</h3>
          <div v-if="txLoading" class="text-center py-8 text-gray-400 text-sm">Loading...</div>
          <div v-else-if="transactions.length === 0" class="text-center py-8 text-gray-400 text-sm">No transactions found</div>
          <div v-else class="space-y-3">
            <div v-for="tx in transactions" :key="tx.id" class="flex items-center justify-between py-3 border-b border-gray-50 last:border-0">
              <div>
                <p class="text-sm font-medium text-gray-800 capitalize">{{ tx.type }}</p>
                <p class="text-xs text-gray-400">{{ new Date(tx.created_at).toLocaleDateString('en-IN') }}</p>
              </div>
              <p class="text-sm font-semibold" :class="tx.type === 'credit' || tx.type === 'deposit' ? 'text-green-600' : 'text-red-600'">
                Rs. {{ Number(tx.amount || 0).toLocaleString('en-IN') }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';
const searchQuery = ref('');
const loading = ref(false);
const searchDone = ref(false);
const customers = ref([]);
const selected = ref(null);
const transactions = ref([]);
const txLoading = ref(false);

const searchCustomer = async () => {
  if (!searchQuery.value.trim()) return;
  loading.value = true; searchDone.value = false; customers.value = [];
  try {
    const res = await axios.get(`${API_URL}/api/banker/customers`, { withCredentials: true });
    const all = res.data.data?.customers || res.data.customers || res.data.data || res.data || [];
    const q = searchQuery.value.toLowerCase();
    customers.value = all.filter(c =>
      c.name?.toLowerCase().includes(q) || c.email?.toLowerCase().includes(q) || c.account_number?.toLowerCase().includes(q)
    );
  } catch (e) { console.error(e); }
  finally { loading.value = false; searchDone.value = true; }
};

const selectCustomer = async (c) => {
  selected.value = c; txLoading.value = true; transactions.value = [];
  try {
    const res = await axios.get(`${API_URL}/api/banker/transactions?limit=100`, { withCredentials: true });
    const all = res.data.transactions || [];
    transactions.value = all.filter(t => t.customer_id === c.id).slice(0, 10);
  } catch (e) { console.error(e); }
  finally { txLoading.value = false; }
};
</script>