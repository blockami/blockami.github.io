<template>
  <div v-if="transaction && transaction.address">
    <EventView
        v-if="transaction.events"
        v-for="(tx_event, index) in transaction.events.sort(function(a,b) { return b.priority - a.priority })"
        :key="index"
        :event="tx_event"
        :transaction="transaction"
      >
      </EventView>
    
    <h5 class="mb-2 text-3xl font-bold text-gray-900 dark:text-white">Full Information</h5>
    <JsonViewer class="mb-4" :value="transaction" copyable sort theme="dark"/>
  </div>
</template>

<script>
import { defineComponent, ref } from "vue";
import {JsonViewer} from "vue3-json-viewer"
import "vue3-json-viewer/dist/index.css";
import EventView from './EventView.vue';

export default defineComponent({
  name: "TransactionView",
  props: ["transaction"],
  components: { JsonViewer, EventView },
});
</script>
