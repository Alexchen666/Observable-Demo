---
toc: false
sql: 
  penguins: data/penguins.csv
---

<div class="hero">
  <h1>Penguins!!!</h1>
  <h2>Making a dashboard with lovely penguin data.</h2>
</div>

```js
import {sql} from "npm:@observablehq/duckdb";
const df = FileAttachment("data/penguins.csv").csv({typed: true});
```

# All about Penguins

## Table View

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

---

## Exploratory Data Analysis

Now, let's look at some key numbers.

<div class="grid grid-cols-3" style="grid-auto-rows: auto;">
  <div class="card">
    <h2>Adelie Penguin</h2>
      <span class="big">${df.filter((d) => d.Species === "Adelie Penguin").length.toLocaleString("en-US")} </span>
  </div>
  <div class="card">
    <h2>Chinstrap Penguin</h2>
      <span class="big">${df.filter((d) => d.Species === "Chinstrap Penguin").length.toLocaleString("en-US")} </span>
  </div>
  <div class="card">
    <h2>Gentoo Penguin</h2>
      <span class="big">${df.filter((d) => d.Species === "Gentoo Penguin").length.toLocaleString("en-US")} </span>
  </div>
</div>

<div class="grid grid-cols-2" style="grid-auto-rows: auto;">
  <div class="card">${
    resize((width) => penguinChart(df, {width}))
  }</div>
  <div class="card">

  ```js
  const species = [
    {name: "Adelie Penguin", select: "Adelie Penguin"},
    {name: "Chinstrap Penguin", select: "Chinstrap Penguin"},
    {name: "Gentoo Penguin", select: "Gentoo Penguin"},
    {name: "All", select: ["Adelie Penguin", "Chinstrap Penguin", "Gentoo Penguin"]}
  ]
  const speciesSelected = view(Inputs.select(species, 
                                          {label: "Select the species(s) to show:",
                                            format: (t) => t.name,
                                            value: species.find((t) => t.name === "All")
                                          }
                                          ));
  ```

  ```js
  const usedData = speciesSelected["name"] != "All" ? df.filter(d => d.Species === speciesSelected["select"]) : df;
  ```
  
  ${
    resize((width) => penguinHist(usedData, {width}))
  }
  
  </div>
</div>

```js
const over4500 = [
  {species: "Adelie Penguin", total: 146, yes: df.filter(d => d['Body Mass (g)'] >= 4500 && d.Species === "Adelie Penguin").length},
  {species: "Chinstrap Penguin", total: 68, yes: df.filter(d => d['Body Mass (g)'] >= 4500 && d.Species === "Chinstrap Penguin").length},
  {species: "Gentoo Penguin", total: 119, yes: df.filter(d => d['Body Mass (g)'] >= 4500 && d.Species === "Gentoo Penguin").length}
]

function penguinRatio({width}){
  return Plot.plot({
    axis: null,
    label: null,
    height: 260,
    width,
    marginTop: 20,
    marginBottom: 70,
    title: "Mass over 4500 kg",
    subtitle: "Of 3 species of penguins",
    marks: [
      Plot.axisFx({lineWidth: 10, anchor: "bottom", dy: 20}),
      Plot.waffleY(over4500, {fx: "species", y: "total", fillOpacity: 0.4, rx: "100%"}),
      Plot.waffleY(over4500, {fx: "species", y: "yes", rx: "100%", fill: "#99ffff"}),
      Plot.text(over4500, {fx: "species", text: (d) => (d.yes / d.total).toLocaleString("en-US", {style: "percent"}), frameAnchor: "bottom", lineAnchor: "top", dy: 6, fill: "#99ffff", fontSize: 24, fontWeight: "bold"})
    ]
  })
}

```
<div class="card">
  ${
    resize((width) => penguinRatio({width}))
  }
</div>

---

Source:

https://github.com/allisonhorst/palmerpenguins/tree/main

https://www.kaggle.com/datasets/parulpandey/palmer-archipelago-antarctica-penguin-data

Gorman KB, Williams TD, Fraser WR (2014). Ecological sexual dimorphism and environmental variability within a community of Antarctic penguins (genus Pygoscelis). PLoS ONE 9(3):e90081. https://doi.org/10.1371/journal.pone.0090081



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
