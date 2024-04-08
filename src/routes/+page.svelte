<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    mapboxgl.accessToken = "pk.eyJ1IjoianVsaWFjYyIsImEiOiJjbHVyNWo0aDcwM2p4MmlvM2RmNDN3NnA3In0.akewVVgfBwvektMVj8KnfQ";

    import { onMount } from "svelte";

    onMount(() => {
        createMap();
    });

    async function createMap() {
        let map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/mapbox/streets-v12",
            center: [-71.09415, 42.36027],
            zoom: 12
        });
        await new Promise(resolve => map.on("load", resolve));

        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });

        map.addLayer({
            id: "bikelanes", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                // paint params, e.g. colors, thickness, etc.
            },
        });
    };

</script>

<h1>Bikewatching</h1>
<p>This page displays Boston's bike traffic throughout the day.</p>
<div id="map" />

<style>
    @import url("$lib/global.css");
</style>