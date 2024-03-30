---
theme: dashboard
title: demo
sidebar: false
toc: false

sql:
   locations: ./data/nyc_provider_subset.csv
---

```js
    var sisterAlice = 32;
    var priceInput  =  Inputs.range([0, 2000], {label: "Max price", step: 25, });
    var priceFilter =  Generators.input(priceInput);
```

```sql id=tmpTable
select all_rates, business_name, first_line_location_address, city_name, longitude, latitude 
from locations where all_rates < ${priceFilter} order by all_rates desc
```

<!DOCTYPE html>
<head>
    <title>sidebar-v2 example</title>
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
            background: rgba(51,51,51,0.7);
            z-index: 10;
        }
    </style>
</head>
<body>
<!-- <script src="https://cdn.jsdelivr.net/npm/htl@0.3.1/dist/htl.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@observablehq/inputs@0.10.6/dist/inputs.min.js"></script>
<script src="https://unpkg.com/@observablehq/stdlib@3.24.0/dist/stdlib.js"></script>
<script type="module" src="components/test.js"></script>
<script>
    const sisterAlice = 55;
    const priceFilter  =  Generators.input(Inputs.range([0, 2000], {label: "Max price", step: 25, }));
    //const priceFilter =  Generators.input(priceInput);
</script>
<script>
    const tony = 3;
</script> -->
<!-- optionally define the sidebar content via HTML markup -->
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
                Exploring MRI Pricing in New York City
                <span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span>
            </h1>
            <div class="card" id="histogram" style="display: flex; flex-direction: column; gap: 1rem;">
                ${priceInput}
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
        <div class="leaflet-sidebar-pane" id="autopan">
            <h1 class="leaflet-sidebar-header">
                autopan
                <span class="leaflet-sidebar-close"><i class="fa fa-caret-left"></i></span>
            </h1>
            <p>
                <code>Leaflet.control.sidebar({ autopan: true })</code>
                makes shure that the map center always stays visible.
            </p>
            <p>
                The autopan behviour is responsive as well.
                Try opening and closing the sidebar from this pane!
            </p>
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
    map.setView([40.744, -73.975], 13);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'Map data &copy; OpenStreetMap contributors'
    }).addTo(map);
    L.marker([51.2, 7]).addTo(map);
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
</body>