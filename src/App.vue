<template>
  <v-main id="home" data-app>
    <div class="card">
      <h3>BlockX Airdrop</h3>
      <v-col cols="10" class="input-box">
        <v-text-field
          v-model="address"
          label="Recipient's address"
          required
          rounded
          dark
          color="success"
          dense
        >
        </v-text-field>
        <div class="button-box">
          <v-btn
            class="claim-btn"
            @click="toggleDialog(true)"
            dark
            color="red"
            rounded
          >
            CLAIM
          </v-btn>
          <v-btn
            class="claim-btn"
            dark
            color="red"
            rounded
            v-if="!connected"
            @click="connectWallet"
          >
            CONNECT
          </v-btn>
        </div>
      </v-col>
    </div>
    <v-dialog v-model="dialog" max-width="450">
      <v-card>
        <v-card-title class="text-h5">
          Confirm
        </v-card-title>

        <v-card-text>
          The {{ address }} will receive {{ amount }} BCX tokens. Click Ok to
          Confirm
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn color="green" text @click="dialog = false"> Cancel </v-btn>

          <v-btn color="green" v-if="amount !== '0.0'" text @click="claim"> OK </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-main>
</template>

<script>
const Web3 = require("web3");
const web3 = new Web3(window.ethereum);
const { address, abi } = require("./config");

export default {
  name: "App",
  mounted() {
    this.checkConnectedWalletExist();
  },
  data: () => ({
    address: "",
    isValidAddress: true,
    connected: false,
    dialog: false,
    amount: "",
  }),
  methods: {
    checkConnectedWalletExist: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Make sure you have metamask!");
          return false;
        } else {
          console.log("We have the ethereum object", ethereum);
        }
        const accounts = await ethereum.request({ method: "eth_accounts" });
        if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          this.currentAccount = account;
          this.connected = true;
          return true;
        } else {
          console.log("No authorized account found");
          return false;
        }
      } catch (error) {
        console.log(error);
        return false;
      }
    },
    connectWallet: async function () {
      try {
        const { ethereum } = window;
        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }
        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });
        console.log("Connected", accounts[0]);
        this.currentAccount = accounts[0];
        this.connected = true;
      } catch (error) {
        console.log(error);
      }
    },
    toggleDialog: async function (open) {
      if (open && !web3.utils.isAddress(this.address)) {
        return alert("The address that you entered is invalid!!!");
      }
      // const chainId = window.ethereum.chainId;
      // if (Number(chainId) !== 56)
      //   return alert("Please switch your Metamask network to Binance Smart Chain Mainnet");
      const provider = new this.$ethers.providers.Web3Provider(
        window.web3.currentProvider
      );
      const signer = provider.getSigner();
      const airdropContract = new this.$ethers.Contract(address, abi, signer);
      const reward = await airdropContract.rewards(this.address);
      
      this.amount = this.$ethers.utils.formatEther(reward)
      this.dialog = open;
    },
    claim: async function () {
      this.dialog = false;
      // console.log(await airdropContract.owner())
      try {
        const provider = new this.$ethers.providers.Web3Provider(
          window.web3.currentProvider
        );
        const signer = provider.getSigner();
        const airdropContract = new this.$ethers.Contract(address, abi, signer);
        const tx = await airdropContract.claim(this.address);
        await tx.wait();
        this.amount = ""
      } catch (e) {
        // console.log(e.data ? e.data.message : e.message);
        return alert(e.data ? e.data.message : e.message);
      }
    },
  },
};
</script>

<style scoped style="scss">
#home {
  background-image: url("~@/assets/background.jpg");
  /* background-color: khaki; */
  width: 100%;
  height: 100vh;
  background-size: 100% 100%;
  background-repeat: no-repeat;
  display: flex;
  justify-content: center;
  align-items: center;
}

.card {
  background-color: #210065;
  border-radius: 20px;
  width: 500px;
  height: 300px;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  padding-top: 50px;
  margin: auto;
}

.card h3 {
  color: white;
  text-align: center;
  margin-top: 10px;
}

.input-box {
  margin: 20px auto;
  padding-top: 20px;
  background-color: #1a0051;
  border: 1px solid #1a0051;
  border-radius: 20px;
  width: 450px;
  height: 70px;
  color: white;
}

.card p {
  margin: 10px 0px 0px 20px;
}

.claim-btn {
  margin-top: 10px;
  background-color: blue !important;
}

.button-box {
  margin-top: 24px;
  display: flex;
  gap: 8px;
  /* justify-content: flex-end; */
  flex-direction: row-reverse;
}
</style>
