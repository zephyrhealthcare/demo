---
theme: glacier
title: demo
sidebar: false
toc: false

sql:
   locations: ./data/nyc_provider_subset_demo.csv
---

<!-- ```sql id=tmpTable
    select all_rates, business_name, first_line_location_address, city_name, longitude, latitude, npi, api_url, score,review_count, map_area
    from locations where all_rates < ${priceFilter} order by all_rates desc
``` -->

```sql id=tmpTable
    select all_rates, business_name, first_line_location_address, city_name, longitude, latitude, npi
    from locations where all_rates < ${priceFilter} order by all_rates desc
```

```js
    var priceInput  =  Inputs.range([50, 4000], {label: "Max price", step: 25, });
    var priceFilter =  Generators.input(priceInput);
```

```js
    // const demo_locations = [  
        // {name: "72195: MRI of the Pelvis, Without Contrast"},
        // {name: "72196: MRI of the Pelvis, With Contrast"},
    //     {name: "NYC"},
    //     {name: "Bay Area"},
    // ]
    // var locationInput = Inputs.select(demo_locations, {label: "Search across locations...", format: x => x.name, value: demo_locations.find(t => t.name === "NYC")})
```

<!-- ```js
    var searchInput = Inputs.search(tmpTable, {placeholder: "Search for diagnostic tests..."});
``` -->

```js
    const demo_diagnostics = [  
        // {name: "72195: MRI of the Pelvis, Without Contrast"},
        // {name: "72196: MRI of the Pelvis, With Contrast"},
        {name: "72197: MRI of the Pelvis, With and Without Contrast"},
    ]
    var searchInput = Inputs.select(demo_diagnostics, {label: "Search across diagnostic tests...", format: x => x.name, value: demo_diagnostics.find(t => t.name === "72197: MRI of the Pelvis, With and Without Contrast")})
```

```js
    const demo_insurance = [  
        {name: "United Healthcare Choice Plus"},
        // {name: "Aetna Choice Plus"},
        // {name: "Humana Choice EPO"},
    ]
    var insuranceInput = Inputs.select(demo_insurance, {label: "Insurance Plan", format: x => x.name, value: demo_insurance.find(t => t.name === "United Healthcare Choice Plus")})
```

```js
    var getValue = function (index) {
        var data  = tmpTable.get(index).toArray();
        let price = data[0];
        let name  = data[1];
        let add   = data[2];
        let city  = data[3];
        let long  = data[4];
        let lat   = data[5];
        let npi   = data[6];
        let api_url      = data[7];
        let score        = data[8];
        let review_count = data[9];

        return [price, name, add, city, long, lat, npi, api_url, score, review_count]
    } 
```


```js
    var getLength = function () {
        return tmpTable.toArray().length
    }
```

```js
    var getValueNPI = function (npi) {
    console.log(npi);
    for (let i = 0; i < tmpTable.batches['0'].data.children['4'].values.length; i++) {
        var data  = tmpTable.get(i).toArray();
        if (npi == data[6]) {
            let price = data[0];
            let name  = data[1];
            let add   = data[2];
            let city  = data[3];
            let long  = data[4];
            let lat   = data[5];
            return [price, name, add, city, long, lat, npi]
            }
        }
    return -1
    }
```

```js
    var greenIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });
```

<!DOCTYPE html>
<head>
    <title>NYC MRI Pricing Example</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
        crossorigin=""/>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
    <link rel="stylesheet" href="components/leaflet-sidebar.css" />
    <style>
        body {
            padding: 0;
            margin: 0;
        }
        html, body, #map {
            height: 100%;
            font-family: 'Roboto', sans-serif;
            /* font: 10pt "Helvetica Neue", Arial, Helvetica, sans-serif; */
        }
        .lorem {
            font-style: italic;
            text-align: justify;
            color: #AAA;
        }
        .full_map {
            position: fixed;
            width: 100%;
            height: 100%;
            left: 0;
            top: 0;
            background: rgba(0,0,0,0.7);
            z-index: 10;
        }
        .card-custom {
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            background-color: #fff;
            padding: 16px;
            display: flex;
            align-items: flex-start;
            position: relative;
        }
        .card-content {
            flex: 1;
            padding-right: 16px;
        }
        .business-name {
            font-size: 18px;
            font-weight: 700; /* Bold */
            margin-bottom: 8px;
        }
        .address {
            font-size: 14px;
            color: #666;
            margin-bottom: 8px;
        }
        .price {
            font-size: 18px;
            font-weight: 450; /* Medium */
            color: #2196F3;
            /* margin-top: auto;
            position: absolute;
            bottom: 69px;
            left: 260px; */
        }
        .phone {
            font-size: 14px;
            font-weight: 700;
            margin-bottom: 8 px;
        }
        .review {
            font-size: 14px;
            font-weight: 700;
            margin-bottom: 8 px;
        }
    </style>
</head>
<body>
<div id="sidebar" class="leaflet-sidebar collapsed">
    <!-- nav tabs -->
    <div class="leaflet-sidebar-tabs">
        <!-- top aligned tabs -->
        <ul role="tablist">
            <li><a href="#home" role="tab"><i class="fa fa-bars active"></i></a></li>
            <li><a href="#autopan" role="tab"><i class="fa fa-arrows"></i></a></li>
        </ul>
        <!-- bottom aligned tabs -->
        <ul role="tablist">
            <li><a href="https://github.com/nickpeihl/leaflet-sidebar-v2"><i class="fa fa-github"></i></a></li>
        </ul>
    </div>
    <!-- panel content -->
    <div class="leaflet-sidebar-content">
        <div class="leaflet-sidebar-pane" id="home">
        <h1 class="leaflet-sidebar-header">
            Explore MRI Pricing across NYC
            <span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span>
        </h1>
            <div class="card" id="searchOptions" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>Search options:</h2>
                ${searchInput}
                ${insuranceInput}
                ${priceInput}
            </div>
            <div class="card" id="displayOnClick" style="display: none; flex-direction: column; gap: 1rem;">
                <h2 id="selectedLocation"></h2>
            </div>
            <!-- 
            <div class="card" id="doctorCard0" onmouseenter="showMarkerFunction()" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(0)}</h2>
            </div>            
            <div class="card" id="doctorCard1" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(1)}</h2>
            </div>
            <div class="card" id="doctorCard2" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(2)}</h2>
            </div>
            <div class="card" id="doctorCard3" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(3)}</h2>
            </div>
            -->
        </div>
        <div class="leaflet-sidebar-pane" id="autopan">
            <h1 class="leaflet-sidebar-header">
                Exploring MRI Pricing in New York City
                <span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span>
            </h1>
            <div class="card" id="histogram" style="display: flex; flex-direction: column; gap: 1rem;">
                ${resize((width) => Plot.plot({
                    title: "Price distribution of lower body MRIs",
                    width,
                    y: {grid: true, label: "Counts"},
                    marks: [
                        Plot.ruleY([0]),
                        Plot.rectY(tmpTable, Plot.binX({y: "count"}, {x: "all_rates"}))
                    ]
                    }))}
            </div>
            <div class="card" id="table">${
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
    </div>
        </div>
        <div class="leaflet-sidebar-pane" id="messages">
            <h1 class="leaflet-sidebar-header">Messages<span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span></h1>
        </div>
    </div>
</div>
<div id="map" class = "full_map"></div>
<script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"
    crossorigin=""></script>
<script src="components/leaflet-sidebar.js"></script>
<script>
    // standard leaflet map setup
    var map = L.map('map');
    var layerGroup = L.layerGroup().addTo(map);
    var markers = [];
    var selectedID = [];
    var selectedMarker = [];
    map.setView([40.744, -73.975], 13);
    map.invalidateSize()
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'Map data &copy; OpenStreetMap contributors'
    }).addTo(map);
    // L.marker([40.744, -73.975]).addTo(layerGroup);
    // create the sidebar instance and add it to the map
    var sidebar = L.control.sidebar({ container: 'sidebar' })
        .addTo(map)
        .open('home');
    // add panels dynamically to the sidebar
    sidebar
        .addPanel({
            id:   'js-api',
            tab:  '<i class="fa fa-gear"></i>',
            title: 'JS API',
            pane: '<p>The Javascript API allows to dynamically create or modify the panel state.<p/><p><button onclick="sidebar.enablePanel(\'mail\')">enable mails panel</button><button onclick="sidebar.disablePanel(\'mail\')">disable mails panel</button></p><p><button onclick="addUser()">add user</button></b>',
        })
        // add a tab with a click callback, initially disabled
        .addPanel({
            id:   'mail',
            tab:  '<i class="fa fa-envelope"></i>',
            title: 'Messages',
            button: function() { alert('opened via JS callback') },
            disabled: true,
        })
    // be notified when a panel is opened
    sidebar.on('content', function (ev) {
        switch (ev.id) {
            case 'autopan':
            sidebar.options.autopan = true;
            break;
            default:
            sidebar.options.autopan = false;
        }
    });
    var userid = 0
    function addUser() {
        sidebar.addPanel({
            id:   'user' + userid++,
            tab:  '<i class="fa fa-user"></i>',
            title: 'User Profile ' + userid,
            pane: '<p>user ipsum dolor sit amet</p>',
        });
    }
</script>

<script>
    var showMarkerFunction = function(npi) {
        console.log('gottohere')
        for (var i in markers) {
            var markerID = markers[i].options.npi;
            if (markerID == npi) {
                markers[i].openPopup();
            }
        }
        return 0
    }
</script>

```js
    layerGroup.clearLayers();
    markers.length = 0;

    var blueIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });

    var redIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });

    // .bindPopup('<b>' + name + '</b><br>$' + price + '<br>' + add)
    // tmpTable.batches['0'].data.children['4'].values.length
    var currLength = getLength()
    for (let i = 0; i < currLength; i++) {
        let [price, name, add, city, long, lat, npi, api_url, score, review_count]  = getValue(i);
        var marker = L.marker([lat, long], {price: price, name: name, add: add, city: city, npi: npi, api_url: api_url, score: score, review_count: review_count}).addTo(layerGroup).on(
            'click', function(e) {
                if (selectedMarker.length > 0) {
                    selectedMarker[0].setIcon(blueIcon)
                }   
                selectedID.length = 0;
                selectedID.push(e.target.options.npi)
                this.setIcon(redIcon)
                document.getElementById("displayOnClick").style.display = "flex";
                document.getElementById("selectedLocation").innerHTML = '<div class="card-custom"><div class="card-content">' + '<div class="business-name">' + e.target.options.name + '</div>' + '<div class="address">' + e.target.options.add + ', ' + e.target.options.city + '</div>' + '</div>' +  '<div class="price">$' + e.target.options.price + '</div>' + '</div>'
                selectedMarker.length = 0;
                selectedMarker.push(this);
            }
        );

        markers.push(marker)

    }

    map.invalidateSize()
```
</body>

<!-- + '<div class="review">' + '<a href="google.com' + '">3.0' + '</a>' + '</div>'  -->