<template>
  <div>
    <!-- PAGE HEADER START -->
    <div class="flex items-center justify-between">
      <h1 class="font-bold text-2xl">Transações</h1>

      <!-- TOGGLE TRANSACTION ADD START -->
      <AppButton @click="isAdding = !isAdding"> Nova transação </AppButton>
      <!-- TOGGLE TRANSACTION ADD END -->
    </div>
    <!-- PAGE HEADER END -->

    <!-- TRANSACTION ADD FORM START -->
    <transactionAdd
      v-if="isAdding"
      @after-add="afterAdd"
      @cancel="isAdding = false"
    />
    <!-- TRANSACTION ADD FORM END -->

    <!-- TRANSACTION FILTER FORM START -->
    <TransactionFilter @filter="onFilter" />
    <!-- TRANSACTION FILTER FORM START -->

    <!-- TRANSACTION ITEM LIST START -->
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
    <!-- TRANSACTION ITEM LIST END -->
  </div>
</template>

<script>
import { groupBy, orderBy } from "lodash";
import Transaction from "~/components/transactions/Transaction.vue";
import transactionAdd from "~/components/transactions/transactionAdd.vue";
import AppButton from "~/components/Ui/AppButton";
import AppFormInput from "~/components/Ui/AppFormInput";
import AppFormLabel from "~/components/Ui/AppFormLabel";
import AppFormSelect from "~/components/Ui/AppFormSelect";
import TransactionFilter from "~/components/transactions/TransactionFilter.vue";

export default {
  name: "IndexPage",

  components: {
    Transaction,
    transactionAdd,
    AppButton,
    AppFormInput,
    AppFormLabel,
    AppFormSelect,
    TransactionFilter,
  },

  // FETCH DATA FROM STORE, WHERE THE DB INJECT DATA
  async asyncData({ store }) {
    return {
      transactions: await store.dispatch("transactions/getTransactions"),
    };
  },

  data() {
    return {
      isAdding: false, //TOGGLE IDENTIFIER
    };
  },

  computed: {
    transactionsGrouped() {
      //ORDER RULE
      return groupBy(orderBy(this.transactions, "date", "desc"), "date");
    },
  },

  methods: {
    formateDate(date) {
      //DATE FORMAT RULE
      return this.$dayjs(date).format("DD/MM/YYYY");
    },
    afterAdd(transaction) {
      //PUSH DATA TO DB
      this.transactions.push(transaction);
    },
    onUpdate(transaction) {
      //UPDATE DATA ON DB AND AUTO INSERT DATA ON LIST
      const idx = this.transactions.findIndex((o) => o.id === transaction.id);
      this.transactions.splice(idx, 1, transaction);
    },
    onFilter(filter) {
      //FILTER RULE
      this.$store
        .dispatch("transactions/getTransactions", filter)
        .then((response) => {
          this.transactions = response;
        });
    },
  },
};
</script>
