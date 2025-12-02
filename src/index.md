---
theme: dashboard
title: Dashboard
toc: false
---

# Pydre Data Dashboard

<div class="tip">

This is a dashboard designed to help perform quick visualizations of the data from R2Drv project.
Please select a CSV file from the 

</div>



```js
//choose input file
import { csvParse, autoType } from "npm:d3-dsv";

const fileName = view(
  Inputs.file({
    label: "Select CSV File",
    accept: ".csv",
    required: true
  })
);
```

```js
const text = await fileName.text();
const raw = csvParse(text);

// Get the first column’s header name
const firstCol = raw.columns[0];

//First column → String
//Everything else → d3.autoType
const customAutoType = (d) => {
  const row = autoType(d);
  row[firstCol] = String(d[firstCol]);
  return row;
};

const example_data = raw.map(customAutoType);
``` 

```js
const headers = Object.keys(example_data[0]);

const xCol = view(Inputs.select(headers, { label: "X Axis", value: headers[0] }));

const yCol = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fill = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

const channels = view(Inputs.checkbox(headers, {label: "Channels"}));
```

```js
const channelObj = Object.fromEntries(channels.map(c => [c, c]));

function plot(data, xAxis, yAxis, fill, width) {
  
    const p = Plot.plot({
            width: 2000,
    color: {legend: true},
    x: {
        tickSpacing: 100,    
        tickRotate: -30,
    },
  marks: [
    Plot.dot(data, {x: xAxis, y: yAxis, color: fill, channels: channelObj, tip: true}),
    Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
  ]
})
    p.classList.add("chart");
    return p;
}
```

```js

function makeplot(data, xAxis, yAxis, fill) {
  const height = 500;
  const marginTop = 20;
  const marginBottom = 60;
  const marginLeft = 70;
  const marginRight = 20;

  const yData = []
  let numbers = false;

  for (let i = 0; i < data.length; i++) {
    yData.push(data[i][yAxis]);
    if (typeof (data[i][yAxis]) === "number"){
      numbers = true;
    }
  }

  let yScale;
  if (numbers){
   yScale = Plot.scale({y: {domain: d3.extent(yData), label: yAxis}});
  } else {
    yScale = Plot.scale({y: {domain: [yData], label: yAxis}});
  }

const xData = []
  for (let i = 0; i < data.length; i++) {
    if (!(xData.includes(data[i][xAxis]))){
    xData.push(data[i][xAxis]);
    }
  }

  let widthData = xData.length * 25;

  if(xData.length < 10){
    widthData = 400;
  }


    const yAxis_plot = Plot.plot({
        width: 40,
        height,
        marginTop,
        marginBottom,
        y: yScale
    });

    const chart = Plot.plot({
        width: widthData,
        height,
        marginTop,
        marginBottom,
        marginLeft: 10,
        x: {nice: true,
            tickRotate: -30},
        y: yScale,
        marks: [
            Plot.dot(data, {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true}),
            Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
            
        ]
    });
    chart.classList.add("chart");

    const scrollbar = html`<div class="scrollbar">`;
    scrollbar.append(chart);

    const div = html`<div class="container">`;
    div.append(yAxis_plot, scrollbar);
    return div;
}
```


<div>

<div class="container">
<div class="scrollbar">
  ${makeplot(example_data, xCol, yCol, fill) }
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
