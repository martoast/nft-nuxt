<template>
  <div>
    <div class="d-block d-md-none">
      <h1>Not on mobile bitch</h1>
    </div>
    <div class="d-none d-md-block">
      <div class="book">
        <label for="page-1" class="book__page book__page--1">
          <div style="height: 100vh"></div>
        </label>

        <label for="page-2" class="book__page book__page--3">
          <img src="/images/CB-1.png" alt="" />
        </label>

        <!-- Resets the page -->
        <input type="radio" name="page" id="page-1" />

        <!-- Goes to the second page -->
        <input type="radio" name="page" id="page-2" />

        <label class="book__page book__page--2">
          <img src="/images/CB-2.png" alt="" />
        </label>
      </div>
    </div>
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

<style lang="scss">
.book {
  display: flex;
  perspective: 1200px;

  &__page {
    position: relative;
    width: 50%;
    height: 100%;
    display: grid;
    transform: rotateY(0deg);
    transition: transform 0.9s cubic-bezier(0.645, 0.045, 0.355, 1);
    transform-origin: 0% 0%;
    &--1 {
      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }

    &--2 {
      cursor: pointer;
      position: absolute;
      right: 0;
      pointer-events: none;

      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }

    &--3 {
      cursor: pointer;

      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }

    &--4 {
      cursor: pointer;

      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }
  }

  input[type="radio"] {
    display: none;

    &:checked + .book__page {
      transition: transform 0.9s cubic-bezier(0.645, 0.045, 0.355, 1);
      transform: rotateY(-180deg);
    }
  }
}
</style>
