
<template>
  <div id="addTradeContainer">
    <form id="userForm">
      <h1 class="titleOfDiv">Add Trade</h1>

      <!-- Add Coins Form -->
      <input
        type="text"
        id="ticker1"
        required="yes"
        placeholder="Input Ticker Code"
      />
      <br /><br />
      <input
        type="number"
        id="quant1"
        required="yes"
        placeholder="Input Trade Quantity"
        v-model="tradeQuantity"
      />
      <br /><br />
      <input
        type="number"
        id="buy1"
        required="yes"
        placeholder="Input Price Per Trade"
        v-model="pricePerTrade"
      />
      <br /><br />

      <h2 id="tradecost">Total: SGD {{ tradeCost.toFixed(2) }}</h2>

      <button id="savebutton" type="button" @click="savetofs()">+ Add</button
      ><br /><br />
    </form>

    <div id="profitContainer">
      <h3>Expected Return</h3>
      <h1 id="totalProfit" v-if="isNaN(totalPL)">Loading..</h1>
      <h1 id="totalProfit" v-else>
        SGD$
        {{
          totalPL.toLocaleString(undefined, {
            useGrouping: true,
            minimumFractionDigits: 2,
            maximumFractionDigits: 2,
          })
        }}
      </h1>
    </div>
  </div>

  <Loading ref="Loading" />
</template>
  
  <script>
import { getAuth, onAuthStateChanged } from "firebase/auth";
import axios from "axios";
import Loading from "@/components/Loading.vue";

export default {
  components: {
    Loading,
  },

  data() {
    return {
      tickerName: "",
      tradeQuantity: null,
      pricePerTrade: null,
      totalCost: 0,
      stockPrice: [],

      // User info
      useremail: "",
    };
  },

  props: {
    totalPL: Number,
  },

  async mounted() {
    const auth = getAuth();
    onAuthStateChanged(auth, (user) => {
      if (user) {
        this.useremail = user.email;
      } else {
        this.useremail = ""; // Ensure it's cleared when the user signs out
      }
    });
  },

  computed: {
    // Calculate the trade profit based on tradeQuantity and pricePerTrade
    tradeCost() {
      return (this.tradeQuantity || 0) * (this.pricePerTrade || 0);
    },
  },

  methods: {
    // Save to Firebase
    async savetofs() {
      // Get input from placeholder
      let ticker = document.getElementById("ticker1").value.toUpperCase();
      let buyPrice = document.getElementById("buy1").value;
      let buyQuantity = document.getElementById("quant1").value;

      //Define valid input
      if (isNaN(parseFloat(buyPrice)) || isNaN(parseFloat(buyQuantity))) {
        alert("Buy Price or Buy Quantity must be valid numbers."); //TODO: Add PopUp Window
        return;
      } else if (
        ticker.trim() === "" ||
        buyPrice.trim() === "" ||
        buyQuantity.trim() === ""
      ) {
        alert("All fields must be filled out.");
        return;
      } else if (buyPrice <= 0) {
        alert("Buy Price must be more than 0");
      }

      // Add Trade
      this.$refs.Loading.onLoading();
      this.stockPrice = [];
      try {
        const apiCheckValidTickerUrl = `https://smartfolio-7gt75z5x3q-as.a.run.app/api/yfinance/curentPrice/${ticker}`;
        const price = await axios.get(apiCheckValidTickerUrl);
        this.stockPrice = [ticker, price.data];
      } catch (error) {
        alert("Please enter a valid ticker!");
        this.$refs.Loading.offLoading();
        return;
      }

      const tradeData = {
        ticker: ticker,
        buyQty: parseFloat(buyQuantity),
        buyPrice: parseFloat(buyPrice),
      };

      const apiUrl = `https://smartfolio-7gt75z5x3q-as.a.run.app/api/update/updateTrade/${this.useremail}`;

      try {
        await axios.put(apiUrl, tradeData);

        this.$refs.Loading.offLoading();
      } catch (error) {
        alert("Error: " + error.response.data);
      }

      // Reset placeholder
      document.getElementById("userForm").reset();
      this.tradeQuantity = null;
      this.pricePerTrade = null;

      this.$emit("added", true);
      console.log("Added: ", ticker);
    },
  },
  emits: ["added"],
};
</script>




<style scope>
#addTradeContainer h1 {
  margin-top: 0;
}

#addTradeContainer {
  position: absolute;
  margin-top: -6.5%;
  width: 25.5%;
}

/* Add Trades */
.titleOfDiv {
  font-size: 2.5vw;
}

#userForm {
  text-align: center;
  background-color: white;
  width: 100%;
  height: 28vw;
  border-radius: 25px;
  box-shadow: 1px 8px 10px rgba(0, 0, 0, 0.3);
  padding-top: 14%;
}

input {
  background-color: #f3f3f3;
}

#userForm input[type="text"],
#userForm input[type="number"] {
  width: 70%;
  height: 7%;
  margin: 1.2% 0;
  font-size: 1.25vw;
  font-weight: 600;
  padding-left: 4%;
  border-radius: 5px;
  border: none;
  text-align: left;
}

input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0%;
}

input::placeholder {
  color: #c2c1c1;
  text-align: left;
}

#tradecost {
  text-align: center;
  margin-top: 1%; /* Adjust the margin-top value to move it closer to the input box */
  margin-bottom: 4%;
  font-weight: bold; /* Make the text bold if needed */
  font-size: 2vw;
}

#savebutton {
  width: 45%;
  height: 18%;
  background-color: #272f51;
  color: white;
  font-size: 1.8vw;
  font-weight: 700;
  border-radius: 20px;
  cursor: pointer;
}

#savebutton:hover {
  background-color: #475281; /* Change the button color on hover */
}

/* Total Profit */
#profitContainer {
  background-color: #3a446e;
  color: white;
  width: 96%;
  border-radius: 1.5vh;
  text-align: right;
  padding-right: 5%;
  padding-top: 0.1%;
  padding-bottom: 0.1%;
  margin-top: 4%;
}

#profitContainer h3 {
  margin-bottom: 0vw;
  font-size: 1.5vw;
  color: rgb(222, 222, 222);
  margin-top: 4%;
}

#totalProfit {
  color: #02dbfd;
  font-size: 5vh;
}
</style>

  