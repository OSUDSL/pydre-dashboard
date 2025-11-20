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
view(data);
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

function histPlot(graph, xAxis, yAxis, fill, width) {
  return (Plot.plot({
    width,
  y: {grid: true},
  color: {legend: true},
  marks: [
    Plot.rectY(graph, Plot.binX({y: "count"}, {x: xAxis, fill: fill, channel: channelObj, tip: true})),
    Plot.ruleY([0]),
  ]
}))
}

function dotPlot(graph, xAxis, yAxis, fill, width){
  return (Plot.plot({
    width,
    color: {legend: true},
      marginBottom: 60,
  x: {
    tickRotate: -30,
  },
  marks: [
    Plot.dot(data, {x: xAxis, y: yAxis, stroke: fill, channels: channelObj, tip: true}),
    Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
  ]
}))
}

const graphTypes = [
  "Dot Plot",
  "Histogram",
];

const graphType = view(Inputs.select(graphTypes, {label: "Choose Graph"}));
```

```js
console.log(graphType)

function chooseGraph(data, xCol, yCol, fill, width){
  switch (graphType){
    case "Histogram":
      return histPlot(data, xCol, yCol, fill, width);
      break;
    case "Dot Plot":
      return dotPlot(data, xCol, yCol, fill, width);
      break;
    default:
      return dotPlot(data, xCol, yCol, fill, width);
      break;
  }
  
}

//make standard graphs if xcol exists
function graphs(card, data, xCol, yCol, fill, width){
  switch(card){
    case 1:
      return chooseGraph(data, xCol, yCol, fill, width);
      break;
    case 2:
      if(headers.includes("ParticipantID")){
        return dotPlot(data, "ParticipantID", yCol, fill, width);
      }
      break;
    case 3:
      if(headers.includes("ScenarioName")){
        return dotPlot(data, "ScenarioName", yCol, fill, width);
      }
      break;
    case 4:
      if(headers.includes("ROI")){
        return dotPlot(data, "ROI", yCol, fill, width);
      }
      break;
    default:
      break;
  }
  
}
```

<div>
  <div class="card">
    ${resize((width) => graphs(1, data, xCol, yCol, fill, width) )}
  </div>
</div>

```js
const yColTwo = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillTwo = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));
```

<div>
  <div class="card">
    ${resize((width) => graphs(2, data, xCol, yColTwo, fillTwo, width) )}
  </div>
</div>

```js
const yColThree = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillThree = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));
```

<div>
  <div class="card">
    ${resize((width) => graphs(3, data, xCol, yColThree, fillThree, width) )}
  </div>
</div>

```js
const yColFour = view(Inputs.select(headers, { label: "Y Axis", value: headers[1] }));

const fillFour = view(Inputs.select(headers, { label: "Color Fill", value: headers[1] }));
```

<div>
  <div class="card">
    ${resize((width) => graphs(4, data, xCol, yColFour, fillFour, width) )}
  </div>
</div>

  <div class="card">
    ${Inputs.table(data)}
  </div>

