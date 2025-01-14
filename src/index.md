---
toc: false
---

<div class="hero">
  <h1>Penguins!!!</h1>
  <h2>Making a dashboard with lovely penguins data.</h2>
</div>

```js
import {DuckDBClient} from "npm:@observablehq/duckdb";
const df = FileAttachment("data/penguins.csv").csv({typed: true});
const world = await FileAttachment("data/countries-110m.json").json();
const land = topojson.feature(world, world.objects.land);
const db = DuckDBClient.of({penguins: FileAttachment("data/penguins.csv")});
```

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
const query = `SELECT ${quotedColumns} FROM "penguins" LIMIT 10;`;
display(Inputs.table(await db.query(query)))
```

---

## Map View

Wait! Where are these islands?

```js
const islands = [
  {name: "Torgersen Island", latitude: -64.77274, longitude: -64.0745},
  {name: "Dream Island", latitude: -64.72678, longitude: -64.22484},
  {name: "Biscoe Island", latitude: -65.26, longitude: -65.30}
]

const neighbours = [
  {name: "Antartica", latitude: -68, longitude: -67},
  {name: "Chile", latitude: -55, longitude: -69}
]
```

```js
function penguinMap({width}){
  return Plot.plot({
    width,
    height: 500,
    projection: ({width, height}) => d3.geoAzimuthalEquidistant()
      .rotate([70, 60])
      .translate([width / 2, height / 2])
      .scale(width*2.5)
      .clipAngle(40),
    color: {legend: true},
    marks: [
      Plot.graticule(),
      Plot.geo(land, {fill: "currentColor"}),
      Plot.frame(),
      Plot.dot(islands, {x: "longitude", y: "latitude", fill: "name", r: 3}),
      Plot.text(neighbours, {x: "longitude", y: "latitude", text: (d) => d.name, fontSize: 30, fill: "black", stroke: "white"})
    ]
  })
}
```

${
  resize((width) => penguinMap({width}))
}

And where are the penguins from?

```js
const breakdown = [
  {species: "Adelie Penguin", island: "Torgersen Island", value: df.filter(d => d.Island === "Torgersen" && d.Species === "Adelie Penguin").length},
  {species: "Adelie Penguin", island: "Dream Island", value: df.filter(d => d.Island === "Dream" && d.Species === "Adelie Penguin").length},
  {species: "Adelie Penguin", island: "Biscoe Island", value: df.filter(d => d.Island === "Biscoe" && d.Species === "Adelie Penguin").length},
  {species: "Chinstrap Penguin", island: "Torgersen Island", value: df.filter(d => d.Island === "Torgersen" && d.Species === "Chinstrap Penguin").length},
  {species: "Chinstrap Penguin", island: "Dream Island", value: df.filter(d => d.Island === "Dream" && d.Species === "Chinstrap Penguin").length},
  {species: "Chinstrap Penguin", island: "Biscoe Island", value: df.filter(d => d.Island === "Biscoe" && d.Species === "Chinstrap Penguin").length},
  {species: "Gentoo Penguin", island: "Torgersen Island", value: df.filter(d => d.Island === "Torgersen" && d.Species === "Gentoo Penguin").length},
  {species: "Gentoo Penguin", island: "Dream Island", value: df.filter(d => d.Island === "Dream" && d.Species === "Gentoo Penguin").length},
  {species: "Gentoo Penguin", island: "Biscoe Island", value: df.filter(d => d.Island === "Biscoe" && d.Species === "Gentoo Penguin").length}
]

function marimekko({
  x,
  y,
  z,
  value = z,
  anchor = "middle",
  inset = 0.5,
  ...options
} = {}) {
  const stackX = /\bleft$/i.test(anchor) ? Plot.stackX1 : /\bright$/i.test(anchor) ? Plot.stackX2 : Plot.stackX;
  const stackY = /^top\b/i.test(anchor) ? Plot.stackY2 : /^bottom\b/i.test(anchor) ? Plot.stackY1 : Plot.stackY;
  const [Xv, setXv] = Plot.column(value);
  const {x: X, x1, x2, transform: tx} = stackX({offset: "expand", y, x: Xv});
  const {y: Y, y1, y2, transform: ty} = stackY({offset: "expand", x, y: value});
  return Plot.transform({x: X, x1, x2, y: Y, y1, y2, z, inset, frameAnchor: anchor, ...options}, (data, facets) => {
    const I = d3.range(data.length);
    const X = Plot.valueof(data, x);
    const Z = Plot.valueof(data, value);
    const sum = d3.rollup(I, I => d3.sum(I, i => Z[i]), i => X[i]);
    setXv(I.map(i => sum.get(X[i])));
    tx(data, facets);
    ty(data, facets);
    return {data, facets};
  });
}

function penguinMekko({width}) {
  const xy = (options) => marimekko({...options, x: "species", y: "island", value: "value"});
  return Plot.plot({
    width,
    height: 640,
    label: null,
    x: {percent: true, ticks: 10, tickFormat: (d) => d === 100 ? `100%` : d},
    y: {percent: true, ticks: 10, tickFormat: (d) => d === 100 ? `100%` : d},
    marks: [
      Plot.frame(),
      Plot.rect(breakdown, xy({fill: "island", fillOpacity: 0.5})),
      Plot.text(breakdown, xy({text: d => [d.value.toLocaleString("en"), d.island, d.species].join("\n"), fontSize: d => d.value === 0? 0 : 12})),
      // Plot.text(breakdown, Plot.selectMinX(xy({z: "island", text: "island", anchor: "right", textAnchor: "middle", lineAnchor: "bottom", rotate: 90, dx: 516}))),
      Plot.text(breakdown, Plot.selectMaxY(xy({z: "species", text: "species", anchor: "top", lineAnchor: "bottom", dy: -6})))
    ]
  });
}
```

<div>
${
  resize((width) => penguinMekko({width}))
}
</div>

---

## Exploratory Data Analysis

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

Is a penguin over 4,500 g common? Let's take a look at it.

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
    title: "Mass over 4500 g",
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

It seems like it depends on the species. Gentoo penguins are bigger than the others.

---

Data:

https://github.com/allisonhorst/palmerpenguins/tree/main

https://www.kaggle.com/datasets/parulpandey/palmer-archipelago-antarctica-penguin-data

Gorman KB, Williams TD, Fraser WR (2014). Ecological sexual dimorphism and environmental variability within a community of Antarctic penguins (genus Pygoscelis). PLoS ONE 9(3):e90081. https://doi.org/10.1371/journal.pone.0090081

---

Plot Reference:

https://observablehq.com/@observablehq/plot-gallery



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
  background: linear-gradient(90deg, var(--theme-foreground-focus), currentColor);
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
