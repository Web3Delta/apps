<template>
  <div>
    <div v-if="entries && entries.validatorCount">
      <!-- <DisabledInput label="Validators" :value="entries.validatorCount" />  -->
      <DisabledInput label="Validators" :value="`${validators.length} / ${entries.validatorCount.toString()}`" /> 
      <div class="columns">
        <div class="column">
          <center><label><b>Epoch</b></label></center>
          <progressbar :value="parseInt(entries.sessionProgress)" :max="parseInt(entries.sessionLength)" show-value></progressbar>
        </div>
        <div class="column">
          <center><label><b>Era</b></label></center>
          <progressbar :value="parseInt(entries.eraProgress)" :max="parseInt(entries.eraLength)" show-value></progressbar>      
        </div>
      </div>
    </div>
    <br>
    <!-- verbose session informations -->
    <!-- <Collapse :open="false" title="Feel Cute 💝👇" :content="sessionResolved" /> -->
  </div>
</template>
<script lang="ts" >
import { Component, Prop, Vue, Watch, Emit } from 'vue-property-decorator';
import DisabledInput from '@/components/shared/DisabledInput.vue';
import Progressbar from '@/components/shared/ProgressBar.vue';
import Connector from '@vue-polkadot/vue-api';
import Collapse from '@/components/shared/Collapse.vue';

@Component({
  components: {
    DisabledInput,
    Progressbar,
    Collapse,
  }
})
export default class SummarySession extends Vue {
  private sessionData: any = {};
  private emitedLoaded: boolean = false;
  private sessionResolved: any = {};
  private stakingOverview: any = [];
  private validators: any = [];
  private entries: any = {};
  private subs: any[] = [];
  @Prop({default: 0}) public currentBlock!: number;
  
  @Watch('currentBlock')
  private resolve(): void {
    if (this.sessionData && this.sessionData.info) {
      const arr = Object.entries(this.sessionData.info)
    
      const a: any[] = []
      for (const [key, value] of arr) {
        a.push([key, (value as any).toString()]);
      }

      const m = new Map(a)    
      const obj = Array.from(m)
        .reduce((acc, [ key, val ]) => Object
        .assign(acc, { [key as string]: val }), {});
      
      this.sessionResolved = m
      this.entries = obj
      if (!this.emitedLoaded) {
        this.$emit('loadedSession')
        this.emitedLoaded = true
      }
    }
  }
  
  private async fetchSessionInfo() {
    const { api } = Connector.getInstance();
    this.subs.push(await api.derive.session.progress((value: any) => this.sessionData.info = value))
    this.subs.push(await api.derive.staking.overview((value: any) => this.stakingOverview = value));
    this.validators = this.stakingOverview.validators.map((a: any) => a.toString());
  }

  public async mounted() {
    this.fetchSessionInfo();
  }

  // Unsubscribe before destroying component
  public beforeDestroy() {
    this.subs.forEach((sub) => sub());
  }
}
</script>
