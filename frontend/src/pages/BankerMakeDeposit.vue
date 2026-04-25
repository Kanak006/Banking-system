<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Make Deposit</h1>
        <p class="text-sm text-gray-500">Credit funds to a customer account</p>
      </div>
    </div>
    <div class="max-w-2xl mx-auto p-6 space-y-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <h2 class="text-base font-semibold text-gray-700 mb-4">Step 1: Search Customer</h2>
        <div class="flex gap-3">
          <input v-model="searchQuery" type="text" placeholder="Search by name, email or account number"
            class="flex-1 px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm"
            @keyup.enter="searchCustomer" />
          <button @click="searchCustomer" :disabled="searchLoading"
            class="bg-blue-600 hover:bg-blue-700 text-white px-5 py-3 rounded-xl font-medium text-sm transition disabled:bg-blue-300">
            {{ searchLoading ? 'Searching...' : 'Search' }}
          </button>
        </div>
        <div v-if="customers.length > 0" class="mt-4 space-y-2">
          <div v-for="c in customers" :key="c.id" @click="selectCustomer(c)"
            class="flex items-center justify-between p-4 rounded-xl border-2 cursor-pointer transition"
            :class="selectedCustomer && selectedCustomer.id === c.id ? 'border-blue-500 bg-blue-50' : 'border-gray-100 hover:border-gray-300'">
            <div>
              <p class="font-medium text-gray-800 text-sm">{{ c.name }}</p>
              <p class="text-xs text-gray-500">{{ c.account_number }} - {{ c.email }}</p>
            </div>
            <div class="text-right">
              <p class="text-sm font-semibold text-gray-800">Rs. {{ Number(c.balance || 0).toLocaleString('en-IN') }}</p>
              <p class="text-xs text-gray-400">Balance</p>
            </div>
          </div>
        </div>
        <p v-if="searchDone && customers.length === 0" class="mt-3 text-sm text-gray-500 text-center py-4">No customers found</p>
      </div>

      <div v-if="selectedCustomer" class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <h2 class="text-base font-semibold text-gray-700 mb-4">Step 2: Deposit Details</h2>
        <div class="bg-blue-50 rounded-xl p-4 mb-5">
          <p class="font-semibold text-gray-800 text-sm">{{ selectedCustomer.name }}</p>
          <p class="text-xs text-gray-500">{{ selectedCustomer.account_number }}</p>
          <p class="text-sm font-bold text-blue-600 mt-1">Balance: Rs. {{ Number(selectedCustomer.balance || 0).toLocaleString('en-IN') }}</p>
        </div>
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Amount (Rs.) *</label>
            <input v-model="depositForm.amount" type="number" min="1" placeholder="Enter deposit amount"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Deposit Type</label>
            <select v-model="depositForm.type"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm bg-white">
              <option value="cash">Cash Deposit</option>
              <option value="cheque">Cheque Deposit</option>
              <option value="transfer">Bank Transfer</option>
              <option value="banker_credit">Banker Credit</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
            <input v-model="depositForm.description" type="text" placeholder="e.g. Cash deposit at branch"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div v-if="error" class="bg-red-50 border border-red-200 rounded-xl px-4 py-3 text-sm text-red-600">{{ error }}</div>
          <div v-if="success" class="bg-green-50 border border-green-200 rounded-xl px-4 py-3 text-sm text-green-600">{{ success }}</div>
          <button @click="makeDeposit" :disabled="loading || !depositForm.amount"
            class="w-full bg-green-600 hover:bg-green-700 disabled:bg-green-300 text-white font-semibold py-3 rounded-xl transition">
            {{ loading ? 'Processing...' : 'Make Deposit' }}
          </button>
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
const searchLoading = ref(false);
const searchDone = ref(false);
const customers = ref([]);
const selectedCustomer = ref(null);
const loading = ref(false);
const error = ref('');
const success = ref('');
const depositForm = ref({ amount: '', type: 'cash', description: '' });

const searchCustomer = async () => {
  if (!searchQuery.value.trim()) return;
  searchLoading.value = true; searchDone.value = false; customers.value = [];
  try {
      const res = await axios.get(`${API_URL}/api/banker/customers`, { withCredentials: true });
      console.log('API Response:', res.data);
    const all = res.data.data?.customers || res.data.customers || res.data.data || res.data || [];
    const q = searchQuery.value.toLowerCase();
    customers.value = all.filter(c =>
      c.name?.toLowerCase().includes(q) || c.email?.toLowerCase().includes(q) || c.account_number?.toLowerCase().includes(q)
    );
  } catch (err) { error.value = 'Failed to search customers'; }
  finally { searchLoading.value = false; searchDone.value = true; }
};

const selectCustomer = (c) => { selectedCustomer.value = c; error.value = ''; success.value = ''; };

const makeDeposit = async () => {
  if (!depositForm.value.amount || depositForm.value.amount <= 0) { error.value = 'Please enter a valid amount'; return; }
  loading.value = true; error.value = ''; success.value = '';
  try {
    await axios.post(`${API_URL}/api/banker/deposit`, {
      customerId: selectedCustomer.value.id,
      amount: Number(depositForm.value.amount),
      type: depositForm.value.type,
      description: depositForm.value.description || 'Deposit made by banker'
    }, { withCredentials: true });
    success.value = `Successfully deposited Rs. ${Number(depositForm.value.amount).toLocaleString('en-IN')} to ${selectedCustomer.value.name}'s account!`;
    selectedCustomer.value.balance = Number(selectedCustomer.value.balance) + Number(depositForm.value.amount);
    depositForm.value = { amount: '', type: 'cash', description: '' };
  } catch (err) {
    error.value = err.response?.data?.message || 'Deposit failed. Please try again.';
  } finally { loading.value = false; }
};
</script>