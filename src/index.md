---
toc: false
sql: 
  penguins: data/penguins.csv
---

<div class="hero">
  <h1>Penguins!!!</h1>
</div>

```js
import {sql} from "npm:@observablehq/duckdb";
const df = FileAttachment("data/penguins.csv").csv({typed: true});
```

First, let's look at the first ten rows in our penguins dataset.

```js
const cols = Object.keys(df[0]);
const usedCols = cols.slice(1-cols.length); // remove the first column "ID"
```


```js
const colsSelected = view(Inputs.checkbox(usedCols, 
                                          {label: "Select the column(s) to show:",
                                            value: usedCols
                                          }
                                          ));
```

```js
const completeCols = ["ID", ...colsSelected]; // make sure "ID" is always selected
const quotedColumns = completeCols.map(col => `"${col}"`).join(', ');
const query = `SELECT ${quotedColumns} FROM penguins LIMIT 10;`;
const queryTable = Inputs.table(await sql([query]));
display(queryTable)
```

```js
function penguinChart(data, {width}) {
  return Plot.plot({
    title: "Big Penguins!",
    subtitle: "Are they really big?",
    width,
    grid: true,
    x: {label: "Body Mass (g)"},
    y: {label: "Flipper Length (mm)"},
    color: {legend: true},
    marks: [
      Plot.linearRegressionY(data, {x: "Body Mass (g)", y: "Flipper Length (mm)", stroke: "Species"}),
      Plot.dot(data, {x: "Body Mass (g)", y: "Flipper Length (mm)", stroke: "Species", tip: true})
    ]
  });
}

function penguinHist(data, {width}){
  return Plot.plot({
    title: "Distribution of Penguin Body Mass",
    subtitle: "Showing both histogram and density curve",
    width,
    grid: true,
    x: {
      label: "Body Mass (g)",
      nice: true
    },
    y: {
      label: "Count"
    },
    marks: [
      Plot.rectY(data, 
        Plot.binX(
          {y: "count"}, 
          {x: "Body Mass (g)", 
          fill: "steelblue",
          fillOpacity: 0.5,
          tip: true}
        )
      ),
      Plot.lineY(data,
        Plot.binX(
          { y: "count" },
          {
            x: "Body Mass (g)",
            thresholds: 10,
            curve: "natural"
          }
        )
    )
    ]
  });
}
```

<div class="grid grid-cols-2" style="grid-auto-rows: auto;">
  <div class="card">${
    resize((width) => penguinChart(df, {width}))
  }</div>
  <div class="card">${
    resize((width) => penguinHist(df, {width}))
  }</div>
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
