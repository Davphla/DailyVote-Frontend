<template>
  <div>
    <button @click="connectWallet">Connect Wallet</button>
    <div v-if="account">
      Connected account: {{ account }}
    </div>
  </div>
</template>

<script lang="ts">

import Web3 from 'web3';
import { MetaMaskInpageProvider } from "@metamask/providers";
import MyContractABI from '@/abis/Proposal.json';

declare global {
  interface Window{
    ethereum: MetaMaskInpageProvider
  }
}

export default {
  data() {
    return {
      account: null as string | null,
      web3: null as Web3 | null,
    };
  },
  methods: {
    async connectWallet() {
      if (window.ethereum) {
        try {
          await window.ethereum.request({ method: 'eth_requestAccounts' });
          this.web3 = new Web3(window.ethereum);
          const accounts = await this.web3.eth.getAccounts();
          this.account = accounts[0];
        } catch (error) {
          console.error("User denied account access" + error);
        }
      } else {
        console.error("No Ethereum provider found. Install MetaMask.");
      }
    },

    instanciateContract() {
      if (this.web3 == null) {
        console.error("Please connect to wallet");
        return;
      }
      const address = '0x4f0d42bC7903dd3b41De839C694E2910D99657c8';

      const contract = new this.web3.eth.Contract(MyContractABI.abi, address);

      contract.methods.createProposal("Hello", ["Red", "Blue"]).send({ from: this.account });


    },

  },
};
</script>

<style scoped>
button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}
</style>
