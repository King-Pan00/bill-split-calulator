<script setup>
import { ref, computed } from 'vue'

// 1. Reactive state
const billAmount = ref(50000)
const tipPercentage = ref(10)
const customTip = ref('')
const isCustomTip = ref(false)
const numberOfPeople = ref(2)

// Tip presets
const tipOptions = [0, 5, 10, 15]

// Select predefined tip percentage
function selectTip(val) {
  isCustomTip.value = false
  tipPercentage.value = val
  customTip.value = ''
}

// Select custom tip mode
function enableCustomTip() {
  isCustomTip.value = true
  if (customTip.value !== '') {
    tipPercentage.value = Number(customTip.value)
  }
}

// Update custom tip value
function updateCustomTip() {
  const val = Number(customTip.value)
  tipPercentage.value = isNaN(val) ? 0 : val
}

// Increment / Decrement people count
function incrementPeople() {
  numberOfPeople.value++
}

function decrementPeople() {
  if (numberOfPeople.value > 1) {
    numberOfPeople.value--
  }
}

// Reset calculator back to default
function resetCalculator() {
  billAmount.value = ''
  tipPercentage.value = 0
  customTip.value = ''
  isCustomTip.value = false
  numberOfPeople.value = 1
}

// 2. Validation checks
const errors = computed(() => {
  const errs = []
  if (billAmount.value !== '' && Number(billAmount.value) < 0) {
    errs.push('Bill amount must be 0 or greater.')
  }
  if (tipPercentage.value < 0 || tipPercentage.value > 100) {
    errs.push('Tip percentage must be between 0 and 100.')
  }
  if (numberOfPeople.value <= 0 || !Number.isInteger(Number(numberOfPeople.value))) {
    errs.push('Number of people must be a whole number greater than 0.')
  }
  return errs
})

const isValid = computed(() => errors.value.length === 0)

// 3. Computed calculations
const safeBill = computed(() => Math.max(0, Number(billAmount.value) || 0))
const safeTipPct = computed(() => Math.max(0, Math.min(100, Number(tipPercentage.value) || 0)))
const safePeople = computed(() => Math.max(1, Number(numberOfPeople.value) || 1))

// Tip Amount = Bill Amount * Tip Percentage / 100
const tipAmount = computed(() => {
  if (!isValid.value) return 0
  return (safeBill.value * safeTipPct.value) / 100
})

// Total Bill = Bill Amount + Tip Amount
const totalBill = computed(() => {
  if (!isValid.value) return 0
  return safeBill.value + tipAmount.value
})

// Amount Per Person = Total Bill / Number of People
const amountPerPerson = computed(() => {
  if (!isValid.value || safePeople.value <= 0) return 0
  return totalBill.value / safePeople.value
})

// 4. Number formatter
function formatCurrency(num) {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 0,
    maximumFractionDigits: 2
  }).format(num)
}
</script>

<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="w-full max-w-md bg-white rounded-3xl p-6 sm:p-8 shadow-lg">
      
      <!-- Header -->
      <div class="text-center mb-6">
        <h1 class="text-xl sm:text-2xl font-bold text-gray-900 flex items-center justify-center gap-2">
          <span>💸</span> Bill Split Calculator
        </h1>
        <p class="text-sm text-gray-500 mt-1">Split your bill easily with friends</p>
      </div>

      <!-- Validation Errors -->
      <div v-if="errors.length > 0" class="mb-5 p-3 rounded-xl bg-red-50 border border-red-200 text-red-600 text-xs sm:text-sm">
        <p v-for="(err, idx) in errors" :key="idx" class="leading-relaxed">{{ err }}</p>
      </div>

      <!-- Input Form -->
      <div class="space-y-5">
        
        <!-- Bill Amount Input -->
        <div>
          <label class="block text-sm font-semibold text-gray-700 mb-2">Bill Amount (MMK)</label>
          <input 
            type="number" 
            v-model.number="billAmount" 
            placeholder="0" 
            min="0"
            class="w-full px-4 py-3 bg-gray-50 border border-gray-200 rounded-2xl text-gray-900 focus:outline-none focus:ring-2 focus:ring-gray-900 focus:bg-white transition"
          />
        </div>

        <!-- Tip Percentage -->
        <div>
          <label class="block text-sm font-semibold text-gray-700 mb-2">Tip Percentage</label>
          <div class="grid grid-cols-4 gap-2 mb-3">
            <button
              v-for="option in tipOptions"
              :key="option"
              type="button"
              :class="[
                'py-2.5 rounded-full font-semibold text-sm border transition',
                !isCustomTip && tipPercentage === option 
                  ? 'bg-gray-900 text-white border-gray-900' 
                  : 'bg-white text-gray-700 border-gray-200 hover:bg-gray-50'
              ]"
              @click="selectTip(option)"
            >
              {{ option }}%
            </button>
          </div>

          <!-- Custom Tip Option -->
          <div class="flex items-center gap-3">
            <label class="flex items-center gap-2 text-sm text-gray-700 cursor-pointer font-medium">
              <input 
                type="radio" 
                name="tipChoice" 
                :checked="isCustomTip" 
                @change="enableCustomTip"
                class="accent-gray-900 w-4 h-4 cursor-pointer"
              />
              <span>Custom tip</span>
            </label>
            <input
              v-if="isCustomTip"
              type="number"
              v-model.number="customTip"
              @input="updateCustomTip"
              placeholder="e.g. 18"
              min="0"
              max="100"
              class="w-28 px-3 py-1.5 text-sm bg-gray-50 border border-gray-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-gray-900 focus:bg-white transition"
            />
          </div>
        </div>

        <!-- Number of People -->
        <div>
          <label class="block text-sm font-semibold text-gray-700 mb-2">Number of People</label>
          <div class="flex items-center gap-2">
            <input 
              type="number" 
              v-model.number="numberOfPeople" 
              min="1" 
              class="flex-1 px-4 py-3 bg-gray-50 border border-gray-200 rounded-2xl text-gray-900 focus:outline-none focus:ring-2 focus:ring-gray-900 focus:bg-white transition"
            />
            <button 
              type="button" 
              @click="decrementPeople"
              class="w-12 h-12 rounded-full border border-gray-200 bg-white hover:bg-gray-100 flex items-center justify-center text-xl text-gray-700 font-bold transition active:scale-95"
            >
              -
            </button>
            <button 
              type="button" 
              @click="incrementPeople"
              class="w-12 h-12 rounded-full border border-gray-200 bg-white hover:bg-gray-100 flex items-center justify-center text-xl text-gray-700 font-bold transition active:scale-95"
            >
              +
            </button>
          </div>
        </div>
      </div>

      <!-- Result Card -->
      <div class="mt-6 bg-gray-50 border border-gray-100 rounded-2xl p-4 sm:p-5">
        <div class="flex justify-between py-1.5 text-sm text-gray-600">
          <span>Bill Amount</span>
          <span class="font-medium text-gray-900">{{ formatCurrency(safeBill) }} MMK</span>
        </div>
        <div class="flex justify-between py-1.5 text-sm text-gray-600">
          <span>Tip Amount ({{ safeTipPct }}%)</span>
          <span class="font-medium text-gray-900">{{ formatCurrency(tipAmount) }} MMK</span>
        </div>
        <div class="flex justify-between pt-3 pb-2 text-sm border-t border-dashed border-gray-300 font-semibold text-gray-900">
          <span>Total Bill</span>
          <span>{{ formatCurrency(totalBill) }} MMK</span>
        </div>

        <!-- Callout: Each Person Pays -->
        <div class="text-center mt-4 pt-3">
          <div class="text-xs text-gray-500 uppercase tracking-wider font-semibold">Each Person Pays</div>
          <div class="text-2xl sm:text-3xl font-extrabold text-gray-900 mt-1">
            {{ formatCurrency(amountPerPerson) }} MMK
          </div>
        </div>
      </div>

      <!-- Reset Button -->
      <button 
        type="button" 
        @click="resetCalculator"
        class="mt-5 w-full py-3 px-4 rounded-full border border-gray-200 bg-white hover:bg-gray-50 text-gray-700 font-semibold text-sm transition active:scale-[0.98] flex items-center justify-center gap-2"
      >
        <span>🔄️</span> Reset Calculator
      </button>

    </div>
  </div>
</template>