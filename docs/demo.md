---
theme: dashboard
title: demo
sidebar: false
toc: false

sql:
   locations: ./data/nyc_provider_subset.csv
---

```sql id=tmpTable
    select all_rates, business_name, first_line_location_address, city_name, longitude, latitude, npi 
    from locations where all_rates < ${priceFilter} order by all_rates desc
```

```js
    var priceInput  =  Inputs.range([0, 2000], {label: "Max price", step: 25, });
    var priceFilter =  Generators.input(priceInput);
```

```js
    var searchInput = Inputs.search(tmpTable, {placeholder: "Search for diagnostic tests..."});
```

```js
    const demo_insurance = [  
        {name: "United Healthcare Choice Plus"},
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

        return [price, name, add, city, long, lat, npi]
    } 
```

```js
    var getValueNPI = function (npi) {
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
    const selectedNPI = Mutable(-1);
    const update = () => selectedNPI.value = getValueNPI(selectedID);
```

<!DOCTYPE html>
<head>
    <title>NYC MRI Pricing Example</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
        crossorigin=""/>
    <link rel="stylesheet" href="components/leaflet-sidebar.css" />
    <style>
        body {
            padding: 0;
            margin: 0;
        }
        html, body, #map {
            height: 100%;
            font: 10pt "Helvetica Neue", Arial, Helvetica, sans-serif;
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
            <div class="card" id="searchOptions" style="display: flex; flex-direction: column; gap: 1rem;">
                ${searchInput}
                ${insuranceInput}
                ${priceInput}
            </div>
            <div class="card" id="doctorCardZero" onmouseenter="showMarkerFunction()" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${
                    display(getValueNPI(selectedID))
                }</h2>
            </div>            
            <div class="card" id="doctorCard1" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(1)}</h2>
            </div>
            <div class="card" id="doctorCard" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(2)}</h2>
            </div>
            <div class="card" id="doctorCard" style="display: flex; flex-direction: column; gap: 1rem;">
                <h2>${getValue(3)}</h2>
            </div>
        </div>
        <div class="leaflet-sidebar-pane" id="autopan">
            <h1 class="leaflet-sidebar-header">
                Exploring MRI Pricing in New York City
                <span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span>
            </h1>
            <div class="card" id="histogram" style="display: flex; flex-direction: column; gap: 1rem;">
                ${resize((width) => Plot.plot({
                    title: "Price distribution of lower body MRIs",
                    subtitle: "Woah!",
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
    map.setView([40.744, -73.975], 13);
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

    for (let i = 0; i < tmpTable.batches['0'].data.children['4'].values.length; i++) {
        let [price, name, add, city, long, lat, npi]  = getValue(i);
        var marker = L.marker([lat, long], {npi: npi}).bindPopup('<b>' + name + '</b><br>$' + price + '<br>' + add).addTo(layerGroup).on(
            'click', function(e) {    
                selectedID.length = 0;
                selectedID.push(e.target.options.npi)
            }
        );

        markers.push(marker)

    }
```
</body>