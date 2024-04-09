<script>
    import mapboxgl from "mapbox-gl";
    import * as d3 from "d3";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    mapboxgl.accessToken = "pk.eyJ1IjoianVsaWFjYyIsImEiOiJjbHVyNWo0aDcwM2p4MmlvM2RmNDN3NnA3In0.akewVVgfBwvektMVj8KnfQ";

    import { onMount } from "svelte";

    let map;
    let stations = [];
    let trips = [];
    let departures;
    let arrivals;
    let totalTraffic;

    $: filteredTrips = timeFilter === -1? trips : trips.filter(trip => {
        let startedMinutes = minutesSinceMidnight(trip.started_at);
        let endedMinutes = minutesSinceMidnight(trip.ended_at);
        return Math.abs(startedMinutes - timeFilter) <= 60
            || Math.abs(endedMinutes - timeFilter) <= 60;
    });

    $: filteredDepartures = d3.rollup(filteredTrips, v => v.length, d => d.start_station_id);
    
    $: filteredArrivals = d3.rollup(filteredTrips, v => v.length, d => d.end_station_id);

    $: filteredStations = stations.map(station => {
        let id = station.Number;
        station = {...station};
        station.departures = filteredDepartures.get(id) ?? 0;
        station.arrivals = filteredArrivals.get(id) ?? 0;
        station.totalTraffic = station.departures + station.arrivals;
        return station;
    });

    let radiusScale;
    $: radiusScale = d3.scaleSqrt()
	.domain([0, d3.max(stations, d => d.totalTraffic)])
	.range([0, 25]);

    let mapViewChanged = 0;
    $: map?.on("move", evt => mapViewChanged++);

    onMount(async () => {
        await createMap();
        stations = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-stations.csv");
        // trips = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-traffic-2024-03.csv");
        trips = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-traffic-2024-03.csv").then(trips => {
            for (let trip of trips) {
                trip.started_at = new Date(trip.started_at);
                trip.ended_at = new Date(trip.ended_at);
            }
            return trips;
        });

        departures = d3.rollup(trips, v => v.length, d => d.start_station_id);
        arrivals = d3.rollup(trips, v => v.length, d => d.end_station_id);
        stations = stations.map(station => {
            let id = station.Number;
            station.departures = departures.get(id) ?? 0;
            station.arrivals = arrivals.get(id) ?? 0;
            station.totalTraffic = station.departures + station.arrivals;
            return station;
        });
        // console.log("stations:", stations);
    });

    function minutesSinceMidnight (date) {
        return date.getHours() * 60 + date.getMinutes();
    }

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    };

    async function createMap() {
        map = new mapboxgl.Map({
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
            id: "boston_bikelanes", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                "line-color": "green",
                "line-width": 4,
                "line-opacity": 0.4
            },
        });

        map.addSource("cambridge_route", {
            type: "geojson",
            data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson"
        });

        map.addLayer({
            id: "cambridge_bikelanes", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "cambridge_route", // The id we specified in `addSource()`
            paint: {
                "line-color": "green",
                "line-width": 4,
                "line-opacity": 0.4
            },
        });

    };

    let timeFilter = -1;

    $: timeFilterLabel = new Date(0, 0, 0, 0, timeFilter)
                     .toLocaleString("en", {timeStyle: "short"});

    let stationFlow = d3.scaleQuantize()
        .domain([0, 1])
        .range([0, 0.5, 1]);

</script>

<h1>Bikewatching</h1>
<p>This page displays Boston's bike traffic throughout the day.</p>
<label>
    Filter by time:
    <input type=range min=-1 max=1440 bind:value={timeFilter}>
    {#if timeFilter != -1}
        <time>{timeFilterLabel}</time>
    {/if}
    {#if timeFilter == -1}
        <em>(any time)</em>
    {/if}
</label>

<div id="map">
	<svg>
        {#key mapViewChanged}
            {#each filteredStations as station}
                <circle { ...getCoords(station) } r={radiusScale(station.totalTraffic)} 
                style="--departure-ratio: { stationFlow(station.departures / station.totalTraffic) }" />
            {/each}
        {/key}
    </svg>
</div>
<div class="legend">
	<div style="--departure-ratio: 1">More departures</div>
	<div style="--departure-ratio: 0.5">Balanced</div>
	<div style="--departure-ratio: 0">More arrivals</div>
</div>



<style>
    @import url("$lib/global.css");

    #map svg {
        position: absolute;
        z-index: 1;
        width: 100%;
        height: 100%;
        pointer-events: none;
        /* background: yellow;
        opacity: 50%; */
        circle {
            fill: steelblue;
            fill-opacity: 60%;
            stroke: white;
        }
    }

    #map svg circle, .legend > div {
        --color-departures: steelblue;
        --color-arrivals: darkorange;
        --color: color-mix(
            in oklch,
            var(--color-departures) calc(100% * var(--departure-ratio)),
            var(--color-arrivals)
        );
        fill: var(--color);
        background-color: var(--color);
    }

    .legend {
        display: flex;
        justify-content: space-between;
        margin-top: 1em;
        color: white;
        font-weight: bold;
    }

    .legend > div {
        padding-left: 1em;
        padding-right: 1em;
        padding-top: 0.5em;
        padding-bottom: 0.5em;
    }

    em {
        color: gray;
    }

    label {
        margin-bottom: 1em;
    }

</style>