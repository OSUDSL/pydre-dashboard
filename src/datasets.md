---
theme: dashboard
title: Data Collected
toc: false
---

```js
//choose input file
import { csvParse, autoType } from "npm:d3-dsv";

const fileName = view(
  Inputs.file({
    label: "Upload CSV File",
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

const data = raw.map(customAutoType);
``` 

```js
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

const xCol = view(Inputs.select(headers, { label: "X Axis", value: headers[0] }));

const yCol = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fill = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

const channels = view(Inputs.checkbox(headers, {label: "Channels"}));
```

```js
const channelObj = Object.fromEntries(channels.map(c => [c, c]));

function histPlot(graph, xAxis, yAxis, fill) {
  return (Plot.plot({
    width: 600,
  y: {grid: true},
  color: {legend: true},
  marks: [
    Plot.rectY(graph, Plot.binX({y: "count"}, {x: xAxis, fill: fill, channel: channelObj, tip: true})),
    Plot.ruleY([0]),
  ]
}))
}

function dotPlot(graph, dodge, xAxis, yAxis, fill){
  const height = 500;
  const marginTop = 20;
  const marginBottom = 60;
  const marginLeft = 70;
  const marginRight = 20;

  const yData = []
  let numbers = false;
  let xType;

  for (let i = 0; i < graph.length; i++) {
    yData.push(graph[i][yAxis]);
    if (typeof (graph[i][yAxis]) === "number"){
      numbers = true;
    }
  }

  let yScale;
  if (numbers){
   yScale = Plot.scale({y: {domain: d3.extent(yData), label: yAxis}});
   xType = "Linear";
  } else {
    yScale = Plot.scale({y: {domain: [yData], label: yAxis}});
    xType = "Point";
  }

const xData = []
  for (let i = 0; i < graph.length; i++) {
    if (!(xData.includes(graph[i][xAxis]))){
    xData.push(graph[i][xAxis]);
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

  let chart;

  if ((dodge.length > 0) && ((typeof graph[1][xAxis]) === "string")){
    chart = (Plot.plot({
      width: widthData,
      height,
      marginTop,
      marginBottom,
      marginLeft: 10,
    x: {nice: true,
      tickRotate: -30, type: xType},
    y: yScale,
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
      marginLeft: 10,
    x: {nice: true,
      tickRotate: -30, type: xType},
    y: yScale,
    marks: [
      Plot.dot(graph, {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true}),
      Plot.crosshair(graph, {x: xAxis, y: yAxis, color: fill, opacity: 0.5})
    ]
  }))
  }

  let legend = chart.legend("color");

    chart.classList.add("chart");

    const scrollbar = html`<div class="scrollbar">`;
    scrollbar.append(legend, chart);
    const div = html`<div class="container">`;
    div.append(yAxis_plot, scrollbar);
    return div;
}

const graphTypes = [
  "Dot Plot",
  "Histogram",
];

const graphType = view(Inputs.select(graphTypes, {label: "Choose Graph"}));
```

```js
console.log(graphType)

function chooseGraph(data, dodge, xCol, yCol, fill){
  switch (graphType){
    case "Histogram":
      return histPlot(data, xCol, yCol, fill);
      break;
    case "Dot Plot":
      return dotPlot(data, dodge, xCol, yCol, fill);
      break;
    default:
      return dotPlot(data, dodge, xCol, yCol, fill);
      break;
  }
  
}
```

```js
const dodgeOne = view(Inputs.checkbox(["Dodge"], {label: "Dodge"}));
```

<div>
  <div class="card">
  <div class="container">
  <div class="scrollbar">
    ${chooseGraph(data, dodgeOne, xCol, yCol, fill)}
  </div>
  </div>
  </div>
</div>

```js
const yColTwo = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillTwo = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

const dodgeTwo = view(Inputs.checkbox(["Dodge"], {label: "Dodge"}));
```

<div>
  <div class="card">
  <div class="container">
  <div class="scrollbar">
    ${dotPlot(data, dodgeTwo, "ParticipantID", yColTwo, fillTwo )}
  </div>
  </div>  
  </div>
</div>

```js
const yColThree = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillThree = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

const dodgeThree = view(Inputs.checkbox(["Dodge"], {label: "Dodge"}));
```

<div>
  <div class="card">
  <div class="container">
  <div class="scrollbar">
    ${dotPlot( data, dodgeThree, "ScenarioName", yColThree, fillThree )}
  </div>
  </div>
  </div>
</div>

```js
const yColFour = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillFour = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));

const dodgeFour = view(Inputs.checkbox(["Dodge"], {label: "Dodge"}));
```

<div>
  <div class="card">
  <div class="container">
  <div class="scrollbar">
    ${dotPlot(data, dodgeFour, "ROI", yColFour, fillFour)}
  </div>
  </div>
  </div>
</div>

  <div class="card">
    ${Inputs.table(data)}
  </div>

<style>
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