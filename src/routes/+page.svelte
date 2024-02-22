<script>
  import { onMount, afterUpdate } from 'svelte';
  import * as d3 from 'd3';
  import mapboxgl from 'mapbox-gl';
  import RangeSlider from "svelte-range-slider-pips";

  let years = [2008,2023];
  let magns = [6,7];
  let dpths = [0,55];
  let map;
  let data = [];
  let year = 1995;
  let markers = [];
  let popup = new mapboxgl.Popup({ offset: 25 });
  let eqcount = 0;
  let total = 0;

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
    total = data.length;
  });

  function createMarkers(filtered_data) {
    markers = filtered_data.map(d => {
      let color;
      let alertLevel;

      if (d.sig >= 875){
        color = `rgba(120,0,0, 0.7)`;
        alertLevel = 'White';
      } else if (d.sig >= 744){
        color = `rgba(220,0,0, 0.7)`;
        alertLevel = 'White';
      } else if (d.sig >= 691){
        color = `rgba(253,140,0, 0.7)`;
        alertLevel = 'White';
      } else {
        color = `rgba(253,197,0, 0.7)`;
        alertLevel = 'White';
      }

      const marker = new mapboxgl.Marker({
        element: createCustomMarker(color),
      })
        .setLngLat([+d.longitude, +d.latitude])
        .addTo(map);

      marker.getElement().addEventListener('mouseenter', () => {
        popup.setHTML(`
        <p>Location: ${d.location}</p>\
        <p>Time: ${d.date_time}</p>\
        <p>Significance: ${d.sig}</p>\
        <p>Magnitude: ${d.magnitude}</p>\
        <p>Depth: ${d.depth} km</p>
        `);
        marker.setPopup(popup);
        marker.togglePopup();
      });

      return marker;
    });
  }

  function createCustomMarker(color) {
    // Creating a custom marker with a colored circle
    const markerElement = document.createElement('div');
    markerElement.style.width = '20px';
    markerElement.style.height = '20px';
    markerElement.style.borderRadius = "100px";
    markerElement.style.backgroundColor = color;

    return markerElement;
  }
  
  function clearMarkers() {
      markers.forEach(marker => marker.remove());
      markers = [];
  }

  function filterMarkers(yrs,mgs,dps,sgs) {
    filtered_data = data.filter(d => d.year >= yrs[0] && d.year <= yrs[1] &&
                            d.magnitude >= mgs[0] && d.magnitude <= mgs[1] &&
                            d.depth >= dps[0] && d.depth <= dps[1]);
    eqcount = filtered_data.length;
    return filtered_data;
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
      <h1>QuakeChronicle: Unveiling Earth's Tremors - A Visual Exploration of Signficance, Location, and Other Insights</h1>
      <p> How does the geographic location of an earthquake impact its characteristics? </p>
    </div>
    
    <div class ="overlay-slider">
      
      <p style="color: white;">Years: {years[0]} - {years[1]}</p>
      <RangeSlider float min={1995} max={2023} bind:values={years} range={true} step={1}/>
      <p style="color: white;">Magnitude: {magns[0]} - {magns[1]}</p>
      <RangeSlider float min={6} max={10} bind:values={magns} range={true} step={0.1}/>
      <p style="color: white;">Depth(km): {dpths[0]} - {dpths[1]}</p>
      <RangeSlider float min={0} max={700} bind:values={dpths} range={true} step={1}/>
      <p style="color: white;">Earthquakes Displayed: {eqcount} ({(eqcount/total*100).toFixed(1)}%)</p>
  </div>

    <div class ="overlay-legend">
      <p style="text-align: center;">Significance Level</p>
      <p><span class="color-box" style="background: #780000;"></span> &gt;875</p>
      <p><span class="color-box" style="background: #dc0000;"></span> 875-744</p>
      <p><span class="color-box" style="background: #fd8c00;"></span> 744-691</p>
      <p><span class="color-box" style="background: #fdc500;"></span> &lt;691</p>
      <p style="text-align: center;">min: 650, max: 2910</p>
      <p style="font-size: 12px;">Determined by a number of factors, including: magnitude, maximum MMI, felt reports, and estimated impact</p>
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
  width: 40%;


}
.overlay-legend {
  font-size: 0.9em;
	background-color: rgba(0, 0, 0, 0.6);
  position: absolute;
  min-width: 200px;
  width: 10%;
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