<template>
    <h1>{{current}} / {{total}}</h1><br>
    <h1>{{percent}}%</h1>
</template>

<script>
    import axios from 'axios';
    import {ref} from 'vue';

    export default {
        name: 'App',
        setup() {
            const total = ref();
            const current = ref();
            const percent = ref();

            axios.get('./data.csv').then(response => {
                const data = response.data.split('\n').reverse();
                processMigrationData(data);
            })

            const processMigrationData = (data) => {
                total.value = parseInt(data[0].split(',')[1]);
                current.value = parseInt(data[0].split(',')[0]);
                percent.value = Math.ceil(current.value/total.value*100);
            }


            return {
                total,
                current,
                percent
            }
        }
    }
</script>

<style scoped>
    h1 {
        font-size: 24px;
    }
</style>
