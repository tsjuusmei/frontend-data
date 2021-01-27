<template>
  <body>
    <section>
      <h1>Where can you park best with your electric car?</h1>
      <p>
        On the side you can see a scatterplot with the available parking garages
        with availability for electric vehicles
      </p>
      <p>Hover over the points in the graph for more information</p>
      <p>Select your wished city with the buttons or the dropdown below</p>
      <h2 class="margintop">Select your city</h2>
      <select v-model="selectedCity" name="city" id="city">
        <option value="All">All</option>
        <option
          style="text-transform: capitalize"
          v-for="city in cities"
          :key="city"
          :value="city"
        >
          {{ city }}
        </option>
      </select>
      <h2 class="margintop">Popular cities</h2>
      <button @click="selectedCity = 'All'">All</button>
      <button @click="selectedCity = 'Amsterdam'">Amsterdam</button>
      <button @click="selectedCity = 'Almere'">Almere</button>
      <button @click="selectedCity = 'Rotterdam'">Rotterdam</button>
    </section>
    <section>
      <svg :height="height" :width="width">
        <g class="plot__axes">
          <g
            class="plot__axes__x"
            :transform="`translate(30, ${height - 20})`"
          ></g>
          <text class="plot__axes__x__label"></text>
          <g
            class="plot__axes__y"
            :transform="`translate(30, ${-20 + yMargin})`"
          ></g>
          <text
            class="plot__axes__y__label"
            :transform="`translate(${-20 + yMargin}, 30)`"
          ></text>
        </g>
        <g :transform="`translate(30, ${yMargin})`">
          <circle
            v-for="item in modifiedData"
            :key="item.areaid"
            r="5"
            :cx="xScale(item.tariffs.tue)"
            :cy="yScale(item.chargingpointcapacity)"
            :fill="pickColor(item)"
            :areaid="item.areaid"
            @mouseover="createTooltip(), mouseOver()"
            @mouseout="mouseOut()"
          ></circle>
          <div></div>
        </g>
      </svg>
    </section>
  </body>
</template>

<style>
svg {
  width: 620px;
  height: 650px;
}

body {
  display: flex;
}

section {
  height: 100vh;
  width: 45vw;
}

p {
  max-width: 60%;
  margin: 1em auto;
}

.margintop {
  margin-top: 3em;
}

button {
  padding: 1em 2em;
  margin: 1em;
  font-family: "Segoe UI";
  font-weight: 500;
  background-color: lightcyan;
  border-radius: 10px;
  border-color: aliceblue;
}

select {
  padding: 1em;
  font-family: "Segoe UI";
}

div.tooltip {
  position: absolute;
  text-align: left;
  padding: 10px;
  line-height: 4em;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
}
</style>

<script>
import { scaleLinear, select, axisBottom, axisLeft } from "d3";

export default {
  name: "Chartv2",
  props: ["data"],
  data() {
    return {
      height: 600,
      width: 600,
      selectedCity: "All",
      allCities: [],
      yMargin: 30,
    };
  },
  computed: {
    yScale() {
      return scaleLinear()
        .range([this.height - this.yMargin, 0])
        .domain([
          0,
          Math.max(...this.modifiedData.map((d) => d.chargingpointcapacity)),
        ]);
    },
    xScale() {
      return scaleLinear()
        .range([20, this.width - this.yMargin])
        .domain([0, Math.max(...this.data.map((d) => d.tariffs.tue))]);
    },
    modifiedData() {
      if (this.selectedCity.toLowerCase() === "all") return this.data;
      return this.data.filter((d) => {
        return d.city == this.selectedCity;
      });
    },
    cities() {
      return new Set(this.data.map((d) => d.city).sort());
    },
    leftAxis() {
      return axisLeft(this.yScale);
    },
  },
  methods: {
    renderAxes() {
      select(".plot__axes__y")
        .attr("transform", "translate(50, 30)")
        .call(axisLeft(this.yScale));
      select(".plot__axes__x")
        .attr("transform", "translate(30 ,600)")
        .call(axisBottom(this.xScale));
    },
    renderLabels() {
      select(".plot__axes__x__label")
        .attr(
          "transform",
          "translate(" + this.width / 2 + " ," + this.height + ")"
        )
        .attr("y", 40)
        .attr("x", 30)
        .style("text-anchor", "middle")
        .style("font-family", "sans-serif")
        .style("font-size", "12px")
        .text("€ per day");
      select(".plot__axes__y__label")
        .attr("y", 20)
        .attr("x", -290)
        .style("text-anchor", "middle")
        .style("font-family", "sans-serif")
        .style("font-size", "12px")
        .style("transform", "rotate(270deg)")
        .text("# of charging points");
    },
    pickColor(d) {
      if (d.tariffs.tue / d.chargingpointcapacity + 5 > 20) {
        return "red";
      } else {
        return "green";
      }
    },
    mouseOver(d) {
      const div = select("div")
        .append("div")
        .attr("class", "tooltip")
        .style("opacity", 0);

      div.transition().duration(200).style("opacity", 0.9);

      div
        .html(
          "<b>" +
            d.city +
            "</b> <br/>" +
            "<b># of charging points: </b>" +
            d.chargingpointcapacity +
            "<br/>" +
            "<b>€ per day: </b>" +
            Math.floor(d.tariffs.tue)
        )
        .style("left", event.pageX + "px")
        .style("top", event.pageY - 28 + "px");

      return div;
    },
    mouseOut() {
      // console.log("mouseout");
      // div.transition().duration(500).style("opacity", 0);
    },
  },
  watch: {
    data(val) {
      if (val.length === 0) return;
      this.renderAxes();
      this.renderLabels();
    },
    selectedCity() {
      if (this.data.length === 0) return;
      this.renderAxes();
      this.renderLabels();
    },
  },
};
</script>