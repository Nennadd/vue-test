
// Function taticDataHasValue removed, because it's not used, 
// Conditional renderings in the template added
// Removed event listener from Mounted and Destroyed hooks, @click event added on the main tag
// Added :key attribute to v-for
// API request and setInterval moved to created hook to fetch the data immediately when Vue's options API is avaliable
// Interval cleared in beforeDestroy() hook
// Changed staticDataSummary() function to avoid nested for loops
// title changed to computed
// Added default value for a prop if its not passed to the component
// Added scoped to style

<template>
  <main @click="clicksCount++">
    <h1>{{ title }}</h1> // Maybe it would be ok to go with slot instead of props

    <div>Clicks: {{ clicksCount }}</div>
    <div>Seconds elapsed: {{ secondsElapsed }}</div>
    <div v-if="someOtherField">API Response: {{ someOtherField }}</div>
    <div>Static data summary: {{ staticDataSummary }}</div>
    <div>Time: {{ timeNow }}</div>

    <div v-if="someStaticData.length">
      <StaticDataRenderer
        v-for="item in someStaticData"
        :item="item"
        :key="item.id"
      ></StaticDataRenderer>
    </div>

    <!-- Or maybe in Vue 3 -->
    <Suspense>
      <StaticDataRenderer
        v-for="item in someStaticData"
        :item="item"
        :key="item.id"
      ></StaticDataRenderer>

      <template #fallback> Loading... </template>
    </Suspense>
  </main>
</template>

<style scoped lang="scss">
h1 {
  color: gray;
  font-size: 60px;
}
</style>

<script>
import getAPIResponseOne from "@/api/one";
import getAPIResponseTwo from "@/api/two";
import StaticDataRenderer from "@/components/static-data-renderer";
import someDOMDependentTask from "@/other/dom-dependent";
import someStaticData from "@/data/some-static-data.json";

import { ref, reactive, computed, onBeforeUnmount } from "vue"; // import for composition api

export default {
  components: {
    StaticDataRenderer,
  },
  data() {
    return {
      clicksCount: 0,
      secondsElapsed: 0,
      apiResponse: null,
    };
  },
  props: {
    title: {
      type: String,
      required: false,
      default: "Default Title",
    },
  },
  created() {
    this.getAPIResponse();
    setInterval(() => this.secondsElapsed++, 1000);
  },

  computed: {
    title() {
      return this.title + " - MyBlog";
    },
    timeNow() {
      let now = new Date();
      return now.getHours() + ":" + now.getMinutes();
    },
    staticDataSummary() {
      if (!someStaticData.length) return 0;
      return someStaticData
        .filter((data) => data.isEnabled)
        .reduce((carry, newVal) => (carry += newVal), 0);
    },
    someOtherField() {
      return apiResponse?.someField?.otherField;
    },
  },
  methods: {
    async getAPIResponse() {
      // Will try first and fall back to second in case of error
      try {
        this.apiResponse = await getAPIResponseOne();
      } catch (e) {
        this.apiResponse = await getAPIResponseTwo();
      } finally {
        setTimeout(someDOMDependentTask, 0);
      }
    },
  },
  beforeDestroy() {
    clearInterval(this.secondsElapsed);
  },

  // ************************************* Vue 3 Composition API  ******************************************************
  setup() {
    const props = defineProps({
      title: {
        type: String,
        required: false,
        default: "Default Title",
      },
    });

    const clicksCount = ref(0);
    const secondsElapsed = ref(0);
    const apiResponse = ref(null);

    // Or maybe as a reactive state object
    const state = reactive({
      clicksCount: 0,
      secondsElapsed: 0,
      apiResponse: null,
    });

    // NOTE setup() function runs before options api is created, so I think there is no need for onMounted()
    async function getAPIResponse() {
      try {
        apiResponse.value = await getAPIResponseOne();
      } catch (e) {
        apiResponse.value = await getAPIResponseTwo();
      } finally {
        setTimeout(someDOMDependentTask, 0);
      }
    }

    const title = computed(() => {
      return props.title + " - MyBlog";
    });

    const timeNow = computed(() => {
      var now = new Date();
      return now.getHours() + ":" + now.getMinutes();
    });

    const staticDataSummary = computed(() => {
      if (!someStaticData.length) return 0;
      return someStaticData
        .filter((data) => data.isEnabled)
        .reduce((carry, newVal) => (carry += newVal), 0);
    });

    const someOtherField = computed(() => {
      return apiResponse.value?.someField?.otherField;
    });

    onBeforeUnmount(() => {
      clearInterval(secondsElapsed.value);
    });

    return {
      title,
      secondsElapsed,
      clicksCount,
      timeNow,
      staticDataSummary,
      someOtherField,
    };
  },
};
</script>
