<template>
  <div>
    <div class="d-block d-md-none">
      <h1>Not on mobile bitch</h1>
    </div>
    <div class="d-none d-md-block">
      <div class="book">
        <a @click="onPreviousPage()" class="book__page book__page--1">
          <img class="img-left" src="/images/1.png" alt="" />
        </a>

        <a @click="onNextPage()" class="book__page book__page--2">
          <img class="img-right" src="/images/2.png" alt="" />
        </a>
      </div>
    </div>
    <b-modal
      id="wallet-modal"
      centered
      hide-header
      hide-header-close
      hide-footer
      no-close-on-backdrop
      no-close-on-esc
      size="lg"
    >
      <b-container>
        <div class="text-center">
          <h2 class="py-3" style="font-family: 'Market'; font-size: 42px">
            The Comic Book
          </h2>
          <p style="font-family: 'Champange'; font-size: 22px" class="pb-3">
            To view this content you must connect your wallet to verify as
            holder.
          </p>
          <b-btn
            v-if="!initialState.account"
            @click="connectWallet"
            style="font-family: 'Market'; background-color: black"
            >Connect wallet</b-btn
          >

          <b-btn
            v-else-if="initialState.amount_holding > 0"
            @click="onEnterComic"
            style="font-family: 'Market'; background-color: black"
          >
            Enter The Heist
          </b-btn>

          <h2
            v-else-if="initialState.amount_holding == 0"
            style="font-family: 'Market'"
          >
            Must be a holder to enter.
          </h2>

          <div v-else>
            <b-spinner label="Spinning"></b-spinner>
          </div>
        </div>
      </b-container>
    </b-modal>
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
      metamaskIsInstalled: false,
      left_page: null,
      left_img: null,
      right_page: null,
      right_img: null,
      current_page: 1,
      initialState: {
        loading: false,
        account: null,
        amount_holding: null,
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

    this.left_img = document.querySelector(".img-left");

    this.left_page = document.querySelector(".book__page--1");

    this.right_img = document.querySelector(".img-right");

    this.right_page = document.querySelector(".book__page--2");

    this.$bvModal.show("wallet-modal");
  },

  methods: {
    async getBalance() {
      const response = await this.initialState.smartContract.methods.balanceOf(
        this.initialState.account
      );

      let call = await response.call();

      return parseInt(call);
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

            await this.getBalance().then((amount) => {
              setTimeout(() => {
                this.initialState.amount_holding = amount;
              }, 1000);
            });
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

    onEnterComic() {
      this.$bvModal.hide("wallet-modal");
    },

    onNextPage() {
      console.log(this.right_page);

      this.current_page += 1;

      this.right_page.style.transform =
        "0.9s cubic-bezier(0.645, 0.045, 0.355, 1)";

      this.right_page.style.transform = "rotateY(-180deg)";

      // this.right_img.src = "/images/" + this.current_page + ".png";
    },

    onPreviousPage() {
      console.log(this.left_page.src);

      this.current_page -= 1;

      this.left_img.src = "/images/" + this.current_page + ".png";

      this.left_page.style.transform =
        "0.9s cubic-bezier(0.645, 0.045, 0.355, 1)";

      // this.left_page.style.transform = "rotateY(180deg)";
    },
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: "Champange";
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url("~assets/fonts/Champagne.ttf") format("truetype");
}

@font-face {
  font-family: "Market";
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url("~assets/fonts/Market.ttf") format("truetype");
}

.modal-backdrop {
  opacity: 0.9;
}

.book {
  display: flex;
  perspective: 1200px;

  &__page {
    position: relative;
    width: 50%;
    height: 100%;
    padding: 0;
    display: grid;
    transform: rotateY(0deg);
    transition: transform 0.9s cubic-bezier(0.645, 0.045, 0.355, 1);
    transform-origin: 0% 0%;
    &--1 {
      cursor: pointer;
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

      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }

    &--3 {
      cursor: pointer;
      position: absolute;
      right: 0;

      img {
        width: 100%;
        max-width: 100%;
        height: auto;
      }
    }
  }
}
</style>
