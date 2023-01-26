<script>
import { onMount } from "svelte";

const BASE_URL = "https://open.er-api.com/v6/latest"

let currencies = []
let amount = 1
let from = "EUR"
let to = 'USD'
let exchangeRates = {};
let currentRate= 0;
let isFlipped = false;
$: convertedAmount = (amount * currentRate).toFixed(3);

function switchCurrency() {
    isFlipped = !isFlipped;
    [from, to] = [to, from];

    Object.keys(exchangeRates).map(function(key) {
        exchangeRates[key] = 1 / exchangeRates [key];
    })
    currentRate = 1 / currentRate;
}

async function fetchRates(base, current) {
    isFlipped = false;
    const res = await fetch(`${BASE_URL}?base=${base}`);
    const data =  await res.json();

    exchangeRates = data.rates;
    if (base === "EUR") {
        exchangeRates["EUR"] = 1;
    }
    currentRate = data.rates[current]; 
}

function handleFromCurrencyChange(e) {
    if (isFlipped) {
        //pull from existing list
        currentRate = exchangeRates[from];
    } else {
        //fetch new list
        fetchRates(from, to);
    }
}

function handleToCurrencyChange(e) {
    if (isFlipped) {
        fetchRates(to, from);
    } else {
        currentRate = exchangeRates[to];
    }
}

function handleFocus() {
    this.select()
}

function handleInput(e) {
    if (e.target.name === "fromCurrency") {
      from = e.target.value.toUpperCase();
    } else {
      to = e.target.value.toUpperCase();
    }
  }

onMount(async () => {
    const res = await fetch(`${BASE_URL}?base=${from}`)
    const data = await res.json()
    currencies = [from, ...Object.keys(data.rates)].sort();
    exchangeRates = { ...data.rates, [from]: 1 };
    currentRate = exchangeRates[to];
});

</script>

<main class="flex items-center justify-center">
    <div
      class="flex flex-col w-full rounded-lg shadow bg-white sm:w-1/2 lg:max-w-xl"
    >
      <h1
        class="m-0 py-2 px-4 text-center font-bold text-3xl bg-white-700 rounded-t underline"
      >
       Currency Converter 💱
      </h1>
      <div class="flex flex-col items-center justify-center w-full pt-4">
        <label for="amountToConvert">Amount to convert:</label>
        <input
          id="amountToConvert"
          class="block py-2 px-3 leading-6 border border-gray-300 rounded"
          name="amountToConvert"
          type="number"
          min="0"
          placeholder="Amount"
          bind:value={amount} 
        />
      </div>
  
      <div class="flex items-center w-full h-full py-4 text-center">
        <div class="flex flex-col items-center w-3/5 px-4 sm:w-full">
          <label for="fromCurrency">Convert From:</label>
          <input
            id="fromCurrency"
            class="block w-full py-2 px-3 leading-6 border border-gray-300 rounded"
            name="fromCurrency"
            maxlength="3"
            value={from}
            on:input={handleInput}
            on:focus={handleFocus}
            on:change={handleFromCurrencyChange}
            list="fromCurrencyList"
          />
          <datalist id="fromCurrencyList">
            {#each currencies as option}
            <option>{option}</option>
            {/each}
          </datalist>
          
        </div>
  
        <button
          class="flex items-center justify-center m-0 bg-slate-100 border-2
          border-transparent pointer focus:border-gray-300"
          on:click={switchCurrency}
        >
         SWITCH 🔄
        </button>
  
        <div class="flex flex-col items-center w-3/5 px-4 sm:w-full">
          <label for="toCurrency">Convert To:</label>
          <input
            id="toCurrency"
            class="block w-full py-2 px-3 leading-6 border border-gray-300 rounded"
            name="toCurrency"
            maxlength="3"
            value={to}
            on:input={handleInput}
            on:focus={handleFocus}
            on:change={handleToCurrencyChange}
            list="toCurrencyList"
          />
          <datalist id="toCurrencyList">
            {#each currencies as option}
            <option>{option}</option>
            {/each}
          </datalist>
        </div>
      </div>
  
      <div class="flex flex-wrap justify-center w-full pb-2 mb-10">
        {#if (amount === 0 || amount === null || amount === '' || amount === undefined) || from === 0 || from === null || from === '' || from === undefined || to === 0 || to === null || to === '' || to === undefined}
          <p>Please enter a valid amount/currency.</p>
        {:else}
          <p>{amount} {from} = {convertedAmount} {to}</p>
        {/if}
      </div>
      
  
    </div>
  </main>


<style>
  :global(body) {
    padding-top: 4rem;
    background-color: #000000;
  }
</style>


