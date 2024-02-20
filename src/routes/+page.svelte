<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
    import mapboxgl from 'mapbox-gl';
  
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
        center: [-71.0942, 42.3601],
        zoom: 2.5,
        minZoom: 0,
        maxZoom: 15,
      });
  
      data = await d3.csv('https://raw.githubusercontent.com/patsals/earthquake_visualization_website/main/data/earthquake_1995-2023.csv');
    });
  
    afterUpdate(() => {
      if (markers.length > 0) {
        // Add event listener on the map to close popup on mouse leave
        map.on('mouseleave', () => {
          if (popup.isOpen()) {
            popup.remove();
          }
        });
      }
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
          popup.setHTML(`<p>Location: ${d.location}</p><p>Coordinates: ${d.latitude}, ${d.longitude}</p><p>Alert Level: ${alertLevel}</p>`);
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

    function filterMarkers(sliderYear) {
      return data.filter(d => new Date(d.date_time).getFullYear() === sliderYear);
    }
  
    let filtered_data = [];
    $: {
      if (data.length !== 0) {
        clearMarkers();
        filtered_data = filterMarkers(year);
        createMarkers(filtered_data);
      }
    };
  </script>
  
  <style>
    #map {
      height: 1000px;
    }
  </style>
  
  <main>
    <input type="range" bind:value={year} min="1995" max="2023">
    <p>Year: {year}</p>
    <div id="map"></div>
  </main>
  