---
theme: dashboard
title: Example dashboard
toc: false

sql:
   locations: ./data/nyc_provider_subset.csv

---

<head>
<link rel = "stylesheet" href = "https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</head>

# Price transparency 🚀

<!-- Load and transform the data -->

```js
const priceFilter = view(
  Inputs.range(
    [0, 1000],
    {label: "Max price", step: 25, }
  )
);

const search = view(
  Inputs.search(
    locations, {placeholder: "Search hospitals and providers…"}));

```

```sql id=tmpTable
select all_rates, business_name, first_line_location_address, city_name, longitude, latitude 
from locations where all_rates < ${priceFilter} order by all_rates desc
```


```js
const launches = FileAttachment("data/launches.csv").csv({typed: true});
const locations = FileAttachment("data/nyc_provider_subset.csv").csv({typed: true});
```

<div id = "megamap" style = "width: 504px; height: 504px"></div>


<!-- <div id = "megamap" style = "width: 900px; height: 580px"></div> -->
<div class="grid grid-cols-2" style="grid-auto-rows: 504px;">
  <div class="card">
     
  </div>
  <div class="card">${
    resize((width) => Inputs.table(tmpTable, {
      columns: [
        "all_rates",
        "business_name",
        "first_line_location_address",
        "city_name",
      ],
      header: {
        all_rates: "In-network rate",
        business_name: "Business name",
        first_line_location_address: "Address",
        city_name: "City",
      }
    }))
  }</div>
</div>


<div class="grid grid-cols-2" style="grid-auto-rows: 504px;">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Price distribution of lower body MRI 🚀",
      subtitle: "Woah!",
      width,
      y: {grid: true, label: "Counts"},
      marks: [
        Plot.ruleY([0]),
        Plot.rectY(tmpTable, Plot.binX({y: "count"}, {x: "all_rates"}))
      ]
    }))
  }</div>
  <div class="card">${
    resize((width) => Inputs.table(tmpTable, {
      columns: [
        "all_rates",
        "business_name",
        "first_line_location_address",
        "city_name",
      ],
      header: {
        all_rates: "In-network rate",
        business_name: "Business name",
        first_line_location_address: "Address",
        city_name: "City",
      }
    }))
  }</div>
</div>




<!-- ```js
const histogram = Plot.plot({
      title: "Price distribution of lower body MRI 🚀",
      subtitle: "Woah!",
      width,
      y: {grid: true, label: "Counts"},
      marks: [
        Plot.ruleY([0]),
        Plot.rectY(tmpTable, Plot.binX({y: "count"}, {x: "all_rates"}))
      ]
    })
display(histogram)
```  -->


```js
import { map } from "https://unpkg.com/leaflet@1.9.4/dist/leaflet-src.esm.js";

// Create a Leaflet map instance
const megamap = map('megamap').setView([40.744, -73.975], 13);

var markers = new Array();

// Add a tile layer
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
}).addTo(megamap);

```


```js

// for (i=0;i<markers.length;i++) {
//   megamap.removeLayer(markers[i]);
// }

var markers = new Array();

// Write a for loop that iterates through tmpTable's long + lat
for (let i = 0; i < tmpTable.batches['0'].data.children['4'].values.length; i++) {
  const data  = tmpTable.get(i).toArray()
  const long  = data[4] // tmpTable.batches['0'].data.children['4'].values[i]
  const lat   = data[5] // tmpTable.batches['0'].data.children['5'].values[i]
  const name  = data[1] // tmpTable.batches['0'].data.children['1'].valueOffsets.buffer
  const price = data[0] 

  var marker = L.marker([lat, long])
  megamap.addLayer(marker)

  marker.bindPopup('<b>' + name + ' ' + price + '</b>')
      .openPopup();

  // marker.addTo(megamap)
  //     .bindPopup('<b>' + name + ' ' + price + '</b>')
  //     .openPopup();

  markers.push(marker)
}

console.log('HIIIII')
```

<!-- <div id = "map" style = "width: 900px; height: 580px"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
  var mapOptions = {center: [17.385044, 78.486671],
                    zoom: 10}

  var map = L.map('map', mapOptions);

  // Set up the OSM layer
  var layer = new L.TileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png');

  L.marker([17.385044, 78.486671]).addTo(map)
      .bindPopup('A pretty CSS popup.<br> Easily customizable.')
      .openPopup();

  // Adding layer to the map
  map.addLayer(layer);
</script> -->
<!-- 
<!-- A shared color scale for consistency, sorted by the number of launches -->
