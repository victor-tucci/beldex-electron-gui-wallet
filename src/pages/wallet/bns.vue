<template>
  <q-page class="bns-page">
    <div class="header row items-center justify-center q-pt-md">
      <q-btn-toggle
        v-model="screen"
        toggle-color="primary"
        color="accent"
        :options="[
          {
            label: $t('titles.bns.purchase'),
            value: 'purchase'
          },
          {
            label: $t('titles.bns.myBns'),
            value: 'my_bns'
          }
        ]"
      />
    </div>
    <BNSPurchase v-if="screen === 'purchase'" ref="purchase" />
    <MyBNS v-if="screen === 'my_bns'" @onUpdate="onUpdate" @onRenew="onRenew" />
  </q-page>
</template>

<script>
import BNSPurchase from "components/bns/bns_purchase";
import MyBNS from "components/bns/bns_mybns";
import Vue from "vue";
import _ from "lodash";

export default {
  components: {
    MyBNS,
    BNSPurchase
  },
  data() {
    return {
      screen: "purchase"
    };
  },
  methods: {
    purchasePageAction(record, action) {
      // don't update the pointer to the record (else it'll update on the records page)
      let recordCopy = _.cloneDeep(record);

      if (record.type === "belnet" && record.name.endsWith(".bdx")) {
        // The UI expects no ".bdx" extension
        recordCopy.name = recordCopy.name.slice(0, -5);
        recordCopy.value = recordCopy.value.slice(0, -5);
      }
      this.screen = "purchase";
      // refs are not dynamic, so let the purchase tab render
      // then we can call the ref method
      Vue.nextTick().then(() => {
        this.$refs.purchase[action](recordCopy);
      });
    },
    onUpdate(record) {
      let updateRecord = {
        ...record,
        // Don't pre-fill these fields on update
        value: "",
        owner: "",
        backup_owner: ""
      };
      this.purchasePageAction(updateRecord, "startUpdating");
    },
    onRenew(record) {
      this.purchasePageAction(record, "startRenewing");
    }
  }
};
</script>

<style lang="scss"></style>
