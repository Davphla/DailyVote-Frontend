<template>
  <div>
    <button @click="connectWallet">Connect Wallet</button>
    <div v-if="account">
      Connected account: {{ account }}
      <button @click="castVote(0, 0)">Vote for Option 0</button>
      <button @click="castVote(0, 1)">Vote for Option 1</button>
      <div v-if="response">
        {{ response }}

      </div>
    </div>
  </div>
</template>

<script lang="ts">

import Web3, { Contract } from 'web3';
import { MetaMaskInpageProvider } from "@metamask/providers";
import MyContractABI from '@/abis/Proposal.json';
import { ref } from 'vue';


declare global {
  interface Window {
    ethereum: MetaMaskInpageProvider
  }
}


export default {
  data() {
    return {
      address: '0x4f0d42bC7903dd3b41De839C694E2910D99657c8',
      account: null as string | null,
      web3: null as Web3 | null,
      contract: null as Contract<typeof MyContractABI.abi> | null,
      isOwner: false,
      response: ref(),
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
          this.contract = new this.web3.eth.Contract(MyContractABI.abi, this.address);

        } catch (error) {
          console.error("User denied account access" + error);
        }
      } else {
        console.error("No Ethereum provider found. Install MetaMask.");
      }
    },


    async castVote(proposalId: number, optionId: number) {
      if (!this.contract || !this.account) {
        console.log("Contract or account not initialized");
        return;
      }
      try {
        this.response.value = await this.contract.methods.castVote(proposalId, optionId).send({ from: this.account });
        console.log("Vote cast successfully");
      } catch (error) {
        console.error("Error casting vote: " + error);
      }
    },

    async createProposal(name: string, params: Array<string>) {
      if (!this.contract || !this.account) {
        console.log("Contract or account not initialized")
        return;
      }

      try {
        this.response.value = await this.contract.methods.createProposal(name, params).send({ from: this.account });
        console.log("Proposal created successfully");
      } catch (error) {
        console.error("Error creating proposal: " + error);
      }
    },

    async closeProposal(proposalId: number) {
      if (!this.contract || !this.account) {
        console.log("Contract or account not initialized")
        return;
      }
      try {
        this.response.value = await this.contract.methods.closeProposal(proposalId).send({ from: this.account });
        console.log("Proposal closed successfully");
      } catch (error) {
        console.error("Error closing proposal: " + error);
      }
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
