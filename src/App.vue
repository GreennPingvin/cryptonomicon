<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div
      v-if="spinnerIsShown"
      class="fixed w-100 h-100 bg-purple-800 inset-0 z-50 flex items-center justify-center"
    >
      <svg
        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"
        fill="none"
        viewBox="0 0 24 24"
        xmlns="http://www.w3.org/2000/svg"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
          fill="currentColor"
        ></path>
      </svg>
    </div>
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label class="block text-sm font-medium text-gray-700" for="wallet"
              >Ticker</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="tickerName"
                v-on:keydown.enter="add(tickerName)"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                name="wallet"
                placeholder="Например DOGE"
                type="text"
              />
            </div>
            <div class="flex bg-white shadow-md p-1 rounded-md flex-wrap">
              <span
                v-for="(promptTickerName, i) in promptTickerNames"
                :key="i"
                @click="add(promptTickerName)"
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
              >
                {{ promptTickerName }}
              </span>
            </div>
            <div v-if="warningIsShown" class="text-sm text-red-600">
              Такой тикер уже добавлен
            </div>
          </div>
        </div>
        <button
          @click="add(tickerName)"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
          type="button"
        >
          <!-- Heroicon name: solid/mail -->
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            fill="#ffffff"
            height="30"
            viewBox="0 0 24 24"
            width="30"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>
      <hr class="w-full border-t border-gray-600 my-4" />
      <template v-if="tickers.length > 0">
        <button
          v-if="page > 1"
          @click="page = page - 1"
          class="my-4 mx-1 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Назад
        </button>
        <button
          v-if="hasNextPage"
          @click="page = page + 1"
          class="my-4 mx-1 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Вперед
        </button>
        <div>Фильтр: <input type="text" v-model="filter" /></div>
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="ticker of paginatedTickers"
            :key="ticker.name"
            @click="select(ticker)"
            :class="{
              'border-4': selectedTicker === ticker,
            }"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ ticker.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ ticker.price }}
              </dd>
            </div>
            <div class="w-full border-ticker border-gray-200"></div>
            <button
              @click.stop="remove(ticker.name)"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
            >
              <svg
                aria-hidden="true"
                class="h-5 w-5"
                fill="#718096"
                viewBox="0 0 20 20"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  clip-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  fill-rule="evenodd"
                ></path>
              </svg>
              Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <section v-if="selectedTicker" class="relative">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          {{ selectedTicker.name }} - USD
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div
            v-for="(bar, idx) in normalizedGraph"
            :key="idx"
            :style="{ height: `${bar}%` }"
            class="bg-purple-800 border w-10"
          ></div>
        </div>
        <button
          @click="selectedTicker = null"
          class="absolute top-0 right-0"
          type="button"
        >
          <svg
            height="30"
            viewBox="0 0 511.76 511.76"
            width="30"
            x="0"
            xml:space="preserve"
            xmlns="http://www.w3.org/2000/svg"
            y="0"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                data-original="#000000"
                fill="#718096"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      tickerName: "",
      filter: "",

      tickers: [],
      selectedTicker: null,
      promptTickerNames: ["BTC", "BTCP", "DOGE", "BCH"],
      warningIsShown: false,
      intervalIds: new Map(),

      graph: [],
      spinnerIsShown: true,

      page: 1,
    };
  },
  created() {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );
    if (windowData.filter) {
      this.filter = windowData.filter;
    }
    if (windowData.page) {
      this.page = windowData.page;
    }

    const savedTickers = JSON.parse(localStorage.getItem("TICKERS"));
    if (savedTickers) {
      this.tickers = savedTickers;
      savedTickers.forEach((ticker) => this.subscribeToUpdates(ticker.name));
    }
  },
  async mounted() {
    const f = await fetch(
      "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
    );
    const { Data: coinList } = await f.json();
    this.spinnerIsShown = false;
    console.log(coinList);
  },
  computed: {
    startIndex() {
      return (this.page - 1) * 3;
    },
    endIndex() {
      return this.startIndex + 3;
    },
    filteredTickers() {
      return this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter.toUpperCase())
      );
    },
    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },
    hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
    },
    normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);
      return this.graph.map(
        (price) => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },
    pageStateOptions() {
      return {
        filter: this.filter,
        page: this.page,
      };
    },
  },
  methods: {
    subscribeToUpdates(tickerName) {
      const intervalId = setInterval(async () => {
        const f = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=7784ac0ed1b1ebe26a1dea01a49f5b869768d5a062fc6f6f1987099972bb64da`
        );
        const data = await f.json();
        const ticker = this.tickers.find(
          (ticker) => ticker.name === tickerName
        );
        if (ticker && data.USD) {
          ticker.price =
            data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);
          if (this.selectedTicker?.name === tickerName) {
            this.graph.push(data.USD);
          }
        }
      }, 1000);
      this.intervalIds.set(tickerName, intervalId);
    },
    add(tickerName) {
      if (!tickerName) {
        return;
      }
      const currentTicker = {
        name: tickerName.toUpperCase(),
        price: "-",
      };

      if (this.isTickerNameInList(currentTicker.name)) {
        this.warningIsShown = true;
        this.tickerName = tickerName;
        return;
      }
      this.warningIsShown = false;

      this.tickers = [...this.tickers, currentTicker];

      this.subscribeToUpdates(currentTicker.name);
      this.filter = "";
    },
    remove(tickerName) {
      this.tickers = this.tickers.filter(
        (ticker) => ticker.name !== tickerName
      );
      clearInterval(this.intervalIds.get(tickerName));

      if (tickerName === this.selectedTicker?.name) {
        this.selectedTicker = null;
      }
    },
    select(ticker) {
      this.selectedTicker = ticker;
      this.graph = [];
    },
    isTickerNameInList(tickerName) {
      return Boolean(this.tickers.find((ticker) => ticker.name === tickerName));
    },
  },
  watch: {
    filter() {
      this.page = 1;
    },
    pageStateOptions(value) {
      window.history.pushState(
        null,
        document.title,
        `${location.pathname}?filter=${value.filter}&page=${value.page}`
      );
    },
    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.page > 1) {
        this.page -= 1;
      }
    },
    selectedTicker() {
      this.graph = [];
    },
    tickers() {
      localStorage.setItem("TICKERS", JSON.stringify(this.tickers));
    },
  },
};
</script>
