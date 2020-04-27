<template>
    <div>
        <span class="number">{{ number }}</span>
        <span class="words">
            <template v-for="(word, index) in numberToWords(number)">
                <span class="word" v-bind:class="{passed: index <= currentIndex, active: index === currentIndex}">{{ word }}</span>
            </template>
        </span>
    </div>
</template>

<script>
    import Howler from 'howler'
    import _replace from "lodash/replace";
    import numberToWords from "number-to-words";

    export default {
        name: 'Number',
        data() {
            return {
                currentIndex: 0,
                currentWord: undefined
            }
        },
        watch: {
            number() {
                this.sayNumbers();
            }
        },
        methods: {
            numberToWords(number) {
                let words = numberToWords.toWords(number).split(' ');
                let wordsSplit = [];

                words.forEach((word) => {
                    word = _replace(word, new RegExp(",", "g"), '');
                    if (word.indexOf('-') > -1) {
                        word.split('-').forEach((wordSplit) => {
                            wordsSplit.push(wordSplit);
                        });
                    } else {
                        wordsSplit.push(word);
                    }
                });

                return wordsSplit;
            },
            sayNumbers() {
                let promise = new Promise(resolve => {
                    resolve();
                });

                this.numberToWords(this.number)
                    .forEach((word, index) => {
                        promise = promise.then(() => {
                            this.currentIndex = index;
                            this.currentWord = word;
                            return this.sayNumber(word);
                        })
                    });

                promise = promise.then(() => {
                    this.$emit('done')
                });

                return promise;
            },
            sayNumber(numberWord) {
                return new Promise((resolve) => {
                    let howl = new Howl({
                        src: [`./audio/${numberWord}.mp3`],
                        onend: function () {
                            resolve();
                        }
                    });

                    howl.play();
                })
            }
        },
        mounted() {
            this.sayNumbers();
        },
        props: {
            number: Number
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    .number {
        display: block;
        font-size: 15em;
        color: white;
        text-shadow: 0px 0px 0px rgba(0, 0, 0, 0.5);
    }

    .words {
        .word {
            display: inline-block;
            padding: 5px;
            color: white;
            font-weight: 100;

            &.active {
                font-style: italic;
            }

            &.passed {
                font-weight: bold;
            }
        }
    }
</style>
