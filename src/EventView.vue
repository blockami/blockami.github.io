<template>
  <div
    class="mb-4 p-4 w-full rounded-lg border shadow-md sm:p-8 dark:bg-gray-900 dark:border-gray-900 opacity-75"
  >
    <h5 class="mb-2 text-3xl font-bold text-gray-900 dark:text-white">
      {{ event.name }}
    </h5>
    <div class="mb-4"  v-if="token1">
      <div v-if="event.name == 'swap'">
        {{transaction.address.substring(0, 8)}}... swapped {{ token_decimals(token1.decimals, event.additional_info.input.amount) }} {{ token1.ticker }} for {{ token_decimals(token2.decimals, event.additional_info.output.amount) }} {{ token2.ticker }}
      </div>
      <div v-if="event.name == 'transfer_token'">
        {{event.additional_info.from.substring(0, 8)}}... sent {{ token_decimals(token1.decimals, event.value.amount) }} {{ token1.ticker }} to {{event.additional_info.to.substring(0, 8)}}...
      </div>
      <div v-if="event.name == 'transfer_nft' && event.additional_info.type=='ERC721'">
        {{event.additional_info.from.substring(0, 8)}}... sent {{ token1.name }} {{ event.additional_info.nft_id }} to {{event.additional_info.to.substring(0, 8)}}...
      </div>
      <div v-if="event.name == 'transfer_nft' && event.additional_info.type=='ERC1155'">
        {{transaction.address.substring(0, 8)}}... sent {{ event.additional_info.nft_amount }} NFT {{ event.additional_info.nft_id }} from {{ token1.name }} to {{event.additional_info.to.substring(0, 8)}}...
      </div>
      <div v-if="event.name == 'tx_value'">
        {{transaction.address.substring(0, 8)}}... paid {{ token_decimals(token1.decimals, event.value.amount) }} {{ token1.ticker }} in the transaction
      </div>
      <div v-if="event.name == 'unwrap_token'">
        {{transaction.address.substring(0, 8)}}... unwrapped {{ token_decimals(token1.decimals, event.additional_info.input.amount) }} {{ token1.ticker }}
      </div>
      <div v-if="event.name == 'wrap_token'">
        {{transaction.address.substring(0, 8)}}... wrapped {{ token_decimals(token1.decimals, event.additional_info.input.amount) }} {{ token1.ticker }}
      </div>
      <div v-if="event.name == 'lp_remove'">
        {{transaction.address.substring(0, 8)}}... removed LP {{ token_decimals(token1.decimals, event.additional_info.token1.amount) }} {{ token1.ticker }} - {{ token_decimals(token2.decimals, event.additional_info.token2.amount) }} {{ token2.ticker }}
      </div>
      <div v-if="event.name == 'lp_add'">
        {{transaction.address.substring(0, 8)}}... added LP {{ token_decimals(token1.decimals, event.additional_info.token1.amount) }} {{ token1.ticker }} - {{ token_decimals(token2.decimals, event.additional_info.token2.amount) }} {{ token2.ticker }}
      </div>
    </div>
    <JsonViewer
      :value="event"
      expanded
      :expandDepth="expandDepth"
      theme="dark"
    />
  </div>
</template>

<script>
import { defineComponent, ref } from "vue";
import { JsonViewer } from "vue3-json-viewer";
import "vue3-json-viewer/dist/index.css";
import { Client } from "micro-js-client";

export default defineComponent({
  name: "EventView",
  props: ["event", "transaction"],
  components: { JsonViewer },
  async mounted() {
    let token1_contract = "";
    let token2_contract = "";

    if (this.event.name == "transfer_token" || this.event.name == "tx_value") {
      token1_contract = this.event.value.token;
    }
    else  if (this.event.name == "unwrap_token" || this.event.name == "wrap_token") {
      token1_contract = this.event.additional_info.input.token;
    }
    else if (this.event.name == "transfer_nft") {
      token1_contract = this.event.additional_info.collection;
    }
    else if (this.event.name == "swap") {
      token1_contract = this.event.additional_info.input.token;
      token2_contract = this.event.additional_info.output.token;
    }
    else if (this.event.name == "lp_remove" || this.event.name == "lp_add") {
      token1_contract = this.event.additional_info.token1.token;
      token2_contract = this.event.additional_info.token2.token;
    }


    if (token1_contract != "") {
      this.micro_client
        .call("token", "GetTokenInfo", {
          network: "ETH",
          address: token1_contract,
        })
        .then((result) => {
          this.token1 = result;
        });
    }
    if (token2_contract != "") {
      this.micro_client
        .call("token", "GetTokenInfo", {
          network: "ETH",
          address: token2_contract,
        })
        .then((result) => {
          this.token2 = result;
        });
    }
  },
  setup(props) {
    const token1 = ref(undefined);
    const token2 = ref(undefined);

    const expandDepth = 3;

    const api =
      window.localStorage.getItem("API_URL") ??
      "https://blockamiapi.drondin.xyz/";

    const micro_client = new Client({
      address: api,
      prefix: "",
    });

    const token_decimals = (decimals, amount) =>{
      return (amount / Math.pow(10, decimals)).toFixed(3)
    }

    return {
      micro_client,
      token1,
      token2,
      expandDepth,
      token_decimals
    };
  },
});
</script>
