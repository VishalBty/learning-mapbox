<template>
  <div class="map-box">
    <div id="map"></div>
    <div class="map-box__fly-to-box">
      <h5>Fly to?</h5>
      <b-button-group>
        <b-button-group style="gap: 4px">
          <b-button
            variant="primary"
            @click="fly_to(location)"
            v-for="(cordinate, location) in cordinates"
            :key="location"
            :data-cordinates="cordinate"
            class="text-capitalize"
            >{{ location }}</b-button
          >
        </b-button-group>
      </b-button-group>
    </div>

    <b-button
      :variant="!show_home ? 'success' : 'warning'"
      @click="toggle_home_visibility"
      class="text-capitalize map-box__home-visibility-box"
      >{{ show_home ? "Hide" : "Show" }} my Home</b-button
    >
  </div>
</template>

<script>
import "mapbox-gl/dist/mapbox-gl.css";
import mapboxgl from "!mapbox-gl";

export default {
  name: "App",
  data() {
    return {
      loading: false,
      location: "",
      center: [0, 0],
      map: {},

      cordinates: {
        india: {
          lat: 28.7041,
          lng: 77.1025,
        },
        usa: {
          lat: 37.0902,
          lng: -95.7129,
        },
        china: {
          lat: 39.9042,
          lng: 116.4074,
        },
        home: {
          lat: 26.4032,
          lng: 90.26442370714403,
        },
      },
      show_home: false,
      access_token:
        "pk.eyJ1IjoidmlzaGFsYnR5IiwiYSI6ImNqdXNuMmoyNzBsY2U0NHA1NzJlMnlnYmcifQ.zMLy1JfHCeQi766Vx7DcRA",
    };
  },
  components: {},
  mounted() {
    this.load_map();
  },
  methods: {
    fly_to(location) {
      this.map.flyTo({
        center: this.cordinates[location],
        zoom: 4,
      });
    },
    toggle_home_visibility() {
      this.show_home = !this.show_home;
      this.map.setLayoutProperty(
        "home",
        "visibility",
        this.show_home ? "visible" : "none"
      );
    },
    async load_map() {
      try {
        mapboxgl.accessToken = this.access_token;
        const map = new mapboxgl.Map({
          container: "map",
          style: "mapbox://styles/mapbox/streets-v11",
          center: this.center,
          zoom: 1,
        });
        const home = this.cordinates.home;
        map.on("load", () => {
          // Load an image from an external URL.
          map.loadImage("/home.png", (error, image) => {
            if (error) throw error;

            // Add the image to the map style.
            map.addImage("home", image);

            // Add a data source containing one point feature.
            map.addSource("point", {
              type: "geojson",
              data: {
                type: "FeatureCollection",
                features: [
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [home.lng, home.lat],
                    },
                  },
                ],
              },
            });

            // Add a layer to use the image to represent the data.
            map.addLayer({
              id: "home",
              type: "symbol",
              source: "point", // reference the data source
              layout: {
                "icon-image": "home", // reference the image
                "icon-size": 0.1,
                visibility: "none",
              },
            });
          });
        });
        this.map = map;
      } catch (err) {
        console.log("map error", err);
      }
    },
  },
};
</script>

<style lang="scss">
.map-box {
  position: relative;
  #map {
    height: 100vh;
  }
  &__fly-to-box {
    position: absolute;
    top: 20px;
    right: 20px;
    width: 300px;
    height: 100px;
    background: white;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    z-index: 1;
  }
  &__home-visibility-box {
    position: absolute;
    bottom: 40px;
    right: 20px;

    background: white;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    z-index: 1;
  }
}
</style>
