<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import MapLibreGlDirections, {
        LoadingIndicatorControl,
    } from "@maplibre/maplibre-gl-directions";
    import {
        isChecked,
        firstMarker,
        initLoadLocations,
        getInitialData,
        updateMap,
        searchLocations,
        updateLocationList
    } from "./stores";
    import Controls from "./controls.svelte";
    import { bind } from "svelte/internal";

    let map;
    let directions;

    let isCollapsed = false;

    function toggleCollapse() {
        isCollapsed = !isCollapsed;
    }

    onMount(() => {
        map = new maplibregl.Map({
            container: "map",
            style: "https://raw.githubusercontent.com/go2garret/maps/main/src/assets/json/openStreetMap.json",
            center: originalCenter,
            zoom: originalZoom,
            preserveDrawingBuffer: true

        });

        map.addControl(
            new maplibregl.GeolocateControl({
                positionOptions: { enableHighAccuracy: true },
                trackUserLocation: true,
            }),
        );
        map.addControl(new maplibregl.NavigationControl());

        const homeButton = document.createElement("button");
        homeButton.className = "maplibregl-ctrl-icon maplibregl-ctrl-home ";
        homeButton.type = "button";
        homeButton.onclick = resetMap;
        homeButton.title = "Reset to Home";

        const navigationControl = document.querySelector(
            ".maplibregl-ctrl-top-right .maplibregl-ctrl-group",
        );
        if (navigationControl) {
            navigationControl.insertBefore(
                homeButton,
                navigationControl.firstChild,
            );
        }

        map.on("click", (e) => {
            if ($isChecked) {
                addMarker(e.lngLat.lat, e.lngLat.lng);
            }
        });

        map.on("load", async () => {
            // addGeofencePolygon(geofencePolygon);
            await getInitialData();
            addNeighborhoodPoints();
            initLoadLocations(map);
            directions = new MapLibreGlDirections(map);
            directions.clear();
            map.addControl(new LoadingIndicatorControl(directions));
        });

        document.getElementById('searchBar').addEventListener('input', handleSearch);
    });

    // const polygonCoordinates = [
    //     [-96.0, 25.0],
    //     [-92.5, 28.0],
    //     [-95.5, 30.0],
    //     [-97.0, 32.0],
    //     [-100.0, 31.0],
    //     [-100.0, 29.0],
    //     [-96.0, 25.0],
    // ];

    // const geofencePolygon = turf.polygon([polygonCoordinates]);

    // function addGeofencePolygon(polygon) {
    //     if (map.getSource("geofence")) {
    //         map.getSource("geofence").setData(polygon);
    //     } else {
    //         map.addSource("geofence", {
    //             type: "geojson",
    //             data: polygon,
    //         });

    //         map.addLayer({
    //             id: "geofence-border",
    //             type: "line",
    //             source: "geofence",
    //             paint: {
    //                 "line-color": "#FF0000",
    //                 "line-width": 2,
    //             },
    //         });
    //     }
    // }

    async function addNeighborhoodPoints() {
        try {
            const response = await fetch(
                "https://data.austintexas.gov/resource/a7ap-j2yt.geojson",
            );
            const geojson = await response.json();
            map.addSource("source", { type: "geojson", data: geojson });
            map.addLayer({
                id: "polygons",
                type: "fill",
                source: "source",
                paint: {
                    "fill-color": "#888888",
                    "fill-outline-color": "red",
                    "fill-opacity": 0.5,
                },
                filter: ["==", "$type", "Polygon"],
            });

            const popup = new maplibregl.Popup({
                closeButton: false,
                closeOnClick: false,
            });

            // map.on("mouseover", "polygons", (e) => {
            //     map.getCanvas().style.cursor = "pointer";
            //     const description = e.features[0].properties.neighname;
            //     popup.setLngLat(e.lngLat).setHTML(description).addTo(map);
            // });

            map.on("click", "polygons", (e) => {
                map.getCanvas().style.cursor = "pointer";
                const description = e.features[0].properties.neighname;
                popup.setLngLat(e.lngLat).setHTML(description).addTo(map);
            });

            map.on("mouseleave", "polygons", () => {
                map.getCanvas().style.cursor = "";
                popup.remove();
            });
        } catch (error) {
            console.error("Error fetching neighborhood data:", error);
        }
    }

    function handleSearch(event) {
        query = event.target.value.toLowerCase();
        searchLocations(query).then(results => {
            updateLocationList(results, map);
        }).catch(error => {
            console.error('Search error:', error);
        });
    }

    function addMarker(lat, lng) {
        if ($firstMarker.length === 0) {
            const fMarker = new maplibregl.Marker()
                .setLngLat([lng, lat])
                .addTo(map);
            firstMarker.set([fMarker]);
        } else {
            $firstMarker.forEach((marker) => marker.remove());
            // firstMarker.set([]);
        }
        updateMap(lat, lng, map,'',3);
    }


    function resetMap() {
        map.flyTo({ center: originalCenter, zoom: originalZoom });
    }

    const originalCenter = [-97.69, 30.28];
    const originalZoom = 10;
    let i = 0;
    let j = 0;
    let query = '';
</script>

<Controls bind:map bind:directions />
<!-- <div style="display: flex; flex-direction: column;"> -->
<div id="map">
</div>

<div id="locations" >
    <h2>Location List</h2>
    <input type="text" id="searchBar" placeholder="Search by name, type, or rating...">
    <ul id="locationList"></ul>
<!-- </div> -->


<!-- <div id="rChart" style="width: 300px; height: 400px;">
    <button on:click={() => showChart.set(false)}>X</button>
    <canvas id="ratingsChart" style="width: 300px; height: 300px;"
    ></canvas>
    <h2>Rating Pie Chart</h2>
</div> -->

</div>
<!-- <Child {name} {age} {email} bind:name bind:age bind:email /> -->
<!-- <div class="maplibregl-ctrl maplibregl-ctrl-group">
    <button class="maplibregl-ctrl-button maplibregl-ctrl-home" on:click={resetMap}></button>
    
</div> -->
<!-- <div id="map" style="width: 100%; height: 100%;"></div> -->
