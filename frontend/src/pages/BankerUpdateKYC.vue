<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Update KYC</h1>
        <p class="text-sm text-gray-500">Manage customer KYC verification</p>
      </div>
    </div>
    <div class="max-w-2xl mx-auto p-6 space-y-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="flex gap-3">
          <input v-model="searchQuery" type="text" placeholder="Search customer by name, email or account number"
            class="flex-1 px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm"
            @keyup.enter="searchCustomer" />
          <button @click="searchCustomer" :disabled="loading"
            class="bg-blue-600 hover:bg-blue-700 text-white px-5 py-3 rounded-xl font-medium text-sm transition disabled:bg-blue-300">
            Search
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
          </div>
        </div>
        <p v-if="searchDone && customers.length === 0" class="mt-3 text-sm text-gray-500 text-center py-4">No customers found</p>
      </div>

      <div v-if="selected" class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
        <div class="flex items-center gap-3 mb-6 p-4 bg-gray-50 rounded-xl">
          <div>
            <p class="font-semibold text-gray-800 text-sm">{{ selected.name }}</p>
            <p class="text-xs text-gray-500">{{ selected.account_number }}</p>
          </div>
        </div>
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">KYC Status</label>
            <div class="grid grid-cols-3 gap-3">
              <div v-for="s in kycStatuses" :key="s.value" @click="kycForm.status = s.value"
                class="p-3 rounded-xl border-2 cursor-pointer transition text-center"
                :class="kycForm.status === s.value ? 'border-blue-500 bg-blue-50' : 'border-gray-100 hover:border-gray-300'">
                <p class="text-sm font-medium" :class="kycForm.status === s.value ? 'text-blue-700' : 'text-gray-600'">{{ s.label }}</p>
              </div>
            </div>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Document Type</label>
            <select v-model="kycForm.documentType"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm bg-white">
              <option value="">Select document type</option>
              <option value="aadhaar">Aadhaar Card</option>
              <option value="pan">PAN Card</option>
              <option value="passport">Passport</option>
              <option value="voter_id">Voter ID</option>
              <option value="driving_license">Driving License</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Document Number</label>
            <input v-model="kycForm.documentNumber" type="text" placeholder="Enter document number"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Remarks</label>
            <textarea v-model="kycForm.remarks" rows="3" placeholder="Add verification remarks..."
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm resize-none"></textarea>
          </div>
          <div v-if="error" class="bg-red-50 border border-red-200 rounded-xl px-4 py-3 text-sm text-red-600">{{ error }}</div>
          <div v-if="success" class="bg-green-50 border border-green-200 rounded-xl px-4 py-3 text-sm text-green-600">{{ success }}</div>
          <button @click="submitKYC" :disabled="actionLoading"
            class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-blue-300 text-white font-semibold py-3 rounded-xl transition">
            {{ actionLoading ? 'Updating...' : 'Update KYC' }}
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
const kycStatuses = [
  { value: 'pending', label: 'Pending' },
  { value: 'verified', label: 'Verified' },
  { value: 'rejected', label: 'Rejected' },
];
const kycForm = ref({ status: 'pending', documentType: '', documentNumber: '', remarks: '' });

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

const submitKYC = async () => {
  actionLoading.value = true; error.value = ''; success.value = '';
  try {
    await axios.post(`${API_URL}/api/banker/kyc`, {
      customerId: selected.value.id,
      status: kycForm.value.status,
      documentType: kycForm.value.documentType,
      documentNumber: kycForm.value.documentNumber,
      remarks: kycForm.value.remarks
    }, { withCredentials: true });
    success.value = `KYC updated successfully for ${selected.value.name}!`;
    kycForm.value = { status: 'pending', documentType: '', documentNumber: '', remarks: '' };
  } catch (err) {
    error.value = err.response?.data?.message || 'KYC update failed. Please try again.';
  } finally { actionLoading.value = false; }
};
</script>