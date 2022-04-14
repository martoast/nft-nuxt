<template>
  <div class="bg-dark" style="height: 100vh">
    <b-navbar toggleable>
      <b-navbar-brand class="text-white" href="#">Logo</b-navbar-brand>

      <p
        v-if="initialState.account && initialState.smartContract"
        class="text-white"
        style="text-overflow: ellipsis"
      >
        ...{{ initialState.truncated_account }}
      </p>

      <b-btn v-else @click="connectWallet">Connect wallet</b-btn>
    </b-navbar>
  </div>
</template>

<script>
import abi from "~/static/abi.json";
import config from "~/static/config.json";
import Web3EthContract from "web3-eth-contract";
import Web3 from "web3";
export default {
  data() {
    return {
      ethereum: null,
      mintAmount: 1,
      metamaskIsInstalled: false,
      initialState: {
        loading: false,
        account: null,
        truncated_account: null,
        smartContract: null,
        web3: null,
        errorMsg: "",
      },
    };
  },
  mounted() {
    this.ethereum = window.ethereum;
    this.metamaskIsInstalled = this.ethereum && this.ethereum.isMetaMask;
  },
  methods: {
    async getBalance() {
      await this.initialState.smartContract.methods
        .balanceOf(this.initialState.account)
        .call(function (err, res) {
          if (err) {
            console.log("An error occured", err);
            return;
          } else {
            return res;
          }
        });
    },
    async connectWallet() {
      if (this.metamaskIsInstalled) {
        Web3EthContract.setProvider(ethereum);
        this.initialState.web3 = new Web3(ethereum);

        try {
          const accounts = await this.ethereum.request({
            method: "eth_requestAccounts",
          });
          const networkId = await this.ethereum.request({
            method: "net_version",
          });

          this.initialState.account = accounts[0];

          this.initialState.truncated_account =
            this.initialState.account.substring(38);

          if (networkId == config.NETWORK.ID) {
            const SmartContractObj = new Web3EthContract(
              abi,
              config.CONTRACT_ADDRESS
            );

            this.initialState.smartContract = SmartContractObj;

            console.log(this.initialState);

            // Add listeners start
            this.ethereum.on("accountsChanged", (accounts) => {
              dispatch(updateAccount(accounts[0]));
            });
            this.ethereum.on("chainChanged", () => {
              window.location.reload();
            });
            // Add listeners end
          } else {
            alert("Change to the correct network.");
          }
        } catch (e) {
          console.error(e);
        }
      } else {
        alert("metamask is not installed.");
      }
    },
  },
};
</script>
