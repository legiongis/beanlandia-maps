<script>
    import {onMount} from 'svelte';
    import 'ol/ol.css';

    import { browser } from '$app/environment';

    // import filter1893 from '$lib/filters/1893.geojson';
    // import filter1908 from '$lib/filters/1908.geojson';

    import Map from 'ol/Map';
    import View from 'ol/View';

    import GeoTIFF from 'ol/source/GeoTIFF';
    import OSM from 'ol/source/OSM';
    import XYZ from 'ol/source/XYZ';

    import TileLayer from 'ol/layer/Tile';
    import {WebGLTile as WebGLTileLayer} from 'ol/layer';
    import LayerGroup from 'ol/layer/Group';

    import Crop from 'ol-ext/filter/Crop';

    const mbk = 'pk.eyJ1IjoibGVnaW9uZ2lzIiwiYSI6ImNsYmNvazRvdTB2YWQzdm50YzRmcG5wYjAifQ.eOGJmZJHrXLo46_yTdftqQ'
    const mbSatellite = new TileLayer({
        source: new XYZ({
            url: 'https://api.mapbox.com/styles/v1/mapbox/satellite-streets-v10/tiles/{z}/{x}/{y}?access_token='+mbk,
            tileSize: 512,
        })
    });

    const osmLayer = new TileLayer({
        source: new OSM(),
    })

    let pi;
    $: showPm = pi == "beans" ? false : true;

    let showAboutPanel = false;
    let showLayerPanel = true;
    let layerBtnLabel;
    $: showLayerPanel ? layerBtnLabel = "×" : layerBtnLabel = "•••"

    function generateCogLyr(fileName) {
        const s3base = "https://legion-maps.us-southeast-1.linodeobjects.com/beanlandia/";
        return new WebGLTileLayer({
            source: new GeoTIFF({
                sources: [{ url: s3base + fileName }]
            }),
            // tileSize: 512,
        })
    }

    const lyr1893_part1 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376189304-03376_04_1893-0147_opt.tif")
    const lyr1893_part2 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376188502-03376_02_1885-0051R-crop_opt.tif")
    // this if browser is necessary so the block isn't run on the server side
    // and I don't really understand what that means in this context.
    if (browser) {
        // The idea here is to a GeoJSON feature as a crop filter on the layer
        // However, when I apply it there is a "ctx.save() is not a function" error
        // in the console, and I think it probably has to do with the fact that
        // these are WebGL layers, not standard TileLayers. So, abandoning this for now. 
        // fetch("/data/1893.geojson")
        // .then(response => response.json())
        // .then(result => {
        //     const features = new GeoJSON().readFeatures(result)
        //     features.forEach( function (feature) {
        //         feature.getGeometry().transform("EPSG:4326", "EPSG:3857")
        //     	const crop = new Crop({ 
        //     		feature: feature, 
        //     		wrapX: true,
        //     		inner: false
        //     	});
        //         if (feature.getProperties().id == "v2") {
        //             lyr1893_part2.addFilter(crop)
        //         }
		// 		// newLayer.addFilter(crop);
        //     })
		// 		// 
        // });
    }

    const lyr1893 = new LayerGroup({
        layers: [
            lyr1893_part1,
            lyr1893_part2,
        ]
    })
    const lyr1896 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376189604-03376_04_1896-0347_opt.tif")
    const lyr1908_part1 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376190801-03376_01_1908-0029_opt.tif")
    const lyr1908_part2 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376190801-03376_01_1908-0030_opt.tif")
    const lyr1908 = new LayerGroup({
        layers: [
            lyr1908_part1,
            lyr1908_part2,
        ]
    })
    const lyr1937 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376193709-03376_09_1937-0907_opt.tif")
    const lyr1950 = generateCogLyr("service-gmd-gmd401m-g4014m-g4014nm-g03376195009-03376_09_1950-0907_opt.tif")


    const layerLookup = {
        sanborn1950: {
            name: "Sanborn 1950",
            visible: true,
            opacity: 100,
            layer: lyr1950,
            locLink: "https://www.loc.gov/resource/g4014nm.g03376195009/?sp=9&st=image",
        },
        sanborn1937: {
            name: "Sanborn 1937",
            visible: false,
            opacity: 100,
            layer: lyr1937,
            locLink: "https://www.loc.gov/resource/g4014nm.g03376193709/?sp=9&st=image",
        },
        sanborn1908: {
            name: "Sanborn 1908",
            visible: false,
            opacity: 100,
            layer: lyr1908,
            locLink: "https://www.loc.gov/resource/g4014nm.g03376190801/?sp=31&st=image",
        },
        sanborn1896: {
            name: "Sanborn 1896",
            visible: false,
            opacity: 100,
            layer: lyr1896,
            locLink: "https://www.loc.gov/resource/g4014nm.g03376189604/?sp=26&st=image",
        },
        sanborn1893: {
            name: "Sanborn 1893",
            visible: false,
            opacity: 100,
            layer: lyr1893,
            locLink: "https://www.loc.gov/resource/g4014nm.g03376189304/?sp=65&st=image",
        },
    }

    $: {
        for (const id in layerLookup) {
            if (layerLookup.hasOwnProperty(id)) {
                layerLookup[id].layer.setVisible(layerLookup[id].visible)
                if (layerLookup[id].opacity != 0) {
                    layerLookup[id].layer.setOpacity(layerLookup[id].opacity/100)
                }
            }
        }
    }

    onMount(() => {
        const map = new Map({
            target: 'map',
            layers: [
                mbSatellite,
            ],
            view: new View({
                center: [-10023504, 3498745],
                zoom: 19,
            })
        })
        for (const id in layerLookup) {
            if (layerLookup.hasOwnProperty(id)) {
                map.addLayer(layerLookup[id].layer)
            }
        }
        // map.on('moveend', function() {
        //     console.log(map.getView().getCenter());
        //     console.log(map.getView().getZoom());
        // });
    });

</script>
{#if showPm}
<div class="p-modal-bg">
    <div class="p-modal-content">
        <p>password</p>
        <input type="text" bind:value={pi}>
    </div>
</div>
{/if}
{#if showAboutPanel}
<div class="about-modal-bg">
    <div class="about-modal-content">
        <h1>Map Layers</h1>
        <p>These are historical fire insurance maps made by the Sanborn Map Company. the files used for this project are from the Library of Congress <a href="https://loc.gov/collections/sanborn-maps/about-this-collection">digital collection</a>.</p>
        <ul style="padding-left: 20px;">
            <li>1893 Vol. 4 <a href="https://www.loc.gov/resource/g4014nm.g03376189304/?sp=65&st=image">Sheet 147</a> + piece of 1885 Vol. 2 <a href="https://www.loc.gov/resource/g4014nm.g03376189304/?sp=65&st=image">Sheet 51</a></li>
            <li>1896 Vol. 4 <a href="https://www.loc.gov/resource/g4014nm.g03376189604/?sp=26&st=image">Sheet 347</a></li>
            <li>1908 Vol. 1 <a href="https://www.loc.gov/resource/g4014nm.g03376190801/?sp=30&st=image">Sheet 29</a> + <a href="https://www.loc.gov/resource/g4014nm.g03376190801/?sp=31&st=image">Sheet 30</a></li>
            <li>1937 Vol. 9 <a href="https://www.loc.gov/resource/g4014nm.g03376193709/?sp=9&st=image">Sheet 907</a></li>
            <li>1950 vol. 9 <a href="https://www.loc.gov/resource/g4014nm.g03376195009/?sp=9&st=image">Sheet 907</a>*</li>
        </ul>
        <p><em>*The 1950 sheet is actually from a 1937 atlas that had been kept up-to-date with a paste-on correction slip system that the Sanborn Map Company used. Here is the <a href="https://www.loc.gov/resource/g4014nm.g03376195009/?sp=1&st=image&clip=201,4558,1859,1379&ciw=1024&rot=0">Correction Record</a> for that volume.</em></p>
        <p>For broader coverage of New Orleans (and the rest of Louisiana) visit <a href="https://oldinsurancemaps.net">oldinsurancemaps.net</a>.</p> 
        <button on:click={() => {showAboutPanel=false}}>close</button>
    </div>
</div>
{/if}
<div id="map"></div>
<div id="panel-btn"><button on:click={() => {showLayerPanel=!showLayerPanel}} style="{showLayerPanel ? 'border-color:#333; color:#333;' : ''};">{layerBtnLabel}</button></div>
{#if showLayerPanel}
<div id="layer-panel">
    <div class="panel-header"><h1>Before Beanlandia</h1></div>
    <div class="panel-content">
        {#each Object.entries(layerLookup) as [layerId, layerDef]}
        <div class="layer-item">
            <label for={layerId}>
                <input type="checkbox" id={layerId} bind:checked={layerLookup[layerId].visible}>
                {layerDef.name}
            </label>
            <input disabled={!layerLookup[layerId].visible} type=range class="transparency-slider" bind:value={layerLookup[layerId].opacity} min=0 max=100>
        </div>
        {/each}
    </div>
    <hr>
    <div class="panel-footer"><button on:click={() => {showAboutPanel=true}}>about these maps</button></div>
</div>
{/if}
<style>
    #map {
        height: 100vh;
    }
    #panel-btn {
        position: absolute;
        top: .5em;
        right: .5em;
        width: 50px;
        height: 1.5em;
        text-align: center;
        z-index: 1000;
    }

    #panel-btn button {
        color: #666666;
        background: white;
        border-radius: 4px;
        border: 1px solid rgba(255, 255, 255, 0.75);
        cursor: pointer;
        width: 50px;
        font-size: 1.2em;
    }

    #panel-btn button:hover {
        color: #333333;
        border-color: #333333;
    }

    #layer-panel {
        color: #333333;
        position: absolute;
        top: .5em;
        right: .5em;
        max-width: 100%;
        background: white;
        border-radius: 4px;
        border: 1px solid #333333;
        padding: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
        z-index: 999;
    }

    .p-modal-bg {
        position: absolute;
        background: white;
        z-index: 1000000;
        height: 100vh;
        width: 100%;
    }
    .p-modal-content {
        position: absolute;
        top: 3em;
        right: 0;
        left: 0;
        margin: auto;
        width: 200px;
    }

    .about-modal-bg {
        position: absolute;
        background: rgba(255, 255, 255, .6);
        z-index: 999999;
        height: 100vh;
        width: 100%;
    }
    .about-modal-content {
        position: absolute;
        background: white;
        border-radius: 4px;
        top: 3em;
        right: 0;
        left: 0;
        margin: auto;
        width: 400px;
        max-width: 80%;
        padding: 10px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .panel-content {
        width: 100%;
    }

    .layer-item {
        display: flex;
        justify-content: space-between;
    }
</style>