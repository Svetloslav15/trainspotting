<template>
    <div class="heading">
        <p class="name-route">Name / Route</p>
        <p class="name">Name</p>
        <p class="route">Route</p>
        <p class="timetable-heading">Timetable</p>
        <p class="next-station">Next Station</p>
        <p class="train">Train</p>
    </div>
</template>

<style scoped>
.heading {
    display: flex;
    color: white;
    background-color: #EE3584;
    padding: 10px;
    width: 100%;
    display: flex;
    width: 100%;
    justify-content: space-between;
}

.name, .route {
    width: 15%;
}

.timetable, .timetable-heading {
    width: 50%;
    min-width: 630px;
}

.next-station, .train {
    width: 10%;
}

@media (min-width: 1300px) {
    .name-route {
        display: none;
    }
}

@media (max-width: 1024px) {
    .name-route, .next-station {
        display: none !important;
    }

    .heading {
        height: 100%;
        justify-content: space-between;
    }

    .timetable-heading {
        min-width: unset !important;
    }
}

@media (min-width: 1024px) and (max-width: 1299px) {
    .name-route {
        width: 20%;
        display: flex;
        padding-left: 10px;
    }
    .name, .route {
        display: none;
    }
}

@media (max-width: 1024px) {
    .name, .route {
        width: 22%;
    }
}
</style>