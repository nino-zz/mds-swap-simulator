<template>
  <div class="my-component">
    <div class="header">
      <p class="naslov">{{ title }}</p>
      <p class="podnaslov">{{ desc }}</p>
    </div>

    <div class="form-group">
      <h5 style="font-weight: bold; text-align: left;">Perform a Swap</h5>

      <div class="input-group mb-3">
        <label for="fromSymbol" class="form-label tekst1">From (Symbol):</label>
        <div class="dropdown w-100">
          <button
            class="btn btn-light dropdown-toggle w-100"
            type="button"
            id="dropdownMenuButton1"
            data-bs-toggle="dropdown"
            aria-expanded="false"
          >
            {{ selectedSymbolFrom || '- Select -' }}
          </button>
          <ul class="dropdown-menu w-100" aria-labelledby="dropdownMenuButton1">
            <li v-for="symbol in symbolsFrom" :key="symbol">
              <a class="dropdown-item" href="#" @click="selectSymbol(symbol)">
                {{ symbol }}
              </a>
            </li>
          </ul>
        </div>
      </div>

      <!-- Drugi dropdown -->
        <div class="input-group mb-3">
          <label for="toSymbol" class="form-label tekst1">To (Symbol):</label>
          <div class="dropdown w-100">
            <button
              class="btn btn-light dropdown-toggle w-100"
              type="button"
              id="dropdownMenuButton2"
              data-bs-toggle="dropdown"
              aria-expanded="false"
            >
              {{ selectedSymbolTo || '- Select -' }}
            </button>
            <ul class="dropdown-menu w-100" aria-labelledby="dropdownMenuButton2">
              <li v-for="symbol in symbolsTo" :key="symbol">
                <a class="dropdown-item" href="#" @click="selectSymbolTo(symbol)">
                  {{ symbol }}
                </a>
              </li>
            </ul>
          </div>
        </div>

      <div class="input-group mb-3">
        <label for="amount" class="form-label tekst1">Amount to Swap (units of 'From' symbol):</label>
      </div>

      <input
        type="number"
        ref="amount"
        class="form-control"
        placeholder="Enter amount"
      />

      <button type="button" class="btn btn-success w-100 mt-3" @click="performSwap">
        Swap
      </button>

    </div>

    <div class="results mt-4">
      <p class="tekst1 result">
        {{ swapResult }}
      </p>
    </div>

    <div class="user-selection">
      <label for="userSelect" class="form-label" style="font-weight: bold;">Select User:</label>
      <div class="dropdown w-100">
        <button
          class="btn btn-light dropdown-toggle w-100"
          type="button"
          id="dropdownMenuButton1"
          data-bs-toggle="dropdown"
          aria-expanded="false"
        >
          {{ currentUser || '- Select -' }}
        </button>
        <ul class="dropdown-menu w-100" aria-labelledby="dropdownMenuButton1">
          <li v-for="(user, index) in users" :key="index">
            <a class="dropdown-item" href="#" @click="currentUser = user">
              {{ user }}
            </a>
          </li>
        </ul>
      </div>
    </div>
    

    <div class="za-ispis d-flex">
      <div class="balances card shadow-sm me-3">
          <div class="card-body">
            <h5 class="card-title text-center mb-4">Your Balances ({{ currentUser }})</h5>
            <ul class="list-group list-group-flush">
              <li
                class="list-group-item"
                v-for="(balance, symbol) in userBalances[currentUser]"
                :key="symbol"
              >
                {{ symbol }}: {{ balance.toFixed(2) }}
              </li>
            </ul>
          </div>
      </div>

      <div class="mds-holders card shadow-sm">
        <div class="card-body">
          <h5 class="card-title">MDS Holders</h5>
          <ul class="list-group list-group-flush">
            <li
              class="list-group-item"
              v-for="(data, user) in userBalances"
              :key="user"
            >
              {{ user }}: {{ data.MDS.toFixed(2) }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "SwapComponent",
  data() {
    return {
      title: "Stock Swap Simulator with MDS Fee Distribution",
      desc: "Swap between AAPL, TSLA, AMZN, and MDS. A 0.01% fee is charged on each swap (in USD), then converted to MDS (1 MDS = $0.01) and distributed evenly among 5 total holders (including you).",
      symbolsFrom: ["AAPL", "TSLA", "AMZN", "MDS"], 
      symbolsTo: ["AAPL", "TSLA", "AMZN", "MDS"],   
      selectedSymbolFrom: null, 
      selectedSymbolTo: null,  
      swapResult: null,

      users: ["user1", "user2", "user3", "user4", "user5"], 
      currentUser: "user1",

      userBalances: {
        user1: {
          AAPL: 50,
          TSLA: 20,
          AMZN: 80,
          MDS: 20000000,
        },
        user2: {
          AAPL: 10,
          TSLA: 10,
          AMZN: 40,
          MDS: 20000000,
        },
        user3: {
          AAPL: 30,
          TSLA: 15,
          AMZN: 60,
          MDS: 20000000,
        },
        user4: {
          AAPL: 40,
          TSLA: 30,
          AMZN: 100,
          MDS: 20000000,
        },
        user5: {
          AAPL: 25,
          TSLA: 50,
          AMZN: 90,
          MDS: 20000000,
        }
      },

      stockPrices: {
        AAPL: 15, 
        TSLA: 70, 
        AMZN: 30, 
        MDS: 0.01,
      },
    };
  },

  methods: {
    selectSymbol(symbol) {
      this.selectedSymbolFrom = symbol;
    },
    selectSymbolTo(symbol) {
      this.selectedSymbolTo = symbol;
    },
    
    performSwap() {
      if (!this.selectedSymbolFrom || !this.selectedSymbolTo) {
        alert("Please select both 'From' and 'To' symbols.");
        return;
      }
      if (this.selectedSymbolFrom === this.selectedSymbolTo) {
        alert("'From' and 'To' symbols cannot be the same.");
        return;
      }
      
      const amount = parseFloat(this.$refs.amount.value);
      if (isNaN(amount) || amount <= 0) {
        alert("Please enter a valid amount.");
        return;
      }

      const userBalance = this.userBalances[this.currentUser];

      if (amount > userBalance[this.selectedSymbolFrom]) {
        alert("Insufficient balance for the selected symbol.");
        return;
      }

      const fromPrice = this.stockPrices[this.selectedSymbolFrom]; 
      const toPrice = this.stockPrices[this.selectedSymbolTo];    

      const fromValue = amount * fromPrice; 
      const toAmount = fromValue / toPrice; 

      const fee = 0.01 * fromValue; 
      const mdsAmount = fee / 0.01; 

      console.log(`Amount: ${amount}, FromValue: ${fromValue}, ToAmount: ${toAmount}, Fee: ${fee}, MDSAmount: ${mdsAmount}`);

      userBalance[this.selectedSymbolFrom] -= amount;
      userBalance[this.selectedSymbolTo] += toAmount - fee;

      console.log(`User balance before swap:`, userBalance);

      const users = Object.keys(this.userBalances);
      const distributedFee = mdsAmount / users.length;

      console.log(`Distributed MDS Fee per user: ${distributedFee}`);

      users.forEach((user) => {
        this.userBalances[user].MDS += distributedFee;
      });

      console.log(`User balances after swap and distribution:`, this.userBalances);

      alert(
        `Swapped ${amount.toFixed(2)} ${this.selectedSymbolFrom} for ${toAmount.toFixed(2)} ${this.selectedSymbolTo}. Fee of $${fee.toFixed(2)} converted to MDS and distributed.`
      );

      this.swapResult = `Swapped ${amount.toFixed(2)} ${this.selectedSymbolFrom} for ${toAmount.toFixed(2)} ${this.selectedSymbolTo}. Fee of $${fee.toFixed(2)} converted to MDS and distributed.`;
    },
  },
};
</script>

<style scoped>
.my-component {
  padding: 20px;
  border-radius: 10px;
  max-width: 600px;
  margin: auto;
  background-color: #f9f9f9;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  font-family: Arial, sans-serif;
}

.header .naslov {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
}

.header .podnaslov {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 20px;
  text-align: center;
}

.result {
  font-size: 0.9rem;
  color: #333;
  background-color: #eef8f2;
  padding: 10px;
  border-radius: 5px;
}

.btn {
  font-size: 1rem;
}

.input-group .form-label {
  font-size: 0.9rem;
  margin-bottom: 5px;
}

.dropdown-menu {
  max-height: 200px;
  overflow-y: auto;
}

.tekst1{
  font-weight: bold;
}

.za-ispis {
  display: flex; 
  gap: 20px; 
  margin-top: 20px;
}

.card {
  width: 50%;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #fff;
}

.card-body {
  padding: 15px;
}

.card-title {
  font-size: 1.2rem;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
}

.list-group-item {
  font-size: 0.9rem;
  font-weight: 500;
  padding: 8px 10px;
}

.list-group-item:nth-child(odd) {
  background-color: #f9f9f9; 
}

.shadow-sm {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); 
}

.btn.dropdown-toggle.selected {
  background-color: #d4edda; 
}

.btn.dropdown-toggle {
  background-color: #f5f5f5;
  font-weight: bold;

  border: 2px solid #ddd; 
  border-radius: 5px;     
  padding: 8px 12px;      
}

.btn.dropdown-toggle:hover,
.btn.dropdown-toggle:focus {
  border-color: #28a745; 
  box-shadow: 0 0 5px rgba(40, 167, 69, 0.5); 
}

.dropdown-menu {
  border: 2px solid #ddd; 
  border-radius: 5px;     
  padding: 5px 0;        
}

</style>
