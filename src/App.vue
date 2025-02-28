<template>
  <Header/>
  <div class="container">
    <Balance :total="+total"/>
    <IncomeExpenses :income="+income" :expenses="+expenses"/>
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
  import { ref, computed, onMounted } from 'vue';
  import { useToast } from 'vue-toastification';

  import Header from "./components/Header.vue";
  import Balance from "./components/Balance.vue";
  import IncomeExpenses from "./components/IncomeExpenses.vue";
  import TransactionList from "./components/TransactionList.vue";
  import AddTransaction from "./components/AddTransaction.vue";

  const toast = useToast();

  const transactions = ref([]);

  onMounted(() => {
    const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

    if(savedTransactions){
      transactions.value = savedTransactions;
    }
  });

  // Get total
  const total = computed(() => {
    return transactions.value.reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
  });

  // Get income
  const income = computed(() => {
    return transactions.value
      .filter((transaction) => transaction.amount > 0)
      .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
      .toFixed(2);
  });
  
  // Get expenses
  const expenses = computed(() => {
    return transactions.value
      .filter((transaction) => transaction.amount < 0)
      .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
      .toFixed(2);
  });
   
   // Add transaction
   const handleTransactionSubmitted = (transactionData) => {
     // console.log(transactionData);
     
     transactions.value.push({
       id: generateUniqueId(),
       text: transactionData.text,
       amount: transactionData.amount,
     });

     savedTransactionsToLocalStorage();

     // console.log(generateUniqueId())
     toast.success('Transaction added');
   };

   // Generate unique ID
   const generateUniqueId = () => Math.floor(Math.random() * 1000000);

   // Delete transaction
   const handleTransactionDeleted = (id) => {
    // console.log(id);
     transactions.value = transactions.value.filter(
      (transaction) => transaction.id !== id
     );

     savedTransactionsToLocalStorage();

     toast.success('Transaction deleted');
   }

   // Save to localstorage
   const savedTransactionsToLocalStorage = () => {
    localStorage.setItem('transactions',JSON.stringify(transactions.value))
   }
</script>