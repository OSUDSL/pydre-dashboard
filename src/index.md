---
theme: dashboard
title: Dashboard
toc: false
---

<div>
  <h1>Data Dashboard</h1>
</div>

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

const example_data = raw.map(customAutoType);
view(example_data);
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
  return (Plot.plot({
    width,
    color: {legend: true},
    x: {
    tickRotate: -30,
    },
  marks: [
    Plot.auto(data, {x: xAxis, y: yAxis, color: fill, channels: channelObj, tip: true}),
    Plot.crosshair(data, {x: xAxis, y: yAxis,color: fill, opacity: 0.5})
  ]
}))
}
```


<div>

  <div class="card">
  ${resize((width) => plot(example_data, xCol, yCol, fill, width) )} 
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

</style>
