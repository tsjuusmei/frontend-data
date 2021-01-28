<template>
  <button @click="Amsterdam">Amsterdam</button>
  <button @click="Almere">Almere</button>
  <button @click="Rotterdam">Rotterdam</button>

  <div></div>

  <button @click="$emit('updatechart')">update!</button>
</template>

<style>
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
import cleanData from "../assets/index";
import * as d3 from "d3";
const margin = { top: 20, right: 20, bottom: 30, left: 50 };
const width = 960 - margin.left - margin.right;
const height = 500 - margin.top - margin.bottom;
const x = d3.scaleLinear().range([0, width]).nice();
const y = d3.scaleLinear().range([height, 0]).nice();
async function getData() {
  let data = await cleanData();
  return data;
}

getData().then((data) => {
  let cities = [];
  for (let i = 0; i < data.length; i++) {
    // console.log(data[i].city);
    cities.push(data[i].city);
  }
  console.log(cities.sort());

  let almereGarages = [];
  for (let i = 0; i < data.length; i++) {
    if (data[i].city == "Almere") {
      almereGarages.push(data[i]);
    }
  }

  const div = d3
    .select("div")
    .append("div")
    .attr("class", "tooltip")
    .style("opacity", 0);

  const svg = d3
    .select("body")
    .append("h1")
    .attr("x", width / 2)
    .attr("y", 0 - margin.top / 2)
    .attr("text-anchor", "middle")
    .style("font-size", "24px")
    .style("text-align", "center")
    .style("font-family", "sans-serif")
    .text("Value vs Date Graph")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", "translate(170, 20)");

  // Scale the range of the data
  y.domain([
    0,
    d3.max(data, (d) => {
      return parseInt(d.chargingpointcapacity, 10) + 2;
    }),
  ]);
  x.domain([
    0,
    d3.max(data, (d) => {
      return d.tariffs.tue;
    }),
  ]);

  // Add the scatterplot
  svg
    .selectAll("dot")
    .data(
      data.filter((d) => {
        return d.city == "Almere";
      })
    )
    .enter()
    .append("circle")
    .style("fill", (d) => {
      if (d.chargingpointcapacity > 10) {
        return "green";
      } else {
        return "red";
      }
    })
    .attr("r", 5)
    .attr("cy", (d) => {
      return y(d.chargingpointcapacity);
    })
    .attr("cx", (d) => {
      return x(d.tariffs.tue);
    })
    .attr("areaid", (d) => {
      return d.areaid;
    })
    .on("mouseover", function (event, d) {
      div.transition().duration(200).style("opacity", 0.9);
      div
        .html(
          "<b># of charging points: </b>" +
            d.chargingpointcapacity +
            "<br/>" +
            "<b>€ per day: </b>" +
            d.tariffs.tue
        )
        .style("left", event.pageX + "px")
        .style("top", event.pageY - 28 + "px");
    })
    .on("mouseout", function () {
      div.transition().duration(500).style("opacity", 0);
    });

  // Add the X Axis
  svg
    .append("g")
    .attr("transform", "translate(0," + height + ")")
    .call(d3.axisBottom(x));

  // Text label for the x axis
  svg
    .append("text")
    .attr(
      "transform",
      "translate(" + width / 2 + " ," + (height + margin.top + 20) + ")"
    )
    .style("text-anchor", "middle")
    .style("font-family", "sans-serif")
    .style("font-size", "12px")
    .text("€ per day");

  // Add the Y Axis
  svg.append("g").call(d3.axisLeft(y));

  // Text label for the y axis
  svg
    .append("text")
    .attr("y", 200)
    .attr("x", -100)
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .style("font-family", "sans-serif")
    .style("font-size", "12px")
    .text("# of charging points");
});

export default {
  name: "Scatterplot",
  props: {
    data: Array,
    cities: Array,
  },
  data() {
    return getData()
  },
  mounted() {
    console.log("App loaded");
  },
};
</script>