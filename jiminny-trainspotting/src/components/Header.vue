<script>
export default {
    data() {
        return {
            startingHour: 9,
            minutesInOneHour: 60,
            stepToMove: 5
        };
    },
    props: {
        currentTimeInMinutes: Number,
        isMoving: Boolean,
        toggleMoving: Function,
        formatTrainsInTransit: Function
    },
    methods: {
        formatCurrentTime() {
            const hours = (Math.floor(this.currentTimeInMinutes / this.minutesInOneHour) + this.startingHour).toString().padStart(2, '0');
            const minutes = (this.currentTimeInMinutes % this.minutesInOneHour).toString().padStart(2, '0');
            return `${hours}:${minutes}`;
        },
    }
}
</script>

<template>
    <div class="header">
        <div class="left-block">
            <h1 class="title">Jiminny Trainspotting</h1>
            <button type="button" v-if="!isMoving" @click="toggleMoving">Start</button>
            <button type="button" v-else @click="toggleMoving">Stop</button>
        </div>
        <div class="right-block">
            <p><span class="bold">Trains in Transit: </span>: {{ formatTrainsInTransit() }}</p>
            <p><span class="bold">Current Time: </span> {{ formatCurrentTime() }}</p>
        </div>
    </div>
</template>

<style scoped>
.bold {
    font-weight: bold;
}

.header {
    display: flex;
    width: 100%;
    justify-content: space-between;
}


.left-block {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    ;
}

.left-block .title {
    font-size: 30px;
}

.left-block button {
    display: inline;
    height: fit-content;
    margin-left: 1rem;
}


.right-block {
    display: flex;
    align-self: center;
}

.right-block p {
    margin: 0 5px;
}

@media (max-width: 1024px) {

    button,
    .name-route,
    .next-station,
    .timetable,
    .right-block {
        display: none !important;
    }
}
</style>