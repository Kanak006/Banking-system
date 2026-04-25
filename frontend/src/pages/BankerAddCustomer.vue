<template>
  <div class="min-h-screen bg-gray-50">
    <div class="bg-white border-b border-gray-200 px-6 py-4 flex items-center gap-4">
      <button @click="$router.push('/banker/dashboard')" class="px-3 py-2 hover:bg-gray-100 rounded-lg transition text-sm text-gray-600 font-medium">
        &larr; Back
      </button>
      <div>
        <h1 class="text-xl font-semibold text-gray-800">Add New Customer</h1>
        <p class="text-sm text-gray-500">Create a new customer account</p>
      </div>
    </div>
    <div class="max-w-2xl mx-auto p-6">
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-8">
        <form @submit.prevent="submitForm" class="space-y-5">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Full Name *</label>
            <input v-model="form.name" type="text" placeholder="Enter full name" required
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Email Address *</label>
            <input v-model="form.email" type="email" placeholder="Enter email address" required
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Phone Number *</label>
            <input v-model="form.phone" type="tel" placeholder="+91 XXXXX XXXXX" required
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Initial Password *</label>
            <input v-model="form.password" type="password" placeholder="Set initial password" required
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Address</label>
            <textarea v-model="form.address" rows="3" placeholder="Enter residential address"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm resize-none"></textarea>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Account Type *</label>
            <select v-model="form.accountType"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm bg-white">
              <option value="savings">Savings Account</option>
              <option value="current">Current Account</option>
              <option value="fixed">Fixed Deposit Account</option>
            </select>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Initial Deposit (Rs.)</label>
            <input v-model="form.initialDeposit" type="number" min="0" placeholder="0.00"
              class="w-full px-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm" />
          </div>
          <div v-if="error" class="bg-red-50 border border-red-200 rounded-xl px-4 py-3 text-sm text-red-600">{{ error }}</div>
          <div v-if="success" class="bg-green-50 border border-green-200 rounded-xl px-4 py-3 text-sm text-green-600">{{ success }}</div>
          <button type="submit" :disabled="loading"
            class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-blue-300 text-white font-semibold py-3 rounded-xl transition">
            {{ loading ? 'Creating Account...' : 'Create Customer Account' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const loading = ref(false);
const error = ref('');
const success = ref('');
const form = ref({ name: '', email: '', phone: '', password: '', address: '', accountType: 'savings', initialDeposit: 0 });

const submitForm = async () => {
  loading.value = true; error.value = ''; success.value = '';
  try {
    const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    await axios.post(`${API_URL}/api/banker/add-customer`, form.value, { withCredentials: true });
    success.value = `Customer account created successfully for ${form.value.name}!`;
    form.value = { name: '', email: '', phone: '', password: '', address: '', accountType: 'savings', initialDeposit: 0 };
    setTimeout(() => router.push('/banker/dashboard'), 2000);
  } catch (err) {
    error.value = err.response?.data?.message || 'Failed to create customer. Please try again.';
  } finally {
    loading.value = false;
  }
};
</script>