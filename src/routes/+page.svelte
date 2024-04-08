<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    mapboxgl.accessToken = "pk.eyJ1IjoianVsaWFjYyIsImEiOiJjbHVyNWo0aDcwM2p4MmlvM2RmNDN3NnA3In0.akewVVgfBwvektMVj8KnfQ";

    import { onMount } from "svelte";

    onMount(() => {
        let map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/mapbox/streets-v12",
            center: [-71.09415, 42.36027],
            zoom: 12
        });
    })

</script>

<h1>Bikewatching</h1>
<p>This page displays Boston's bike traffic throughout the day.</p>
<div id="map" />

<style>
    @import url("$lib/global.css");
</style>