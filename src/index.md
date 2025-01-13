---
toc: false
sql: 
  penguins: data/penguins.csv
---

<div class="hero">
  <h1>Penguins!!!</h1>
</div>

```js
const df = FileAttachment("data/penguins.csv").csv({typed: true});
```

First, let's look at the first ten rows in our penguins dataset.

```sql id=first10
SELECT * FROM penguins LIMIT 10
```

```js
function penguinChart(data, {width}) {
  return Plot.plot({
    title: "Big Penguins!",
    subtitle: "Are they really big?",
    width,
    grid: true,
    x: {label: "Body mass (g)"},
    y: {label: "Flipper length (mm)"},
    color: {legend: true},
    marks: [
      Plot.linearRegressionY(data, {x: "Body Mass (g)", y: "Flipper Length (mm)", stroke: "Species"}),
      Plot.dot(data, {x: "Body Mass (g)", y: "Flipper Length (mm)", stroke: "Species", tip: true})
    ]
  });
}
```

<div class="grid grid-cols-1" style="grid-auto-rows: auto;">
  <div class="card">${
    resize((width) => penguinChart(df, {width}))
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
