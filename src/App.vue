<template>
  <div class="container my-5">
    <div class="row">
      <div class="col-md-6">
        <label>From</label>
        <select v-model="from" class="form-control">
          <option :value="currency.id" v-for="(currency, index) in formattedCurrencies" :key="index">{{currency.currencyName}}</option>
        </select>
      </div>
      <div class="col-md-6">
        <label>To</label>
        <select v-model="to" id="" class="form-control">
          <option :value="currency.id" v-for="(currency, index) in formattedCurrencies" :key="index">{{currency.currencyName}}</option>
        </select>
      </div>
    </div>  
    <div class="row">
      <div class="col-md-6 offset-md-3">
        <input type="text" placeholder="Amount" v-model="amount" class="form-control my-5">
        <div class="text-center">
          <button class="btn btn-lg btn-primary" @click="convertCurrency" :disabled="amount == 0 || !amount || loading">
            {{loading ? 'Converting ...' : 'Convert'}}
          </button>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-6 offset-md-3">
        <h1 class="text-center display-2 mt-5">{{calculateResult}}</h1>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      currencies: {},
      amount: 0,
      from: 'EUR',
      to: 'USD',
      result: 0,
      loading: false,
      calculateResult: 0
    };
  },
  created() {
    this.getCurrencies();
  },
  methods: {
    getCurrencies() {
      const currencies = localStorage.getItem('currencies');

      if(currencies) {
        this.currencies = JSON.parse(currencies);

        return;
      }

      axios('https://free.currconv.com/api/v7/currencies?apiKey=554bdbc3e60331625b7b')
      .then(response => {
        this.currencies = response.data.results;
        localStorage.setItem('currencies', JSON.stringify(response.data.results));
      });
    },
    convertCurrency() {
      const key = `${this.from}_${this.to}`;
      this.loading = true;

      axios(`https://free.currconv.com/api/v7/convert?q=${key}&apiKey=554bdbc3e60331625b7b`)
        .then(response => {
          this.loading = false;
          this.result = response.data.results[key].val;
          this.calculateResult = (Number(this.amount) * this.result).toFixed(3);
        });
    }
  },
  computed: {
    formattedCurrencies() {
      return Object.values(this.currencies);
    }
  },
  watch: {
    from() {
      this.result = 0;
    },
    to() {
      this.result = 0;
    }
  }
}
</script>

<style>
* {
  border-radius: 0 !important;
}

body {
  background-color: #e7f7f9;
  font-family: 'Poppins', sans-serif;
}
</style>
