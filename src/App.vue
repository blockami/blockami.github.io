<template>
  <div class="h-full">
    <!--Nav-->
    <div class="w-full container mx-auto">
      <div class="w-full flex items-center justify-between">
        <a
          class="flex items-center text-indigo-400 no-underline hover:no-underline font-bold text-2xl lg:text-4xl"
          href="#"
        >
          <a
            href="/"
            class="bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-pink-500"
            >Blockami</a
          >
        </a>
      </div>
    </div>

    <!--Main-->
    <div
      class="container pt-12 md:pt-12 mx-auto flex flex-wrap flex-col md:flex-row items-center"
    >
      <!--Left Col-->
      <div
        class="flex flex-col w-full justify-center lg:items-start overflow-y-hidden"
      >
        <h1
          class="my-4 text-3xl md:text-5xl text-white opacity-75 font-bold leading-tight text-center md:text-left"
        >
          Decode EVM transactions
        </h1>

        <form
          v-on:submit.prevent
          class="bg-gray-900 opacity-75 w-full shadow-lg rounded-lg px-8 pt-6 pb-8 mb-4"
        >
          <div class="mb-4">
            <input
              class="shadow appearance-none border rounded w-full p-3 text-gray-700 leading-tight focus:ring"
              id="tx_hash"
              type="text"
              v-model="tx_hash"
              placeholder="Transaction Hash"
            />
          </div>
          <a
            @click="transactionInfo"
            class="cursor-pointer bg-gradient-to-r from-blue-400 to-pink-500 text-white font-bold py-2 px-4 rounded"
          >
            Decode
          </a>

          <a v-if="transaction"
            target="_blank"
            :href="'https://etherscan.io/tx/'+tx_hash"
            class="cursor-pointer bg-gradient-to-r ml-4 from-blue-400 to-pink-500 text-white font-bold py-2 px-4 rounded"
          >
            Vierw on Etherscan
          </a>
        </form>
      </div>

      <!--Right Col-->
      <div class="w-full p-12 overflow-hidden">
        <TransactionView :transaction="transaction"></TransactionView>
        <div
          v-if="loading"
          class="mb-4 p-4 w-full text-center rounded-lg border shadow-md sm:p-8 dark:bg-gray-900 dark:border-gray-900 opacity-75"
        >
          <svg
            class="inline mr-2 w-10 h-10 text-gray-200 animate-spin dark:text-gray-600 fill-pink-500"
            viewBox="0 0 100 101"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
              fill="currentColor"
            />
            <path
              d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
              fill="currentFill"
            />
          </svg>
          <span class="text-pink-500">Reticulating splines</span>
        </div>
        <div
          v-else
          class="mb-4 p-4 w-full rounded-lg border shadow-md sm:p-8 dark:bg-gray-900 dark:border-gray-900 opacity-75"
        >
          <h5
            class="my-4 text-3xl md:text-1xl text-white opacity-75 font-bold leading-tight text-center md:text-left"
          >
            Example transactions
          </h5>
          <ul>
            <li
              class="cursor-pointer"
              v-for="tx in example_transactions"
              :key="tx.tx_hash"
              @click="setTx(tx.tx_hash)"
            >
              {{ tx.description }}
            </li>
          </ul>
        </div>
      </div>

      <!--Footer-->
      <div class="w-full pt-16 pb-6 text-sm text-center md:text-left fade-in">
        <a
          class="text-gray-500 no-underline hover:no-underline"
          href="https://github.com/blockami"
          >Github</a
        >
      </div>
    </div>
  </div>
</template>

<script>
import { Client } from "micro-js-client";
import { defineComponent, ref } from "vue";
import TransactionView from "./TransactionView.vue";

export default defineComponent({
  name: "App",
  components: {
    TransactionView,
  },
  setup(props) {
    const tx_hash = ref("");
    const transaction = ref(undefined);
    const loading = ref(false);

    const api =
      window.localStorage.getItem("API_URL") ??
      "https://blockamiapi.drondin.xyz/";

    const micro_client = new Client({
      address: api,
      prefix: "",
    });

    const example_transactions = ref([
      {
        tx_hash:
          "0x681d6598084d120ff304003b86ce276752e36a6ad330070783f06e3ea1d15645",
        description: "Swap tokens on Uniswap",
      },
      {
        tx_hash:
          "0x67b98b2f0f875f28a9982f7781fe852f59804844045bb94fc3e82b8bf8aeec10",
        description: "Swap tokens on Morpheus",
      },
      {
        tx_hash:
          "0x2086dd5e1c33d4ee5e1b1666af413340e802f2e9d87fdbff19b0890740aecd3c",
        description: "Remove LP from Uniswap V2",
      },
      {
        tx_hash:
          "0x0334fbda89594dfa017a127292c00b56786dd880dc0899ecca3a1715795bedd5",
        description: "Wrap Ether",
      },
      {
        tx_hash:
          "0x468c13d4d614d751b95ee12f5cd6c809a27d516da6d4e4d0ab42b2566f47cc0a",
        description: "Unstake from contract Multiple NFT ERC721 //check bought",
      },
      {
        tx_hash:
          "0xbb009b5c578e9ed3fe8ad78b0b991d8c2557abcf6043f066118f783d68c5d0b3",
        description: "Transfer NFT ERC1155 //check sold",
      },
      {
        tx_hash:
          "0xb218bf7eeb9797f3d5699bff570745c7bcb736d04238bf634073f8dcc3efb6c2",
        description: "Mint multiple NFT ERC1155",
      },
      {
        tx_hash:
          "0x9a5eed45ab3b118cd213cb483d401e2e7e4dff80da394c94220b955704fc5847",
        description: "Mint NFT ERC721",
      },
      {
        tx_hash:
          "0x4ee57bebc99d073cc64f187e724ba9dc1450faa1334c318b5188b8f22b2fe0df",
        description: "Bought NFT on OpenSea Seaport 1.1",
      },
      {
        tx_hash:
          "0x8cfcd6d6a4d49c40a8054f3086f9a29b38b2edba4d08678be3e5e4b30cf8b39b",
        description: "Sold NFT on OpenSea Seaport 1.1",
      },
    ]);

    const transactionInfo = () => {
      transaction.value = {};
      loading.value = true;

      micro_client
        .call("information", "TransactionInformation", {
          network: "ETH",
          tx_hash: tx_hash.value,
          address: "",
          currency: "",
        })
        .then(async (response) => {
          transaction.value = response;
          loading.value = false;
        });
    };

    const setTx = (new_tx_hash) => {
      tx_hash.value = new_tx_hash;
      transactionInfo();
    };

    return {
      transaction,
      tx_hash,
      micro_client,
      loading,
      example_transactions,
      transactionInfo,
      setTx,
    };
  },
});
</script>
