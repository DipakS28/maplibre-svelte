<script>
    import { Map } from "maplibre-gl";
    // import { updateMap } from "./Map.svelte";
    import {
        updateMap,
        isChecked,
        tracking,
        showChart,
        showList,
        showRatingsChart,
        closeRatingsChart,
    } from "./stores";
    import { time_ranges_to_array } from "svelte/internal";

    export let map;
    export let directions;

    function startTracking() {
        console.log("Start Tracking");

        if (!$tracking) {
            tracking.set(true);
            intervalId = setInterval(() => {
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition((position) => {
                        let { latitude, longitude } = position.coords;
                        if (
                            Math.abs(latitude - lastLat) <= 0.00005 &&
                            Math.abs(longitude - lastLang) <= 0.00005
                        ) {

                            latitude = lastLat;
                            longitude - lastLang;
                        }
                        if (
                            lastLang != longitude ||
                            lastLat != latitude ||
                            time == 3
                        ) {
                            // latitude += i;
                            // longitude += j;
                            // i += Math.random() * 0.02;
                            // i -= 0.01;
                            // j += Math.random() * 0.03;
                            // j -= 0.01;
                            console.log(
                                `Latitude: ${latitude}, Longitude: ${longitude}`,
                            );
                            lastLang = longitude;
                            lastLat = latitude;
                            if (first) {
                                const fMarker = new maplibregl.Marker()
                                    .setLngLat([longitude, latitude])
                                    .addTo(map);
                                first = false;
                                firstMarker.push(fMarker);
                            } else {
                                firstMarker.forEach((marker) =>
                                    marker.remove(),
                                );
                                firstMarker = [];
                            }
                            updateMap(
                                latitude,
                                longitude,
                                map,
                                directions,
                                time,
                            );
                            if (time != 3) {
                                time = 0;
                            } else {
                                time++;
                            }
                        } else {
                            time++;
                        }
                    });
                }
            }, 5000);
        }
    }

    function stopTracking() {
        console.log("Stop Tracking");
        if (tracking) {
            tracking.set(false);
            clearInterval(intervalId);
        }
    }

    function toggleShowChart() {
        showChart.update((value) => !value);
        if ($showChart) {
            showRatingsChart();
        } else {
            closeRatingsChart();
        }
    }

    function downloadMap() {
        // var img = map.getCanvas().toDataURL('image/png');
        // const downloadLink = document.createElement("a")
        // downloadLink.href = img
        // downloadLink.download = "route.png"
        // downloadLink.click();

        // Assuming your map element has an id of 'map'
        var mapElement = document.getElementById("map");

        html2canvas(mapElement).then((canvas) => {
            const img = canvas.toDataURL("image/png");
            const downloadLink = document.createElement("a");
            downloadLink.href = img;
            downloadLink.download = "route.png";
            downloadLink.click();
        });
    }

    function toggleLocationList() {
        showList.update((value) => !value);
        if ($showList) {
            document.getElementById("map").style.width = "0vw";
            document.getElementById("locations").style.width = "100vw";
            document.getElementById("toggleList").textContent =
                "Hide Location List";
        } else {
            document.getElementById("map").style.width = "100vw";
            document.getElementById("locations").style.width = "0vw";
            document.getElementById("toggleList").textContent =
                "View Location List";
        }
    }

    let intervalId;
    let first = true;
    let time = 0;
    let i = 0;
    let j = 0;
    let firstMarker = [];
    let lastLat;
    let lastLang;
</script>

<div id="controls">
    <button on:click={startTracking}>Start Tracking</button>
    <button on:click={stopTracking}>Stop Tracking</button>
    <!-- <button on:click={toggleShowChart}>Show Ratings Chart</button> -->
    <button id="toggleList" on:click={toggleLocationList}
        >Show Location List</button
    >
    <button on:click={downloadMap}>Download map Route</button>
    <label class="switch" title="Toggle to ON/OFF Add Location Manually">
        <input type="checkbox" bind:checked={$isChecked} />
        <span class="slider round">
            <span class="on">ON</span>
            <span class="off">OFF</span>
        </span>
    </label>
</div>
