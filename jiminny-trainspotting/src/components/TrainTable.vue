<script>
import Header from './Header.vue';
import Heading from './Heading.vue';
import Rows from './Rows.vue';

export default {
    data() {
        return {
            isMoving: false,
            intervalId: null,
            currentTimeInMinutes: 0,
            trainsInTransit: [],
            startingHour: 9,
            minutesInOneHour: 60,
            stepToMove: 5,
            myRef: []
        };
    },
    methods: {
        calculatePosition(value) {
            const stationTime = new Date(value.time);
            const hours = stationTime.getUTCHours();
            const minutes = stationTime.getUTCMinutes();
            return ((hours - this.startingHour) * this.minutesInOneHour + minutes) * this.stepToMove;
        },
        formatTrainsInTransit() {
            return this.trainsInTransit.length > 0 ? this.trainsInTransit.join(', ') : 'None'
        },
        toggleMoving() {
            this.isMoving = !this.isMoving;
        },
        updateCurrentTimeInMinutes() {
            this.currentTimeInMinutes++;
        },
        addTrainToTransit(value) {
            if (!this.trainsInTransit.includes(value)) {
                this.trainsInTransit.push(value);
            }
        }
    },
    components: {
        Header,
        Heading,
        Rows
    }
}
</script>

<template>
    <div class="container">
        <Header 
            :is-moving="isMoving" 
            :toggle-moving="toggleMoving" 
            :current-time-in-minutes="this.currentTimeInMinutes"
            :format-trains-in-transit="formatTrainsInTransit" />
        <div class="table">
            <Heading />
            <Rows 
                :current-time-in-minutes="currentTimeInMinutes" 
                :is-moving="isMoving"
                :calculate-position="this.calculatePosition" 
                :trains-in-transit="this.trainsInTransit"
                @updateCurrentTimeInMinutes="updateCurrentTimeInMinutes" 
                @addTrainToTransit="addTrainToTransit" />
        </div>
    </div>
</template>

<style scoped>
.table {
    width: 100%;
    box-shadow: 0px 5px 11px -1px rgba(0, 0, 0, 0.75);
}

.container {
    width: 90%;
    margin: 0 auto;
    padding: 30px 0;
}
</style>