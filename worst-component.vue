<template>
    <main>
        <h1>
            {{ getTitle() }}
        </h1>

        <div>Clicks: {{ clicksCount }}</div>
        <div>Seconds elapsed: {{ secondsElapsed }}</div>
        <div>API Response: {{ apiResponse.someField.otherField }}</div>
        <div>Static data summary: {{ staticDataSummary }}</div>
        <div>Time: {{ timeNow }}</div>

        <div>
            <StaticDataRenderer
                v-for="item in someStaticData"
                :item="item"
            ></StaticDataRenderer>
        </div>
    </main>
</template>

<style lang="scss">
h1 {
    color: gray;
    font-size: 60px;
}
</style>

<script>
import getAPIResponseOne from '@/api/one';
import getAPIResponseTwo from '@/api/two';
import StaticDataRenderer from '@/components/static-data-renderer';
import someDOMDependentTask from '@/other/dom-dependent';
import someStaticData from '@/data/some-static-data.json';

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
        },
    },
    computed: {
        timeNow() {
            var now = new Date();
            return now.getHours() + ':' + now.getMinutes();
        },
        staticDataSummary() {
            let result = 0;

            for (let i = 0; i < someStaticData.length; i++) {
                if (someStaticData[i].isEnabled) {
                    for (let j = 0; j < someStaticData[i].items.length; j++) {
                        result += someStaticData[i].items[j].value;
                    }
                }
            }

            return result;
        },
        staticDataHasValues() {
            return someStaticData.length > 0 ? true : false;
        },
    },
    mounted() {
        window.addEventListener('click', () => this.clicksCount++);

        setInterval(() => this.secondsElapsed++, 1000);

        this.getAPIResponse();
    },
    destroyed() {
        window.removeEventListener('click', () => this.clicksCount++);
    },
    methods: {
        getTitle() {
            return this.title + ' - MyBlog';
        },
        async getAPIResponse() {
            // Will try first and fall back to second in case of error
            try {
                getAPIResponseOne().then(resp => this.apiResponse = resp);
            } catch (e) {
                const resp = await getAPIResponseTwo();
                this.apiResponse = resp;
            } finally {
                setTimeout(someDOMDependentTask, 0);
            }
        },
    },
};
</script>
