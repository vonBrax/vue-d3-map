<template>
  <div class="map" ref="map">
    <!-- <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 65.5 32.9" width="60" height="30" fill="#FFFFFF">
      <g>
        <path d="M62 .5H3.5c-1.7 0-3 1.3-3 3v20.3c0 1.7 1.3 3 3 3h23.8l5.4 5.4 5.4-5.4H62c1.7 0 3-1.3 3-3V3.5c0-1.7-1.3-3-3-3z" fill="#fff"/>
        <path d="M62 0H3.5C1.6 0 0 1.6 0 3.5v20.3c0 1.9 1.6 3.5 3.5 3.5h23.6l5.6 5.6 5.6-5.6H62c1.9 0 3.5-1.6 3.5-3.5V3.5C65.5 1.6 63.9 0 62 0zm2.5 23.8c0 1.4-1.1 2.5-2.5 2.5H37.9l-5.2 5.2-5.2-5.2h-24c-1.4 0-2.5-1.1-2.5-2.5V3.5C1 2.1 2.1 1 3.5 1H62c1.4 0 2.5 1.1 2.5 2.5v20.3z" fill="#585a51"/>
        <text class="price" x="15" y="18">Hello?</text>
      </g>
    </svg> -->
    <div class="map__controls">
      <button
        type="button"
        class="map__controls__button"
        :disabled="zoom >= MAX_ZOOM"
        @click="zoomIn">
          +
      </button>
      <button
        type="button"
        class="map__controls__button"
        :disabled="zoom <= MIN_ZOOM"
        @click="this.zoomOut">
        -
      </button>
    </div>
    <svg
      v-if="width > 0 && height > 0"
      class="map__svg"
      :viewBox="`0 0 ${width} ${height}`"
      @mousedown="onTouchStart"
      @mousemove="onTouchMove"
      @mouseup="onTouchEnd"
      @mouseleave="onTouchEnd"
      @mousewheel="onWheel">
      <g>
        <!-- 

        :xlink:href="`https://maps.wikimedia.org/osm-intl/${t.z}/${t.x}/${t.y}.png`"
        :xlink:href="`https://korona.geog.uni-heidelberg.de/tiles/roads/x=${t.x}&y=${t.y}&z=${t.z}`"
        -->
        <image
          v-for="(t, i) in tiles"
          :key="`${t.x}_${t.y}_${t.z}_${i}`"
          
          :xlink:href="`https://${servers[i % 3]}.tile.openstreetmap.org/${t.z}/${t.x}/${t.y}.png`"
          :x="(t.x + tiles.translate[0]) * tiles.scale"
          :y="(t.y + tiles.translate[1]) * tiles.scale"
          :width="tiles.scale"
          :height="tiles.scale"
        />
      </g>
      <!--  :transform="translateLayer"  -->
      <g
        id="pathgroup"
        class="countries"
        :transform="translateLayer"
        v-if="topojson"
      >
        <!-- <g
          v-for="(t, i) in geoCentroids"
          :key="`${geoCentroids[i][0]}_${geoCentroids[i][1]}_${i}}`"
          class="labels"
          :x="geoCentroids[i][0]"
          :y="geoCentroids[i][1]"
        >
          <path d="M62 .5H3.5c-1.7 0-3 1.3-3 3v20.3c0 1.7 1.3 3 3 3h23.8l5.4 5.4 5.4-5.4H62c1.7 0 3-1.3 3-3V3.5c0-1.7-1.3-3-3-3z" fill="#fff"/>
          <path d="M62 0H3.5C1.6 0 0 1.6 0 3.5v20.3c0 1.9 1.6 3.5 3.5 3.5h23.6l5.6 5.6 5.6-5.6H62c1.9 0 3.5-1.6 3.5-3.5V3.5C65.5 1.6 63.9 0 62 0zm2.5 23.8c0 1.4-1.1 2.5-2.5 2.5H37.9l-5.2 5.2-5.2-5.2h-24c-1.4 0-2.5-1.1-2.5-2.5V3.5C1 2.1 2.1 1 3.5 1H62c1.4 0 2.5 1.1 2.5 2.5v20.3z" fill="#585a51"/>
          <text
            class="price"
            x="15"
            y="18"
            v-text="topojson.features[i].properties.name"
          />
        </g> -->
        <!--
        xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 65.5 32.9"

        -->
        <svg
          v-for="(t, i) in geoCentroids"
          :key="`${geoCentroids[i][0]}_${geoCentroids[i][1]}_${i}}`"
          class="labels"
          :x="geoCentroids[i][0] - 32.5"
          :y="geoCentroids[i][1] - 40"
          width="66"
          height="32.9"
          fill="#FFFFFF"
        >
          <path d="M62 .5H3.5c-1.7 0-3 1.3-3 3v20.3c0 1.7 1.3 3 3 3h23.8l5.4 5.4 5.4-5.4H62c1.7 0 3-1.3 3-3V3.5c0-1.7-1.3-3-3-3z" fill="#fff"/>
          <path d="M62 0H3.5C1.6 0 0 1.6 0 3.5v20.3c0 1.9 1.6 3.5 3.5 3.5h23.6l5.6 5.6 5.6-5.6H62c1.9 0 3.5-1.6 3.5-3.5V3.5C65.5 1.6 63.9 0 62 0zm2.5 23.8c0 1.4-1.1 2.5-2.5 2.5H37.9l-5.2 5.2-5.2-5.2h-24c-1.4 0-2.5-1.1-2.5-2.5V3.5C1 2.1 2.1 1 3.5 1H62c1.4 0 2.5 1.1 2.5 2.5v20.3z" fill="#585a51" stroke="#585a51"/>
          <text
            class="price"
            x="14"
            y="18"
            v-text="'€1,200'"
          />
        </svg>
        <!-- <text
          x="15"
          y="18"
          v-text="topojson.features[i].properties.name"
        /> -->
      </g>
    </svg>
    <div class="map__copyright">
      ©&nbsp;
      <a
        href="https://www.openstreetmap.org/copyright"
        target="_blank"
        rel="noopener"
      >OpenStreetMap</a>
      &nbsp;contributors
    </div>
  </div>
</template>

<script>
const d3 = {
  ...require('d3-geo'),
  ...require('d3-tile'),
  ...require('d3-selection')
}

export default {
  props: {
    center: {
      type: Array,
      required: false,
      default: () => [19.5033, 47.1625] // Hungary
    },
    initialZoom: {
      type: [Number, String],
      required: false,
      default: 13
    },
  },
  data() {
    return {
      width: 0,
      height: 0,
      zoom: +this.initialZoom,
      MAX_ZOOM: 27,
      MIN_ZOOM: 10,
      scale: 1 << +this.initialZoom,
      translateX: 0,
      translateY: 0,
      touchStarted: false,
      touchLastY: 0,
      touchLastX: 0,
      servers: ['a', 'b', 'c'],
      topojson: null,
      geoCentroids: []
    }
  },
  computed: {
    projection() {
      return d3.geoMercator()
        .scale(+this.scale / (2 * Math.PI))
        .translate([this.translateX, this.translateY])
        .center(this.center)
    },
    tiles() {
      return d3.tile()
        .size([this.width, this.height])
        .scale(+this.scale)
        .translate(this.projection([0, 0]))()
    },
    path() {
      return d3.geoPath(this.projection)
    },
    translateLayer() {
      const x = this.translateX - this.width / 2
      const y = this.translateY - this.height / 2
      // const s = this.scale
      return `translate(${x},${y}) scale(${1})`
    },
    // geoCentroids() {
    //   return this.topojson.features.map(d => this.projection(d3.geoCentroid(d)) )
    // }
  },
  mounted() {
    const rect = this.$el.getBoundingClientRect()
    this.width = rect.width
    this.height = rect.height
    this.translateX = this.width / 2
    this.translateY = this.height / 2
    this.getData()

  },
  methods: {
    calculateCentroids() {
      this.geoCentroids = this.topojson.features.map(d => this.projection(d3.geoCentroid(d)) )
    },
    async getData() {
      const data = await fetch('https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json')
      this.topojson = await data.json()
      
      this.topojson.features = this.topojson.features.filter(d => {
        const countries = {
          Germany: true,
          Austria: true,
          Hungary: true,
          France: true,
          "United Kingdom": true,
          Turkey: true,
          Ireland: true,
          "Czech Republic": true
        }
        return countries[d.properties.name]
      })

      this.calculateCentroids()
      
      // this.geoCentroids = this.topojson.features.map(d => this.projection(d3.geoCentroid(d)) )
      // this.drawPath(this.topojson)
    },
    // drawPath(data){
      
      /** Multiple path elements */
      // d3.select('.countries')
      //   .selectAll('path')
      //   .data(data.features)
      //   .enter().append('path')
      //   .attr('d', this.path)
      
      /** Single path element */
      // d3.select('.countries')
      //   .append('path')
      //   .datum(data)
      //   .attr('d', this.path)

      // d3.select('.countries')
      //   .selectAll('.labels')
      //   .data(data.features).enter().append('text')
      //   .attr('transform', (d) => {
      //     return 'translate(' + this.projection(d3.geoCentroid(d)) + ')'
      //   })
      //   .attr('dy', -3)
      //   .attr('dx', 3)
      //   .text(function(d) {
      //     return d.properties.name
      //   })
      //   .attr('class', 'labels')

   // },
    onTouchStart(e) {
      this.touchStarted = true
      this.touchLastX = e.clientX
      this.touchLastY = e.clientY
    },
    onTouchEnd() {
      this.touchStarted = false
    },
    onTouchMove(e) {
      if (this.touchStarted) {
        this.translateX = this.translateX + e.clientX - this.touchLastX
        this.translateY = this.translateY + e.clientY - this.touchLastY

        this.touchLastX = e.clientX
        this.touchLastY = e.clientY
      }
    },
    zoomIn() {
      this.zoom = Math.min(this.zoom + 1, this.MAX_ZOOM)
      // this.geoCentroids = this.topojson.features.map(d => this.projection(d3.geoCentroid(d)) )
    },
    zoomOut() {
      this.zoom = Math.max(this.zoom - 1, this.MIN_ZOOM)
      // this.geoCentroids = this.topojson.features.map(d => this.projection(d3.geoCentroid(d)) )
    },
    onWheel(e) {
      if (e.deltaY > 0) {
        this.zoomOut()
      } else {
        this.zoomIn()
      }
    }
  },
  watch: {
    zoom(newValue, oldValue) {
      const k = newValue - oldValue > 0 ? 2 : .5;

      this.scale = 1 << newValue
      this.translateX = this.width / 2 - k * (this.width / 2 - this.translateX)
      this.translateY = this.height / 2 - k * (this.height / 2 - this.translateY)
      setTimeout( () => this.calculateCentroids(), 600 )
    }
  }
}
</script>

<style lang="stylus" scoped>
  .price
    fill #585a51
    font bold 13px "NettoWeb W03", "Adelle SansW01", arial, sans-serif
  /*
  .countries
    fill none
    stroke #00c
  */
  .map
    position relative
    width 536px
    height 320px
    &__controls
      position absolute
      left 16px
      top 16px
      display flex
      flex-direction column
      justify-content space-between
      height 56px
      &__button
        cursor pointer
        border 0
        padding 0
        width 24px
        height 24px
        line-height 24px
        border-radius 50%
        font-size 1.15em
        background-color #ffffff
        color #343434
        box-shadow 0 1px 4px rgba(0, 0, 0, 0.4)
        &:hover,
        &:focus
          background-color #eeeeee

        &:disabled
          background-color rgba(#eeeeee, 0.4)
    &__svg
      max-width 100%
      max-height 100%
    &__tile
      pointer-events none
    &__copyright
      font-size .85em
      position absolute
      bottom 5px
      right 5px
</style>
