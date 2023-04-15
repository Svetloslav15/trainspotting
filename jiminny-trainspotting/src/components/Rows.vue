<script>
export default {
    data() {
        return {
            trains: [],
            intervalId: null,
            startingHour: 9,
            minutesInOneHour: 60,
            trainsPositions: [0, 0, 0, 0],
            stepToMove: 5,
            minutesToEndOfWorkingDay: 120,
            moving: false
        };
    },
    props: {
        isMoving: Boolean,
        trainsInTransit: Array,
        calculatePosition: Function,
        currentTimeInMinutes: Number
    },
    watch: {
        isMoving(newVal) {
            if (this.isMoving) {
                this.startMoving();
            }
            else {
                this.stopMoving();
            }
        }
    },
    mounted() {
        fetch('http://localhost:3000/journeys')
            .then(response => response.json())
            .then(data => {
                for (let train of data) {
                    train.nextStation = {
                        name: train.timetable[1].station,
                        index: 1,
                        position: this.calculatePosition(train.timetable[1])
                    }

                    for (let station of train.timetable) {
                        station.position = this.calculatePosition(station);
                    }

                    train.lastStationPosition = this.calculatePosition(train.timetable[train.timetable.length - 1]);
                }
                this.trains = data;
            });
    },
    beforeUnmount() {
        clearInterval(this.intervalId)
    },
    methods: {
        calculateRoadWidth(stations) {
            const position = this.calculatePosition(stations[stations.length - 1])
            return {
                width: position + 'px'
            }
        },
        calculateStationPosition(value) {
            const date = new Date(value);
            const hours = date.getUTCHours();
            const minutes = date.getUTCMinutes();
            const position = ((hours - this.startingHour) * this.minutesInOneHour + minutes) * this.stepToMove;

            return {
                left: position + 'px'
            }
        },
        formatDate(value) {
            return new Date(value).toLocaleTimeString([], {
                hour: '2-digit',
                minute: '2-digit',
                hour12: false,
                timeZone: 'UTC'
            });
        },
        startMoving() {
            this.intervalId = setInterval(() => {
                for (let element of this.$refs.train) {
                    const trainIndex = +element.getAttribute('data-index');
                    this.trainsPositions[trainIndex] += this.stepToMove;
                    const currentTrain = this.trains[trainIndex];

                    // Check if the train has arrived on a station
                    if (currentTrain.nextStation.position === this.trainsPositions[trainIndex] &&
                        currentTrain.timetable[currentTrain.nextStation.index + 1]) {
                        // Change next station value
                        currentTrain.nextStation = {
                            name: currentTrain.timetable[currentTrain.nextStation.index + 1].station,
                            index: currentTrain.nextStation.index + 1,
                            position: currentTrain.timetable[currentTrain.nextStation.index + 1].position
                        }
                    }

                    // Move the train on the road
                    if (this.trains[trainIndex].lastStationPosition >= this.trainsPositions[trainIndex]) {
                        element.style.transform = `translateX(${this.trainsPositions[trainIndex]}px)`;
                    }
                    else {
                        // Move train in transit
                        if (!this.trainsInTransit.includes(currentTrain.train.name)) {
                            this.trainsInTransit.push(currentTrain.train.name);
                            this.$emit('addTrainToTransit', currentTrain.train.name);
                        }
                    }

                }
                if (this.currentTimeInMinutes < this.minutesToEndOfWorkingDay) {
                    this.$emit('updateCurrentTimeInMinutes');
                }
                else {
                    this.stopMoving();
                }
            }, 500);
        },
        stopMoving() {
            clearInterval(this.intervalId);
        }
    }
}
</script>
<template>
    <div class="row" v-for="(train, index) in trains" :key="index">
        <p class="name-route">{{ train.name }} / {{ train.route }}</p>
        <p class="name">{{ train.name }}</p>
        <p class="route">{{ train.route }}</p>
        <!-- Timetable -->
        <div class="timetable">
            <img src="/assets/road-texture.jpg" v-bind:style="calculateRoadWidth(train.timetable)" class="road" />
            <!-- Station -->
            <div class="stations">
                <div class="station" v-bind:style="calculateStationPosition(station.time)"
                    v-for="station in train.timetable">
                    <img src="/assets/station.svg" class="station-image" />
                    <div class="line"></div>

                    <span class="time">{{ formatDate(station.time) }}</span>
                </div>
            </div>
            <img ref="train" :data-index="index" src="/assets/train.svg" alt="" class="train-icon">
            <!-- Station -->
        </div>
        <!-- Timetable -->
        <!-- Timetable mobile -->
        <div class="timetable-mobile">
            <p v-for="station in train.timetable">{{ formatDate(station.time) }}: {{ station.station }}</p>
        </div>
        <!-- Timetable mobile -->
        <p class="next-station">{{ train.nextStation.name }}</p>
        <p class="train">{{ train.train.name }}</p>
    </div>
    <!-- Row -->
</template>

<style scoped>
.bold {
    font-weight: bold;
}

.table {
    box-shadow: 0px 5px 11px -1px rgba(0, 0, 0, 0.75);
}

.container {
    width: 90%;
    margin: 30px auto;
    padding-top: 30px;
}

.table {
    width: 100%;
}

.row {
    display: flex;
    height: 150px;
    align-items: center;
}

.row:nth-child(odd) {
    background-color: #FAFAFA;
}

.row .name,
.row .route {
    width: 15%;
}

.row .timetable,
.row .timetable {
    width: 50%;
    min-width: 630px;
}

.row .next-station {
    width: 10%;
}

.row .train {
    width: 10%;
}

.row .name {
    padding-left: 10px;
}

.stations {
    display: flex;
    position: relative;
    top: 8px;
}

.timetable .station {
    display: flex;
    flex-direction: column;
    align-items: center;
    position: absolute;
    top: -40px;
}

.timetable .train-icon {
    position: absolute;
    top: 45%;
    left: 10px;
    width: 16px;
    height: 16px;
    background-color: white;
    padding: 1px;
    border-radius: 50%;
}

.road {
    background-image: url(/assets/road-texture.jpg);
    background-repeat: repeat-y;
    width: 506px;
    height: 16px;
    position: absolute;
    top: 45%;
    left: 18px;
    border: 1px solid;
    background-size: 504px 19px;
    background-position-y: center;
}

.station .line {
    width: 1px;
    background-color: black;
    height: 24px;
    margin: 12px 0;
}

.station .time {
    transform: rotate(-90deg);
}

.station .station-image {
    width: 20px;
    height: 16px;
}

.timetable-mobile {
    display: none;
}

.row p {
    word-wrap: break-word;
}

@media (min-width: 1300px) {
    .name-route {
        display: none;
    }
}

@media (min-width: 1024px) and (max-width: 1299px) {
    .name-route {
        width: 20%;
        display: flex;
        padding-left: 10px;
        ;
    }

    .name,
    .route {
        display: none;
    }
}


@media (max-width: 1024px) {

    button,
    .name-route,
    .next-station,
    .timetable,
    .right-block {
        display: none !important;
    }

    .timetable-mobile {
        display: flex;
        flex-direction: column;
        width: 40%;
    }

    .row,
    .heading {
        height: 100%;
        justify-content: space-between;
    }

    .row {
        padding: 10px 5px;
    }

    .timetable-heading {
        min-width: unset !important;
    }
}</style>