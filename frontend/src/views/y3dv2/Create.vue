<template>
  <LayoutY3DV2>
    <div class="create">
      <p class="title">Create A y3d Token</p>
      <div class="ui icon message" v-if="deploying">
        <i class="notched circle loading icon"></i>
        <div class="content">
          <div class="header">Please wait in patience</div>
          <p>Deploying your Y3D Token to the blockchain now.</p>
        </div>
      </div>
      <div class="ui positive message" v-if="deployedY3dToken">
        <i class="close icon"></i>
        <div class="header">Your y3dToken is ready.</div>
        <p>Contract Address: {{ deployedY3dToken }}</p>
        <button class="ui primary button" @click="goToY3dContract">Go Y3D Token page</button>
      </div>
      <form class="ui form">
        <div class="field create-field">
          <label>Underlying Token</label>
          <input
            type="text"
            placeholder="Enter Token Contract address..."
            v-model="tokenContract"
          />
        </div>
        <div class="field create-field">
          <label>yToken</label>
          <input
            type="text"
            placeholder="Enter yToken Contract address..."
            v-model="yTokenContract"
          />
        </div>
        <div class="field create-field">
          <label>Fee %</label>
          <input
            type="number"
            placeholder="Enter fee..."
            step="0.1"
            min="0"
            max="25.5"
            v-model="fee"
          />
        </div>
        <div class="field create-field">
          <label>Unknown</label>
          <input type="text" />
        </div>
        <div class="swap-button-content">
          <MainButton :btnLoading="false" @click="create_y3dToken">
            Create
          </MainButton>
        </div>
      </form>
    </div>
  </LayoutY3DV2>
</template>

<script>
import LayoutY3DV2 from '@/layouts/LayoutY3DV2.vue';
import MainButton from '@/components/MainButton.vue';
import { getProvider, utils } from '@/store/ethers/ethersConnect';
import { y3dFactory } from '@/contract';

/* eslint-disable no-alert */
export default {
  name: 'CreateToken',
  data: () => ({
    tokenContract: '',
    yTokenContract: '',
    fee: 0,
    deploying: false,
    deployedY3dToken: '',
  }),
  components: {
    LayoutY3DV2,
    MainButton,
  },
  computed: {},
  methods: {
    async create_y3dToken() {
      if (!utils.isAddress(this.tokenContract)) {
        alert('This is not a ethereum address, please double check your input.');
        return;
      }
      if (!utils.isAddress(this.yTokenContract)) {
        alert('This is not a ethereum address, please double check your input.');
        return;
      }
      if (this.fee < 0 || this.fee > 25.5) {
        alert('The fee is out of range');
        return;
      }
      this.deploying = true;
      const contract = y3dFactory.connect(getProvider().getSigner());
      try {
        const response = await contract.create(this.tokenContract);
        console.log('tx response', response);

        // Wait for 1 confirmation
        const receipt = await response.wait(1);
        console.log('tx receipt', receipt);
        const CreateY3dTokenEvent = receipt.events[1];
        console.log('CreateY3dTokenEvent', CreateY3dTokenEvent);
        const hex64ToAddress = (hex) => `0x${hex.slice(26)}`;
        const y3dToken = hex64ToAddress(CreateY3dTokenEvent.data);
        this.deploying = false;
        this.deployedY3dToken = y3dToken;
        console.log('y3dToken address: ', y3dToken);
      } catch (error) {
        alert(error.message);
        this.deploying = false;
      }
    },
    goToY3dContract() {
      this.$router.push({
        name: 'Y3DToken',
        params: { contractAddress: this.deployedY3dToken },
      });
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/assets/styles/color';
.create {
  margin-top: 30px;
  .title {
    color: white;
    font-weight: 500;
    font-size: 16;
  }
  .create-field {
    border-radius: 20px;
    border: 1px solid $y3d-border;
    background-color: $y3d-black;
    padding: 20px;
    label {
      color: $y3d-sub-title;
      margin-bottom: 10px;
    }
    input {
      background-color: transparent;
      padding-left: 0;
      padding-right: 0;
      caret-color: white;
      outline: none;
      height: 30px;
      color: white;
      border: 0;
      font-size: 20px;
    }
    input:focus {
      background-color: transparent;
      border: 0;
      color: white;
    }
  }
  .y3d-button {
    height: 56px;
    border-radius: 20px;
    width: 100%;
  }
}
</style>
