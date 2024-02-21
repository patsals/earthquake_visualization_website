<script>
  import { onMount, afterUpdate } from 'svelte';
  import * as d3 from 'd3';
  import mapboxgl from 'mapbox-gl';
  import RangeSlider from "svelte-range-slider-pips";

  let years = [1999,2018];
  let magns = [7,9];
  let dpths = [100,600];
  let map;
  let data = [];
  let year = 1995;
  let markers = [];
  let popup = new mapboxgl.Popup({ offset: 25 });

  onMount(async () => {
    mapboxgl.accessToken = "pk.eyJ1IjoiYXJzMDE4IiwiYSI6ImNsc2s1emhjcTAwMzgyaHF1aWFucWtheHEifQ.PUu2vb6c2ZjdGvHjaQKK_g";
    map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/psalsbury/clsoefgeo009u01rb5imr8698",
      center: [0, 0],
      zoom: 2.3,
      minZoom: 0,
      maxZoom: 15,
    });

    data = await d3.csv('https://raw.githubusercontent.com/is-patel/eq-vis/main/static/eq_vis.csv');
  });

  function createMarkers(filtered_data) {
    markers = filtered_data.map(d => {
      let color;
      let alertLevel;
      switch (d.alert) {
        case 'green':
          color = `rgba(255, 208, 0, 0.7)`; // yellow 
          alertLevel = 'Yellow';
          break;
        case 'yellow':
          color = `rgba(245, 110, 0, 0.7)`; // orange 
          alertLevel = 'Orange';
          break;
        case 'red':
          color = `rgba(200, 0, 0, 0.7)`; // red 
          alertLevel = 'Red';
          break;
        default:
          color = `rgba(255, 255, 255, 0.7)`; // white 
          alertLevel = 'White';
          break;
      }

      const marker = new mapboxgl.Marker({
        element: createCustomMarker(color),
      })
        .setLngLat([+d.longitude, +d.latitude])
        .addTo(map);

      marker.getElement().addEventListener('mouseenter', () => {
        popup.setHTML(`<p>Location: ${d.location}</p>\
        <p>Alert Level: ${alertLevel}</p>\
        <p>Time: ${d.date_time}</p>\
        <p>Magnitude: ${d.magnitude}</p>\
        <p>Depth: ${d.depth} km</p>`);
        marker.setPopup(popup);
        marker.togglePopup();
      });

      return marker;
    });
  }

  function createCustomMarker(color) {
    // Creating a custom marker with a colored circle
    const markerElement = document.createElement('div');
    markerElement.style.width = '30px';
    markerElement.style.height = '30px';
    markerElement.style.borderRadius = "100px";
    markerElement.style.backgroundColor = color;

    return markerElement;
  }
  
  function clearMarkers() {
      markers.forEach(marker => marker.remove());
      markers = [];
  }

  function filterMarkers(yrs,mgs,dps) {
    return data.filter(d => d.year >= yrs[0] && d.year <= yrs[1] &&
                            d.magnitude >= mgs[0] && d.magnitude <= mgs[1] &&
                            d.depth >= dps[0] && d.depth <= dps[1]

    );
  }

  let filtered_data = [];
  $: {
    if (data.length !== 0) {
      clearMarkers();
      filtered_data = filterMarkers(years,magns,dpths);
      createMarkers(filtered_data);
    }
  };
</script>



<main>
  <div>
    <div id="map"></div>
    
    <div class='overlay-title'>
      <h1>QuakeChronicle: Unveiling Earth's Tremors - A Visual Exploration of Alertness, Coordinates, and Key Insights</h1>
      <p> question here</p>
    </div>
    
    <div class ="overlay-slider">
      
      <p style="color: white;">Years: {years[0]} - {years[1]}</p>
      <RangeSlider float min={1995} max={2023} bind:values={years} range={true} step={1}/>
      <p style="color: white;">Magnitude: {magns[0]} - {magns[1]}</p>
      <RangeSlider float min={6} max={10} bind:values={magns} range={true} step={0.1}/>
      <p style="color: white;">Depth(km): {dpths[0]} - {dpths[1]}</p>
      <RangeSlider float min={0} max={700} bind:values={dpths} range={true} step={1}/>
  </div>

    <div class ="overlay-legend">
      <p><span class="color-box" style="background: white; color: white;"></span> White: Alert Value Unknown</p>
      <p><span class="color-box" style="background: red;"></span> Red: Severe Activity Alert</p>
      <p><span class="color-box" style="background: orange;"></span> Orange: High Level Activity Alert</p>
      <p><span class="color-box" style="background: yellow;"></span> Yellow: Moderate Activity Alert</p>
    </div>
  </div>

</main>



<style>
 
  #map {
    height: 100vh;
  }

  .overlay-slider {
	font-size: 0.9em;
	background-color: rgba(0, 0, 0, 0.6);
	position: absolute;
	min-width:250px;
	width: 15%;
	top: 20px;
	right: 20px;
	padding: 10px;
	z-index: 3;
 }
 .overlay-title {
  font-size: 0.9em;
  position: absolute;
  min-width: 250px;
  width: 50%;
  top: 5px; /* Adjust the distance from the top as needed */
  left: 20px; /* Set left to 0 to align with the left of the screen */
  /* transform: translateX(-50%); Remove this line if not needed */
  z-index: 3;
  align-items: left; /* This property is not applicable to absolute-positioned elements */
}
.overlay-title h1 {
  text-align: left;
  font-family:'Arial', sans-serif;
  font-size: 2.5em;
  color: black; /* Set the inside color to white */
  text-shadow: -2px -2px 2px white, 2px -2px 2px white, -2px 2px 2px white, 2px 2px 2px white;
  padding: 0px;

}

.overlay-title p {
  text-align: left;
  font-family:'Arial', sans-serif;
  font-size: 1.3em;
  color: white; /* Set the inside color to white */

}
.overlay-legend {
  font-size: 0.9em;
	background-color: rgba(0, 0, 0, 0.6);
  position: absolute;
  min-width: 250px;
  width: 15%;
  bottom: 20px; /* Change top to bottom */
  left: 20px; /* Change right to left */
  padding: 10px;
  z-index: 3;
}

.overlay-legend p {
  font-family:'Arial', sans-serif;
  color: white; /* Set the inside color to white */
}


 p {
  font-family:'Arial', sans-serif
 }

 .color-box {
    display: inline-block;
    width: 20px;
    height: 20px;
    margin-right: 10px;
    vertical-align: middle;
    border: 1px solid black;
  }

</style>