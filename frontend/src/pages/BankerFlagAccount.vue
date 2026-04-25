<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Flag Account</h1>
        <p class="text-sm text-gray-500">Suspend or flag suspicious customer accounts</p>
      </div>
    </div>
    <div class="max-w-2xl mx-auto p-6 space-y-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="flex gap-3">
          <input v-model="searchQuery" type="text" placeholder="Search customer by name, email or account number"
            class="flex-1 px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-red-400 text-sm"
            @keyup.enter="searchCustomer" />
          <button @click="searchCustomer" :disabled="loading"
            class="bg-red-500 hover:bg-red-600 text-white px-5 py-3 rounded-xl font-medium text-sm transition disabled:bg-red-300">
            Search
          </button>
        </div>
        <div v-if="customers.length > 0" class="mt-4 space-y-2">
          <div v-for="c in customers" :key="c.id" @click="selectCustomer(c)"
            class="flex items-center justify-between p-4 rounded-xl border-2 cursor-pointer transition"
            :class="selected && selected.id === c.id ? 'border-red-400 bg-red-50' : 'border-gray-100 hover:border-gray-300'">
            <div>
              <p class="font-medium text-gray-800 text-sm">{{ c.name }}</p>
              <p class="text-xs text-gray-500">{{ c.account_number }} - {{ c.email }}</p>
            </div>
            <span class="px-2 py-1 rounded-full text-xs font-medium"
              :class="c.status === 'active' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">
              {{ c.status }}
            </span>
          </div>
        </div>
        <p v-if="searchDone && customers.length === 0" class="mt-3 text-sm text-gray-500 text-center py-4">No customers found</p>
      </div>

      <div v-if="selected" class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="bg-amber-50 border border-amber-200 rounded-xl p-4 mb-6">
          <p class="text-sm font-semibold text-amber-800">Warning: Account Action</p>
          <p class="text-xs text-amber-700 mt-1">Flagging or suspending restricts customer access. This action is logged.</p>
        </div>
        <div class="p-4 bg-gray-50 rounded-xl mb-5">
          <p class="font-semibold text-gray-800 text-sm">{{ selected.name }}</p>
          <p class="text-xs text-gray-500">{{ selected.account_number }}</p>
          <span class="mt-1 inline-block px-2 py-1 rounded-full text-xs font-medium"
            :class="selected.status === 'active' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">
            {{ selected.status }}
          </span>
        </div>
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Action *</label>
            <select v-model="flagForm.action"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-red-400 text-sm bg-white">
              <option value="flag">Flag for Review</option>
              <option value="suspend">Suspend Account</option>
              <option value="activate">Re-activate Account</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Reason *</label>
            <select v-model="flagForm.reason"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-red-400 text-sm bg-white">
              <option value="">Select a reason</option>
              <option value="suspicious_activity">Suspicious Activity</option>
              <option value="fraud">Fraud Suspected</option>
              <option value="kyc_incomplete">KYC Incomplete</option>
              <option value="multiple_failed_logins">Multiple Failed Logins</option>
              <option value="customer_request">Customer Request</option>
              <option value="other">Other</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Additional Notes</label>
            <textarea v-model="flagForm.notes" rows="3" placeholder="Add any additional notes..."
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-red-400 text-sm resize-none"></textarea>
          </div>
          <div v-if="error" class="bg-red-50 border border-red-200 rounded-xl px-4 py-3 text-sm text-red-600">{{ error }}</div>
          <div v-if="success" class="bg-green-50 border border-green-200 rounded-xl px-4 py-3 text-sm text-green-600">{{ success }}</div>
          <button @click="submitFlag" :disabled="actionLoading || !flagForm.reason"
            class="w-full font-semibold py-3 rounded-xl transition text-white"
            :class="flagForm.action === 'activate' ? 'bg-green-600 hover:bg-green-700 disabled:bg-green-300' : 'bg-red-500 hover:bg-red-600 disabled:bg-red-300'">
            {{ actionLoading ? 'Processing...' : (flagForm.action === 'activate' ? 'Activate Account' : flagForm.action === 'suspend' ? 'Suspend Account' : 'Flag Account') }}
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
const loading = ref(false);
const searchDone = ref(false);
const customers = ref([]);
const selected = ref(null);
const actionLoading = ref(false);
const error = ref('');
const success = ref('');
const flagForm = ref({ action: 'flag', reason: '', notes: '' });

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
  } catch (e) { error.value = 'Search failed'; }
  finally { loading.value = false; searchDone.value = true; }
};

const selectCustomer = (c) => { selected.value = c; error.value = ''; success.value = ''; };

const submitFlag = async () => {
  if (!flagForm.value.reason) { error.value = 'Please select a reason'; return; }
  actionLoading.value = true; error.value = ''; success.value = '';
  try {
    const newStatus = flagForm.value.action === 'activate' ? 'active' : flagForm.value.action === 'suspend' ? 'suspended' : 'flagged';
    await axios.patch(`${API_URL}/api/banker/customers/${selected.value.id}/status`,
      { status: newStatus, reason: flagForm.value.reason, notes: flagForm.value.notes },
      { withCredentials: true }
    );
    success.value = `Account ${flagForm.value.action}ed successfully!`;
    selected.value.status = newStatus;
    flagForm.value = { action: 'flag', reason: '', notes: '' };
  } catch (err) {
    error.value = err.response?.data?.message || 'Action failed. Please try again.';
  } finally { actionLoading.value = false; }
};
</script>