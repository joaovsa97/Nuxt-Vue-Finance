<template>
  <div>
        <div class="flex items-center justify-between">
          <h1 class="font-bold text-2xl">
            Transações
          </h1>

          <AppButton @click="isAdding = !isAdding">
            Nova transação
          </AppButton>
        </div>

        <transactionAdd 
        v-if="isAdding"
        @after-add="afterAdd"
        @cancel="isAdding = false"
        />
        
        <TransactionFilter @filter="onFilter"/>

        <div
        v-for="(group, index) in transactionsGrouped"
        :key="index"
        class="my-4"
        >
          <div class="mb-1">
            <div class="font-bold text-sm">
              {{ formateDate(index) }}
            </div>
          </div>

          <div class="space-y-3">
            <Transaction 
            v-for="transaction in group"
            :key="transaction.id"
            :transaction="transaction"
            @update="onUpdate"
            />   
          </div>
        </div>
    </div>            
</template>

<script>
import { groupBy, orderBy } from 'lodash';
import Transaction from '~/components/transactions/Transaction.vue';
import transactionAdd from '~/components/transactions/transactionAdd.vue';
import AppButton from '~/components/Ui/AppButton';
import AppFormInput from '~/components/Ui/AppFormInput';
import AppFormLabel from '~/components/Ui/AppFormLabel';
import AppFormSelect from '~/components/Ui/AppFormSelect';
import TransactionFilter from '~/components/transactions/TransactionFilter.vue';

export default {
  name: 'IndexPage',

  components: {
    Transaction,
    transactionAdd,
    AppButton,
    AppFormInput,
    AppFormLabel,
    AppFormSelect,
    TransactionFilter,
  },

  async asyncData({store}) {
    return {
      transactions: await store.dispatch('transactions/getTransactions')
    }
  },

  data() {
    return {
      isAdding: false,
    }
  },

  computed: {
    transactionsGrouped() {
      return groupBy(orderBy(this.transactions, 'date', 'desc'), 'date');
    },
  },

  methods: {
    formateDate(date){
      return this.$dayjs(date).format('DD/MM/YYYY') 
    },
    afterAdd(transaction) {
      this.transactions.push(transaction);
    },
    onUpdate(transaction){
      const idx = this.transactions.findIndex(o => o.id === transaction.id);
      this.transactions.splice(idx, 1, transaction)
      // console.log(transaction);
    },
    onFilter(filter){
      this.$store.dispatch('transactions/getTransactions', filter)
      .then((response) => {
        this.transactions = response;
      })
    }
  }
}
</script>
