<template>
  <div class="map" ref="chartdiv">
  </div>
</template>

<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4maps from "@amcharts/amcharts4/maps";
import am4geodata_worldUltra from "@amcharts/amcharts4-geodata/worldUltra";

// France geodata
import geodata_FR_regions from "../assets/FR_regions.json";

export default {
  name: 'WorldMap',
  mounted() {
    let chart = am4core.create(this.$refs.chartdiv, am4maps.MapChart);

    // Set map definition
    chart.geodata = am4geodata_worldUltra;

    // Set projection 
    chart.projection = new am4maps.projections.Mercator();

    // Create map polygon series
    let polygonSeries = chart.series.push(new am4maps.MapPolygonSeries());
 
    // Make map load polygon (like country names) data from GeoJSON
    polygonSeries.useGeodata = true;

    // Configure series
    let polygonTemplate = polygonSeries.mapPolygons.template;
    polygonTemplate.applyOnClones = true;
    polygonTemplate.togglable = true;
    polygonTemplate.tooltipText = "{name}";
    polygonTemplate.propertyFields.fill = "fill";
    polygonTemplate.nonScalingStroke = true;
    polygonTemplate.strokeOpacity = 0.5;
    polygonTemplate.fill = chart.colors.getIndex(0);
    let lastSelected;
    polygonTemplate.events.on("hit", function(ev) {
      if (lastSelected) {
        // This line serves multiple purposes:
        // 1. Clicking a country twice actually de-activates, the line below
        //    de-activates it in advance, so the toggle then re-activates, making it
        //    appear as if it was never de-activated to begin with.
        // 2. Previously activated countries should be de-activated.
        lastSelected.isActive = false;
      }
      ev.target.series.chart.zoomToMapObject(ev.target);
      if (lastSelected !== ev.target) {
        lastSelected = ev.target;
      }
    })

    // Create selected and hover states and set alternative fill color
    let ss = polygonTemplate.states.create("active");
    ss.properties.fill = chart.colors.getIndex(2);

    let hs = polygonTemplate.states.create("hover");
    hs.properties.fill = chart.colors.getIndex(4);

    // Hide Antarctica
    polygonSeries.exclude = ["AQ"];

    // Zoom correction on glitchy countries
    polygonSeries.data = [{
      "id": "NZ",
      "zoomLevel": 11.5,
      "zoomGeoPoint": {
        "latitude": -41,
        "longitude": 173
      }
    }, {
      "id": "RU",
      "zoomLevel": 3,
      "zoomGeoPoint": {
        "latitude": 62,
        "longitude": 97
      }
    }, {
      "id": "US",
      "zoomLevel": 2.9,
      "zoomGeoPoint": {
        "latitude": 51,
        "longitude": -115
      } 
    }];
    
    polygonSeries.dataFields.zoomLevel = "zoomLevel";
    polygonSeries.dataFields.zoomGeoPoint = "zoomGeoPoint";

    // Small map
    chart.smallMap = new am4maps.SmallMap();
    chart.smallMap.series.push(polygonSeries);

    // Zoom control
    chart.zoomControl = new am4maps.ZoomControl();

    let homeButton = new am4core.Button();
    homeButton.events.on("hit", function(){
      chart.goHome();
    });

    homeButton.icon = new am4core.Sprite();
    homeButton.padding(7, 5, 7, 5);
    homeButton.width = 30;
    homeButton.icon.path = "M16,8 L14,8 L14,16 L10,16 L10,10 L6,10 L6,16 L2,16 L2,8 L0,8 L8,0 L16,8 Z M16,8";
    homeButton.marginBottom = 10;
    homeButton.parent = chart.zoomControl;
    homeButton.insertBefore(chart.zoomControl.plusButton);

    // Wine regions
    let FR_regions = chart.series.push(new am4maps.MapPolygonSeries());
    FR_regions.geodata = geodata_FR_regions;
    FR_regions.id = "FR_regions";
    //polygonSeries.getPolygonById("FR")
    let FR_regionsTemplate = FR_regions.mapPolygons.template;
    FR_regionsTemplate.tooltipText = "{name}";
    FR_regionsTemplate.nonScalingStroke = true;
    FR_regionsTemplate.strokeOpacity = 0.5;
    FR_regionsTemplate.propertyFields.fill = "fill";
    FR_regionsTemplate.propertyFields.minZoomLevel = "minZoomLevel";

    // Handle hiding regions on initial load and zoom events
    chart.events.on("datavalidated", updateRegionVisibility);
    chart.events.on("zoomlevelchanged", updateRegionVisibility);

    // Don't show wine regions unless the country is zoomed in on
    function updateRegionVisibility(ev) {
      let chart = ev.target.baseSprite;
      let series = chart.map.getKey("FR_regions");
      series.mapPolygons.each(function(region) {
        if (chart.zoomLevel <= region.minZoomLevel) {
          region.hide();
        }
        else {
          region.show();
        }
      });
    }
    this.chart = chart;
  },

  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map {
  width: 100%;
  height: 100vh;
}
</style>