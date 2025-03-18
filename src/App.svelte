<script>
  import { Map } from 'mapbox-gl';
  import 'mapbox-gl/dist/mapbox-gl.css';
  import { onMount, onDestroy } from 'svelte';
 
  let map;
  let mapContainer;
  let lng, lat, zoom;

  lng = -87.661557;
  lat = 41.893748;
  zoom = 11;

  let initialState = { lng, lat, zoom };

  let geojsonData;

  onMount(async () => {
    try {
      const response = await fetch("/chicago-parks.geojson");
      geojsonData = await response.json();
      console.log("GeoJSON Data:", geojsonData);
    } catch (error) {
      console.error("Error loading GeoJSON:", error);
    }

    map = new Map({
      container: mapContainer,
      accessToken: 'pk.eyJ1IjoibW9uaWthLWZ1IiwiYSI6ImNtNmpqMXl2YjAxZTkycXNodTRycmlqNTAifQ.cZoaoIj92IAzZ5jTWYTLbA',
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom
    });

    function updateData() {
      zoom = map.getZoom();
      lng = map.getCenter().lng;
      lat = map.getCenter().lat;
    }

    map.on('move', () => {
      updateData();
    });

    map.on('load', function () {
      map.addSource('chicago-parks', {
        type: 'geojson',
        data: geojsonData
      })

      map.addLayer({
        id: 'parks-data',
        type: 'circle',
        source: 'chicago-parks',
        paint: {
          'circle-radius': 4,
          'circle-stroke-width': 2,
          'circle-color': 'red',
          'circle-stroke-color': 'white'
        }
      });
  });
  });

  onDestroy(() => {
    map.remove();
  });

  function handleReset() {
    map.flyTo({
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom,
      essential: true
    });
  }
</script>

<main>
  <div id="map" bind:this={mapContainer}></div>

  <div class="sidebar">
    Longitude: {lng.toFixed(4)} | Latitude: {lat.toFixed(4)} | Zoom:
    {zoom.toFixed(2)}
  </div>

  <button onclick={handleReset} class="reset-button">Reset</button>

</main>

<style>
  #map {
        display: flex;
        position: absolute;
        padding: 5;
        margin: 2;
        width: 100%;
        height: 100%;
      }

  .sidebar {
      background-color: rgb(35 55 75 / 90%);
      color: #fff;
      padding: 6px 12px;
      font-family: monospace;
      z-index: 1;
      position: absolute;
      top: 0;
      left: 0;
      margin: 12px;
      border-radius: 4px;
    }

    .reset-button {
    position: absolute;
    top: 50px;
    z-index: 1;
    left: 12px;
    padding: 4px 10px;
    border-radius: 10px;
    cursor: pointer;
  }

</style>
