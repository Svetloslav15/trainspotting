<script>
export default {
    data() {
        return {
            trains: null
        };
    },
    mounted() {
        fetch('http://localhost:3000/journeys')
            .then(response => response.json())
            .then(data => {
                this.trains = data;
            });
    },
    methods: {
        formatDate(value) {
            return new Date(value).toLocaleTimeString([], { 
                hour: '2-digit', 
                minute: '2-digit', 
                hour12: false ,
                timeZone: 'UTC'
            });
        },
        calculateRoadWidth(stations) {
            const lastStation = new Date(stations[stations.length - 1].time);
            const hours = lastStation.getUTCHours();
            const minutes = lastStation.getUTCMinutes();
            const position = ((hours - 9) * 60 + minutes) * 5;

            return {
                width: position + 'px'
            }
        },
        calculateStationPosition(value) {
            const date = new Date(value);
            const hours = date.getUTCHours();
            const minutes = date.getUTCMinutes();
            const position = ((hours - 9) * 60 + minutes) * 5;

            return {
                left: position + 'px'
            }
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
                <button class="button">Start</button>
            </div>
            <div class="right-block">
                <p><span class="bold">Trains in Transit: </span>: asdadad</p>
                <p><span class="bold">Current Time: </span>: 10:12</p>
            </div>
        </div>
        <!-- Header -->
        <!-- Table -->
        <div class="table">
            <!-- Heading -->
            <div class="heading">
                <p>Name</p>
                <p>Route</p>
                <p>Timetable</p>
                <p>Next Station</p>
                <p>Train</p>
            </div>
            <!-- Heading -->
            <!-- Row -->
            <div class="row" v-for="train in trains">
                <p class="name">{{ train.name }}</p>
                <p class="route">{{ train.route }}</p>
                <!-- Timetable -->
                <div class="timetable">
                    <img src="/assets/road-texture.jpg" 
                    v-bind:style="calculateRoadWidth(train.timetable)" class="road"/>
                    <!-- Station -->
                    <div class="stations">
                        <div class="station" v-bind:style="calculateStationPosition(station.time)"
                            v-for="station in train.timetable">
                            <img src="/assets/station.svg" class="station-image" />
                            <div class="line"></div>
                            <span class="time">{{ formatDate(station.time) }}</span>
                        </div>
                    </div>
                    <img src="/assets/train.svg" alt="" class="train">

                    <!-- Station -->
                </div>
                <!-- Timetable -->
                <p class="next-station">Royston</p>
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

.table .row p:nth-child(1),
.heading p:nth-child(1),
.heading p:nth-child(2),
.row p:nth-child(2) {
    width: 20%;
}

.table .row p:nth-child(3),
.heading p:nth-child(3),
.row .timetable {
    width: 40%;
}

.table .row p:nth-child(4),
.heading p:nth-child(4),
.row .next-station {
    width: 10%;
}

.table .row p:nth-child(5),
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

.timetable .train {
    position: absolute;
    top: 45%;
    left: 18px;
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
</style>