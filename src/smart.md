---
theme: dashboard
title: Data By File
toc: false
---

```js
// Load CSV parsing functions
import { csvParse, autoType } from "npm:d3-dsv";

// File upload input (CSV only)
const fileName = view(
  Inputs.file({
    label: "Upload CSV File",
    accept: ".csv",
    required: true
  })
);
```

```js
// Read and parse the uploaded CSV
const text = await fileName.text();
const raw = csvParse(text);

// Get the first column’s header name
const firstCol = raw.columns[0];

// Ensure the first column stays a string
const customAutoType = (d) => {
  const row = autoType(d);
  row[firstCol] = String(d[firstCol]);
  return row;
};

// Apply custom typing to all rows
const data = raw.map(customAutoType);
``` 

```js
// All column names
const headers = Object.keys(data[0]);

const headersToRemove = [];

for (const key of headers) {
  const firstValue = data[0][key];
  let isConstant = true;

  // check if the value is the same in every row
  for (let i = 1; i < data.length; i++) {
    if (data[i][key] !== firstValue) {
      isConstant = false;
      break;
    }
  }

  // if the column is constant, remove it
  if (isConstant) {
    headersToRemove.push(key);
  }
}

// remove constant headers
for (const key of headersToRemove) {
  const idx = headers.indexOf(key);
  if (idx !== -1) headers.splice(idx, 1);
}

// TODO: Remove unused xColumns and yColumns

// Dropdowns for x-axis, y-axis, and fill color
const xCol = view(Inputs.select(headers, { label: "X Axis", value: headers[0] }));

const fill = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

// TODO: All headers as channels
// Channel checkboxes
const channels = view(Inputs.checkbox(headers, {label: "Channels"}));
```

```js
// Convert selected channel names into an object
const channelObj = Object.fromEntries(channels.map(c => [c, c]));

// Main function to draw dot plot
function dotPlot(graph, dodge, xAxis, yAxis, fill, widthMultiplier){
  const height = 500;
  const marginTop = 20;
  const marginBottom = 60;
  const marginLeft = 70;
  const marginRight = 20;

  // Collect Y values and check if numeric
  const yData = []
  let numbers = false;

  for (let i = 0; i < graph.length; i++) {
    yData.push(graph[i][yAxis]);
    if (typeof (graph[i][yAxis]) === "number"){
      numbers = true;
    }
  }

  // Unique X categories for chart width
  const xData = []
  for (let i = 0; i < graph.length; i++) {
    if (!(xData.includes(graph[i][xAxis]))){
    xData.push(graph[i][xAxis]);
    }
  }

  // Auto width adjustment
  let widthData = xData.length * widthMultiplier;
  if(xData.length < 10){
    widthData = 400;
  }

  // Main chart (dodged or normal)
  let chart;
  if ((dodge.length > 0) && ((typeof graph[1][xAxis]) === "string")){
    chart = (Plot.plot({
      width: widthData,
      height,
      marginTop,
      marginBottom,
      marginLeft,
    x: {Domain: xData, nice: true,
      tickRotate: -30},
    marks: [
      Plot.dot(graph, Plot.dodgeX("middle", {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true})),
      Plot.crosshair(graph, {x: xAxis, y: yAxis, color: fill, opacity: 0.5})
    ]
  }))
  } else {
    chart = (Plot.plot({
      width: widthData,
      height,
      marginTop,
      marginBottom,
      marginLeft,
    x: {Domain: xData, nice: true,
      tickRotate: -30},
    marks: [
      Plot.dot(graph, {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true}),
      Plot.crosshair(graph, {x: xAxis, y: yAxis, color: fill, opacity: 0.5})
    ]
  }))
  }
  
  // Add legend and enable scrolling
    let legend = chart.legend("color");
    let title = fileName["name"];

    chart.classList.add("chart");

    const scrollbar = html`<div>`;
    scrollbar.append(title, legend, chart);
    const div = html`<div>`;
    div.append(scrollbar);
    return div;
}

// TODO: Make graphs for every yColumns (Driving Stats)
```



<style>
    .container {
    display: flex;
    align-items: flex-start;
    padding-bottom: 0px;
  }
  .container .scrollbar {
    overflow-x: scroll;
    flex: 1;
  }
  .container .chart {
    max-width: none;
  }
</style>