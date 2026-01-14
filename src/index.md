---
theme: dashboard
title: Dashboard
toc: false
---

# Pydre Data Dashboard

<div class="tip">

This is a dashboard designed to help perform quick visualizations of the data from R2Drv project.
Please select a CSV file from the data folder

</div>



```js
// Load CSV parsing functions
import { csvParse, autoType } from "npm:d3-dsv";

// File upload input (CSV only)
const fileName = view(
  Inputs.file({
    label: "Select CSV File",
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
const example_data = raw.map(customAutoType);
``` 

```js
// All column names
const headers = Object.keys(example_data[0]);

const headersToRemove = [];

for (const key of headers) {
  const firstValue = example_data[0][key];
  let isConstant = true;

  // check if the value is the same in every row
  for (let i = 1; i < example_data.length; i++) {
    if (example_data[i][key] !== firstValue) {
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

// Dropdowns for x-axis, y-axis, and fill color
const xCol = view(Inputs.select(headers, { label: "X Axis", value: headers[0] }));

const yCol = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fill = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

// Channel and dodge checkboxes
const channels = view(Inputs.checkbox(headers, {label: "Channels"}));

const dodge = view(Inputs.checkbox(["Dodge"], {label: "Dodge"}));

const width_Multiplier = view(Inputs.range([1, 100], {step: 1}));
```

```js
// Convert selected channel names into an object
const channelObj = Object.fromEntries(channels.map(c => [c, c]));

// Main function to draw the plot
function makeplot(data, xAxis, yAxis, fill, dodge) {
  const height = 500;
  const marginTop = 20;
  const marginBottom = 60;
  const marginLeft = 70;
  const marginRight = 50;

  // Collect Y values and check if numeric
  const yData = []
  let numbers = false;

  for (let i = 0; i < data.length; i++) {
    yData.push(data[i][yAxis]);
    if (typeof (data[i][yAxis]) === "number"){
      numbers = true;
    }
  }

  // Choose numeric or categorical Y scale
  let yScale;
  if (numbers){
   yScale = Plot.scale({y: {domain: d3.extent(yData), label: yAxis}});
  } else {
    yScale = Plot.scale({y: {domain: [yData], label: yAxis}});
  }

  // Unique X categories for chart width
  const xData = []
  for (let i = 0; i < data.length; i++) {
    if (!(xData.includes(data[i][xAxis]))){
    xData.push(data[i][xAxis]);
    }
  }

  // Auto width adjustment
  let widthData = xData.length * width_Multiplier;
  if(xData.length < 10){
    widthData = 400;
  }

  // Y-axis plot
    const yAxis_plot = Plot.plot({
        width: 0,
        height,
        marginTop,
        marginBottom,
        y: yScale
    });

  // Main chart (dodged or normal)
let chart;
     if ((dodge.length > 0) && ((typeof data[1][xAxis]) === "string")){
     chart = Plot.plot({
        width: widthData,
        height,
        marginTop,
        marginBottom,
        marginLeft,
        x: {Domain: xData, nice: true,
            tickRotate: -30},
        y: yScale,
        marks: [
            Plot.dot(data, Plot.dodgeX("middle", {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true})),
            Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
            
        ]
    });
  } else {
     chart = Plot.plot({
        width: widthData,
        height,
        marginTop,
        marginBottom,
        marginLeft,
        x: {Domain: xData, nice: true,
            tickRotate: -30},
        y: yScale,
        marks: [
            Plot.dot(data, {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true}),
            Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
            
        ]
    });
  }

  // Add legend and enable scrolling
  let legend = chart.legend("color");

  chart.classList.add("chart");

  const scrollbar = html`<div class="scrollbar">`;
  scrollbar.append(legend, chart);
  const div = html`<div class="container">`;
  div.append(yAxis_plot, scrollbar);
  return div;
}
```


<div>

<div class="container">
<div class="scrollbar">
  ${makeplot(example_data, xCol, yCol, fill, dodge) }
</div>
</div>

  <div class="card">
    ${Inputs.table(example_data)}
  </div>

</div>

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

  .container {
    display: flex;
    align-items: flex-start;
    padding-bottom: 30px;
  }
  .container .scrollbar {
    overflow-x: scroll;
    flex: 1;
  }
  .container .chart {
    max-width: none;
  }

</style>
