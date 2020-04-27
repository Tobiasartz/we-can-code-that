<template>
    <div id="app" class="app">
        <div class="top-right">
            <span class="title">Counting to a <strong>Billion</strong></span>
            <div class="time-passed" v-if="started">
                <strong>Time Passed:</strong>
                {{ timePassedSinceStart }}
                <br/>
                <strong>Time Left:</strong>
                {{ timeLeft }}
            </div>
        </div>
        <div class="container">
            <template v-if="started">
                <number-sayer v-bind:number="currentNumber" v-on:done="onDone"></number-sayer>
            </template>
            <template v-else="">
                <button v-on:click="start" class="start-button">
                    <strong>{{ hasPrevious ? 'Continue' : 'Start' }}</strong>
                    Counting
                </button>
            </template>
        </div>
    </div>
</template>

<script>
    import NumberSayer from './components/Number'
    import 'moment-duration-format'
    import moment from 'moment'
    import store from 'store'

    const STORE_START_TIME = 'START_TIME';
    const STORE_CURRENT_NUMBER = 'CURRENT_NUMBER';

    export default {
        name: 'Number',
        data() {
            return {
                currentNumber: 0,
                started: false,
                startTime: moment(),
                now: moment(),
                tickDelta: 0,
                animationFrame: undefined
            }
        },
        methods: {
            start() {
                let storedStartTime = store.get(STORE_START_TIME);
                this.startTime = storedStartTime ? storedStartTime : moment();

                let storedCurrentNumber = store.get(STORE_CURRENT_NUMBER);
                this.currentNumber = storedCurrentNumber ? storedCurrentNumber : 1;

                if (this.queryStartNumber) {
                    this.currentNumber = parseInt(this.queryStartNumber);
                }


                this.started = true;
                this.onTick();
            },
            onDone() {
                this.currentNumber++;
            },
            onTick() {
                this.tickDelta++;

                if (this.tickDelta > 50) {
                    this.tickDelta = 0;
                    this.now = moment();
                }

                window.requestAnimationFrame(() => {
                    this.onTick();
                })
            }
        },
        computed: {
            queryParams() {
                let mapped = {};
                (location.search ? location.search.replace('?', '').split('&') : [])
                    .forEach((part) => {
                        console.log(part);
                        let parts = part.split('=');
                        if (parts.length) {
                            mapped[parts[0]] = parts[1];
                        }

                    });

                return mapped;
            },
            queryStartNumber() {
                return this.queryParams['startNumber'];
            },
            totalTimeNeeded() {
                return 2151209528005.2996;
            },
            timeLeft() {
                return moment.duration(moment(this.totalTimeNeeded).diff(this.now)).format('Y[yrs] M[m] D[d] Hh[hrs] mm[m] ss[s]');
            },
            hasPrevious() {
                return store.get(STORE_START_TIME);
            },
            timePassedSinceStart() {
                return moment.duration(this.now.diff(this.startTime)).format('Y[yrs] M[m] D[d] Hh[hrs] mm[m] ss[s]');
            },
        },
        mounted() {
            window.onbeforeunload = () => {
                store.set(STORE_START_TIME, this.startTime);
                store.set(STORE_CURRENT_NUMBER, this.currentNumber);
            };
        },
        beforeDestroy() {
            window.cancelAnimationFrame(this.animationFrame);
        },
        props: {},
        components: {
            NumberSayer
        }
    }
</script>


<style lang="scss">
    html,
    body {
        height: 100%;
        padding: 0;
        margin: 0;
        font-family: 'Roboto', sans-serif;
    }

    .button-go {
        font-size: 2em;
    }

    .top-right {
        position: absolute;
        top: 1em;
        left: 1em;
        text-align: left;
    }

    .app {
        height: 100%;
        display: flex;
        justify-content: center;
        background: #6ec6ff;
        align-items: center;
        text-align: center;
    }

    .container {
        position: absolute;
    }

    .title {
        color: white;
        font-size: 2em;
        font-weight: 100;
        display: block;
    }

    .time-passed {
        color: white;
        font-size: 1em;
        font-weight: 100;
        border-top: 1px dotted white;
        padding-top: 1em;
        margin-top: .5em;
    }

    .start-button {
        border: none;
        padding: 1em 2em;
        border-radius: 10px;
        font-size: 1.5em;
        font-family: 'Roboto', sans-serif;
        text-transform: uppercase;
        font-weight: 100;
        background: #03a9f4;
        color: white;
        cursor: pointer;
        outline: none;

        &:hover {
            background: #007ac1;
        }
    }

</style>
