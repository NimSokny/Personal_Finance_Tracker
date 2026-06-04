<template>
  <div class="min-h-screen bg-slate-100 p-8">
    <div class="max-w-5xl mx-auto">

      <h1 class="text-4xl font-bold text-center mb-8">
        Personal Finance Tracker
      </h1>

      <!-- Add Transaction -->
      <div class="bg-white p-6 rounded-lg shadow mb-6">
        <h2 class="text-xl font-semibold mb-4">
          Add Transaction
        </h2>

        <div class="grid md:grid-cols-4 gap-4">

          <input
            v-model="desc"
            type="text"
            placeholder="Description"
            class="border rounded px-3 py-2"
          />

          <input
            v-model.number="amount"
            type="number"
            placeholder="Amount"
            class="border rounded px-3 py-2"
          />

          <select
            v-model="type"
            class="border rounded px-3 py-2"
          >
            <option value="income">Income</option>
            <option value="expense">Expense</option>
          </select>

          <button
            @click="addTransaction"
            class="bg-blue-500 text-white rounded px-4 py-2 hover:bg-blue-600"
          >
            Add
          </button>

        </div>
      </div>

      <!-- Budget Section -->
      <div class="bg-white p-6 rounded-lg shadow mb-6">

        <div class="flex flex-col md:flex-row gap-4">

          <input
            type="number"
            v-model.number="budgetLimit"
            class="border rounded px-3 py-2"
            placeholder="Budget Limit"
          />

          <select
            v-model="filterType"
            class="border rounded px-3 py-2"
          >
            <option value="all">All</option>
            <option value="income">Income</option>
            <option value="expense">Expense</option>
          </select>

        </div>

      </div>

      <!-- Summary Cards -->
      <div class="grid md:grid-cols-3 gap-6 mb-6">

        <div class="bg-green-500 text-white p-6 rounded-lg shadow">
          <h3 class="text-lg font-semibold">
            Income
          </h3>
          <p class="text-2xl">
            ${{ totalIncome }}
          </p>
        </div>

        <div class="bg-red-500 text-white p-6 rounded-lg shadow">
          <h3 class="text-lg font-semibold">
            Expenses
          </h3>
          <p class="text-2xl">
            ${{ totalExpenses }}
          </p>
        </div>

        <div class="bg-blue-500 text-white p-6 rounded-lg shadow">
          <h3 class="text-lg font-semibold">
            Balance
          </h3>
          <p class="text-2xl">
            ${{ balance }}
          </p>
        </div>

      </div>

      <!-- Progress Bar -->
      <div class="bg-white p-6 rounded-lg shadow mb-6">

        <h3 class="font-semibold mb-2">
          Budget Usage
        </h3>

        <div class="w-full bg-gray-200 rounded-full h-6">

          <div
            class="bg-purple-500 h-6 rounded-full text-white text-center"
            :style="{ width: expensePercentage + '%' }"
          >
            {{ expensePercentage.toFixed(0) }}%
          </div>

        </div>

        <p
          class="mt-3 font-semibold"
          :class="
            isOverBudget
              ? 'text-red-500'
              : 'text-green-500'
          "
        >
          {{ budgetStatus }}
        </p>

      </div>

      <!-- Transactions -->
      <div class="bg-white p-6 rounded-lg shadow mb-6">

        <div class="flex justify-between mb-4">

          <h2 class="text-xl font-semibold">
            Transactions
          </h2>

          <button
            @click="clearAll"
            class="bg-red-500 text-white px-4 py-2 rounded"
          >
            Clear All
          </button>

        </div>

        <table class="w-full">

          <thead>
            <tr class="border-b">
              <th class="text-left p-2">
                Description
              </th>
              <th class="text-left p-2">
                Amount
              </th>
              <th class="text-left p-2">
                Type
              </th>
              <th class="text-left p-2">
                Action
              </th>
            </tr>
          </thead>

          <tbody>

            <tr
              v-for="transaction in filteredTransactions"
              :key="transaction.id"
              class="border-b"
            >
              <td class="p-2">
                {{ transaction.desc }}
              </td>

              <td class="p-2">
                ${{ transaction.amount }}
              </td>

              <td class="p-2 capitalize">
                {{ transaction.type }}
              </td>

              <td class="p-2">
                <button
                  @click="deleteTransaction(transaction.id)"
                  class="bg-red-500 text-white px-3 py-1 rounded"
                >
                  Delete
                </button>
              </td>
            </tr>

          </tbody>

        </table>

      </div>

      <!-- Category Summary -->
      <div class="bg-white p-6 rounded-lg shadow">

        <h2 class="text-xl font-semibold mb-4">
          Category Summary
        </h2>

        <div
          v-for="(value,key) in categorySummary"
          :key="key"
          class="mb-2"
        >
          <span class="font-semibold">
            {{ key }}
          </span>

          : ${{ value }}
        </div>

      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'

interface Transaction {
  id: number
  desc: string
  amount: number
  type: 'income' | 'expense'
  date: string
}

const transactions = ref<Transaction[]>(
  JSON.parse(
    localStorage.getItem('transactions') || '[]'
  )
)

const desc = ref('')
const amount = ref(0)

const type = ref<'income' | 'expense'>(
  'income'
)

const filterType = ref<
  'all' | 'income' | 'expense'
>('all')

const budgetLimit = ref(1000)

const addTransaction = (): void => {
  if (!desc.value || amount.value <= 0)
    return

  transactions.value.push({
    id: Date.now(),
    desc: desc.value,
    amount: amount.value,
    type: type.value,
    date: new Date().toISOString()
  })

  desc.value = ''
  amount.value = 0
}

const deleteTransaction = (
  id: number
): void => {
  transactions.value =
    transactions.value.filter(
      item => item.id !== id
    )
}

const clearAll = (): void => {
  transactions.value = []
}

const filteredTransactions = computed(() => {
  if (filterType.value === 'all')
    return transactions.value

  return transactions.value.filter(
    item =>
      item.type === filterType.value
  )
})

const totalIncome = computed(() =>
  transactions.value
    .filter(t => t.type === 'income')
    .reduce(
      (sum, t) => sum + t.amount,
      0
    )
)

const totalExpenses = computed(() =>
  transactions.value
    .filter(t => t.type === 'expense')
    .reduce(
      (sum, t) => sum + t.amount,
      0
    )
)

const balance = computed(
  () =>
    totalIncome.value -
    totalExpenses.value
)

const isOverBudget = computed(
  () =>
    totalExpenses.value >
    budgetLimit.value
)

const expensePercentage =
  computed(() =>
    Math.min(
      (totalExpenses.value /
        budgetLimit.value) *
        100,
      100
    )
  )

const categorySummary = computed(() => {
  const summary: Record<
    string,
    number
  > = {}

  transactions.value.forEach(item => {
    summary[item.type] =
      (summary[item.type] || 0) +
      item.amount
  })

  return summary
})

const budgetStatus = computed(() =>
  isOverBudget.value
    ? 'Over Budget'
    : 'Within Budget'
)

watch(
  transactions,
  value => {
    localStorage.setItem(
      'transactions',
      JSON.stringify(value)
    )
  },
  { deep: true }
)

watch(balance, value => {
  if (value < 0) {
    alert('Balance is negative!')
  }
})

watch(isOverBudget, value => {
  if (value) {
    console.log(
      'Budget limit exceeded!'
    )
  }
})
</script>