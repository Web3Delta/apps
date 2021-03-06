<template>
  <div>
    <Card v-if="fetchedBlock && fetchedBlock.stateRoot"
      :nature="addCommas(fetchedBlock.number.toString())"
      :natureDesc="blockHash && blockHash.toString()"
      :type="fetchedBlock.parentHash.toString()"
      :extrinsicHash="fetchedBlock.extrinsicsRoot.toString()"
      :lifetime="fetchedBlock.stateRoot.toString()"
      :open="open"
    />
  </div>
</template>
<script lang="ts" >
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import Connector from '@vue-polkadot/vue-api';
import Card from '../shared/Card.vue';

@Component({
  components: {
    Card,
  }
})
export default class SingleBlockDetail extends Vue {
  private getBlock: any = null;
  private fetchedBlock: any = {}
  private blockHash: any = '';
  private subs: any[] = [];
  @Prop() public hash!: any;
  @Prop() public blockNumber!: any;
  @Prop({ default: false}) public open!: boolean;

  @Watch('$route.params.hash')
  public async singleMode(hash: any) {
    if (this.$parent.$options.name !== 'RecentBlocks') {
      this.loadExternalInfoByHash(hash);
    }
  }

  private addCommas(num: any) {
     return num.toString().replace(/\B(?<!\.\d*)(?=(\d{3})+(?!\d))/g, ',');
  }

  public async loadExternalInfoByBlockNumber(blockNumber: any) {
    const { api } = Connector.getInstance();
    this.subs.push(this.blockHash = await api.rpc.chain.getBlockHash((blockNumber)));
    this.subs.push(this.fetchedBlock = await api.rpc.chain.getHeader(this.blockHash));    
  }

  public async loadExternalInfoByHash(hash: any) {
    const { api } = Connector.getInstance();
    this.subs.push(this.fetchedBlock = await api.rpc.chain.getHeader(this.hash));
    this.subs.push(this.blockHash = await api.rpc.chain.getBlockHash((this.fetchedBlock.number)));
  }

  public async loadExternalInfoDefault() {
    const { api } = Connector.getInstance();
    this.subs.push(this.fetchedBlock = await api.rpc.chain.getHeader());
    this.subs.push(this.blockHash = await api.rpc.chain.getBlockHash());
  }

  public async mounted(): Promise<void> {
    if (this.hash) {
      this.loadExternalInfoByHash(this.hash);
    }
    if (this.blockNumber) {
      this.loadExternalInfoByBlockNumber(this.blockNumber);
    } else {
      this.loadExternalInfoDefault();
    }
  }
}
</script>
