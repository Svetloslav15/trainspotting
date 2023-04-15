<script>
export default {
    data() {
        return {
            trains: null,
            isMoving: false,
            intervalId: null,
            trainsPositions: [0, 0, 0, 0], //starting positions,
            currentTimeInMinutes: 0,
            trainsInTransit: [],
            startingHour: 9,
            minutesInOneHour: 60,
            stepToMove: 5
        };
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
    methods: {
        formatDate(value) {
            return new Date(value).toLocaleTimeString([], {
                hour: '2-digit',
                minute: '2-digit',
                hour12: false,
                timeZone: 'UTC'
            });
        },
        calculatePosition(value) {
            const stationTime = new Date(value.time);
            const hours = stationTime.getUTCHours();
            const minutes = stationTime.getUTCMinutes();
            return ((hours - this.startingHour) * this.minutesInOneHour + minutes) * this.stepToMove;
        },
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
        startMoving() {
            this.isMoving = true;
            this.intervalId = setInterval(() => {
                    for (let element of this.$refs.train) {
                        const trainIndex = +element.getAttribute('data-index');
                        this.trainsPositions[trainIndex] += this.stepToMove;

                        const currentTrain = this.trains[trainIndex];

                        if (currentTrain.nextStation.position === this.trainsPositions[trainIndex] &&
                        currentTrain.timetable[currentTrain.nextStation.index + 1]) {
                            currentTrain.nextStation = {
                                name: currentTrain.timetable[currentTrain.nextStation.index + 1].station,
                                index: currentTrain.nextStation.index + 1,
                                position: currentTrain.timetable[currentTrain.nextStation.index + 1].position
                            }
                        }
                        if (this.trains[trainIndex].lastStationPosition >= this.trainsPositions[trainIndex]) {
                            element.style.transform = `translateX(${this.trainsPositions[trainIndex]}px)`;
                        }
                        else {
                            // Move train in transit
                            if (!this.trainsInTransit.includes(currentTrain.train.name)) {
                                this.trainsInTransit.push(currentTrain.train.name);
                            }
                        }

                    }
                if (this.currentTimeInMinutes < 120) {
                    this.currentTimeInMinutes++;
                }
                else {
                    this.stopMoving();
                }
            }, 500);
        },
        stopMoving() {
            this.isMoving = false;
            clearInterval(this.intervalId);
        },
        formatCurrentTime() {
            const hours = (Math.floor(this.currentTimeInMinutes / this.minutesInOneHour) + this.startingHour).toString().padStart(2, '0');
            const minutes = (this.currentTimeInMinutes % this.minutesInOneHour).toString().padStart(2, '0');
            return `${hours}:${minutes}`;
        },
        formatTrainsInTransit() {
            return this.trainsInTransit.length > 0 ? this.trainsInTransit.join(', ') : 'None'
        }
    }
}
</script>

<template>
    <div class="container">
        <!-- Header -->
        <div class="header">
            <div class="left-block">
                <h1 class="title">Jiminny Trainspotting</h1>
                <button class="button" v-if="!isMoving" @click="startMoving">Start</button>
                <button class="button" v-else @click="stopMoving">Stop</button>
            </div>
            <div class="right-block">
                <p><span class="bold">Trains in Transit: </span>: {{ formatTrainsInTransit() }}</p>
                <p><span class="bold">Current Time: </span> {{ formatCurrentTime() }}</p>
            </div>
        </div>
        <!-- Header -->
        <!-- Table -->
        <div class="table">
            <!-- Heading -->
            <div class="heading">
                <p class="name-route">Name / Route</p>
                <p class="name">Name</p>
                <p class="route">Route</p>
                <p class="timetable">Timetable</p>
                <p class="next-station">Next Station</p>
                <p class="train">Train</p>
            </div>
            <!-- Heading -->
            <!-- Row -->
            <div class="row" v-for="(train, index) in trains" :key="index">
                <p class="name-route">{{ train.name }} / {{train.route}}</p>
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
                <p class="next-station">{{ train.nextStation.name }}</p>
                <p class="train">{{ train.train.name }}</p>
            </div>
            <!-- Row -->
        </div>
        <!-- Table -->
    </div>
</template>

<style scoped>
.bold {
    font-weight: bold;
}

.container {
    width: 90%;
    margin: 30px auto;
    padding-top: 30px;
}

.header {
    display: flex;
    width: 100%;
    justify-content: space-between;
}

.left-block {
    display: flex;
    align-items: center;
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

.table {
    width: 100%;
}

.table .heading {
    display: flex;
    color: white;
    background-color: #EE3584;
    padding: 10px;
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

.table .row .name,
.heading .name,
.heading .route,
.row .route {
    width: 15%;
}

.table .row .timetable,
.heading .timetable,
.row .timetable {
    width: 50%;
    min-width: 630px;
}

.table .heading .next-station,
.row .next-station {
    width: 10%;
}

.table .row .train,
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

@media (min-width: 1300px) {
    .name-route {
        display: none;
    }
}

@media (min-width: 1024px) and (max-width: 1299px) {
    .name-route {
        width: 20%;
        display: flex;
    }

    .name, .route {
        display: none;
    }
}
</style>