<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpense
      :income="+income"
      :expense="+expense"
    />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="handleTransactionDeleted"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSummited" />
  </div>
</template>

<script setup>
import TransactionList from './components/TransactionList.vue'
import IncomeExpense from './components/IncomeExpense.vue'
import Balance from './components/Balance.vue'
import Header from './components/Header.vue'
import AddTransaction from './components/AddTransaction.vue'

import { ref, computed, onMounted } from 'vue'

import { useToast } from 'vue-toastification'

const toast = useToast()

const transactions = ref([])

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'))
  if (savedTransactions) transactions.value = savedTransactions
})

const total = computed(() => {
  return transactions.value
    .reduce((acc, t) => {
      return acc + t.amount
    }, 0)
    .toFixed(2)
})

const income = computed(() => {
  console.log()
  return transactions.value
    .filter((t) => t.amount > 0)
    .reduce((acc, t) => {
      return acc + t.amount
    }, 0)
    .toFixed(2)
})

const expense = computed(() => {
  return (
    transactions.value
      .filter((t) => t.amount < 0)
      .reduce((acc, t) => {
        return acc + t.amount
      }, 0)
      .toFixed(2) * -1
  )
})

const generateUniqueId = () => {
  return parseInt(transactions.value[transactions.value.length - 1].id) + 1
}

const handleTransactionSummited = (transactionData) => {
  let id = 1
  if (transactions.value.length > 0) id = generateUniqueId()
  transactions.value.push({
    id,
    ...transactionData,
  })
  saveTransactionsToLocalStorage()
  toast.success('Transaction added')
}

const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((t) => t.id !== id)
  saveTransactionsToLocalStorage()

  console.log(id)
  toast.success('Transaction deleted')
}

const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value))
}
</script>
