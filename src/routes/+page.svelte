<script>
  const GRID_SIZE = 30;
  function initRows(isRandom) {
    let grid = [];
    for (let i = 0; i < GRID_SIZE; i += 1) {
      let q = [];
      for (let j = 0; j < GRID_SIZE; j += 1) {
        q.push({ value: Number(isRandom) * Math.floor(Math.random() * 2) });
      }
      grid.push(q);
    }
    return grid;
  }

  let rows = initRows(1);

  let source = { r: 0, c: 0 };
  let target = { r: GRID_SIZE - 1, c: GRID_SIZE - 1 };

  $: {
    rows[source.r][source.c].value = 3;
    rows[target.r][target.c].value = 4;
  }

  function getColor(v) {
    switch (v) {
      case 0:
        return "white";
      case 1:
        return "black";
      case 3:
        return "red";
      case 4:
        return "green";
      case 5:
        return "rgba(255, 0, 0, 0.4)";
      case 6:
        return "rgba(0, 255, 0, 0.4)";
      case 7:
        return "cyan";
      case 8:
        return "blue";
      case 9:
        return "yellow";
    }
  }

  function handleMouseDown(i, j) {
    if (!resetGrid && !playing) {
      if (selectingSource) {
        if (rows[i][j].value === 5) {
          rows[source.r][source.c].value = 0;
          source = { r: i, c: j };
        }

        selectingSource = false;
      } else if (selectingTarget) {
        if (rows[i][j].value === 6) {
          rows[target.r][target.c].value = 0;
          target = { r: i, c: j };
        }

        selectingTarget = false;
      } else {
        if (rows[i][j].value === 0) {
          rows[i][j].value = 1;
        } else if (rows[i][j].value === 1) {
          rows[i][j].value = 0;
        }
      }
    }
  }
  function handleMouseEnter(i, j) {
    if (rows[i][j].value === 0) {
      if (selectingSource) {
        rows[i][j].value = 5;
      } else if (selectingTarget) {
        rows[i][j].value = 6;
      }
    }
  }

  function handleMouseLeave(i, j) {
    if (rows[i][j].value === 5 || rows[i][j].value === 6) {
      rows[i][j].value = 0;
    }
  }

  function getNeighbors(node) {
    let n = [];
    const { r, c } = node;
    if (
      0 <= r - 1 &&
      r - 1 < GRID_SIZE &&
      (rows[r - 1][c].value === 0 || rows[r - 1][c].value === 4)
    ) {
      n.push({ r: r - 1, c });
    }
    if (
      0 <= r + 1 &&
      r + 1 < GRID_SIZE &&
      (rows[r + 1][c].value === 0 || rows[r + 1][c].value === 4)
    ) {
      n.push({ r: r + 1, c });
    }
    if (
      0 <= c - 1 &&
      c - 1 < GRID_SIZE &&
      (rows[r][c - 1].value === 0 || rows[r][c - 1].value === 4)
    ) {
      n.push({ r, c: c - 1 });
    }
    if (
      0 <= c + 1 &&
      c + 1 < GRID_SIZE &&
      (rows[r][c + 1].value === 0 || rows[r][c + 1].value === 4)
    ) {
      n.push({ r, c: c + 1 });
    }
    return n;
  }

  let selectingSource = false;
  let selectingTarget = false;
  let playing = false;

  function play() {
    let queue = [{ ...source }];
    let cnt = 1;
    let found = false;
    let searching = setInterval(() => {
      if (cnt !== 0 && !found) {
        const node = queue.shift();
        cnt -= 1;
        const neighbors = getNeighbors(node);
        for (let n of neighbors) {
          if (rows[n.r][n.c].value === 0) {
            rows[n.r][n.c].value = 7;
          } else {
            found = true;
          }
          rows[n.r][n.c].parent = { ...node };
          queue.push(n);
          cnt += 1;
        }
        if (rows[node.r][node.c].value !== 3) {
          rows[node.r][node.c].value = 8;
        }
      } else {
        clearInterval(searching);
        if (found) {
          let z = rows[target.r][target.c].parent;
          let pathfinding = setInterval(() => {
            if (z.r !== source.r || z.c !== source.c) {
              rows[z.r][z.c].value = 9;
              z = rows[z.r][z.c].parent;
            } else {
              clearInterval(pathfinding);
              playing = false;
              resetGrid = true;
            }
          }, 50);
        } else {
          playing = false;
          resetGrid = true;
        }
      }
    }, 30);
  }
  let resetGrid = false;

  function handleReset(isRandom) {
    rows = initRows(isRandom);
    source = { r: 0, c: 0 };
    target = { r: GRID_SIZE - 1, c: GRID_SIZE - 1 };
    resetGrid = false;
  }
</script>

<main>
  <div style="margin-bottom: 30px;">
    <h1>Graph Visualizer</h1>
  </div>

  <div class="btn-panel" style="width: {GRID_SIZE * 24}px">
    {#if !resetGrid}
      <button
        class="btn-source"
        on:click={() => {
          selectingSource = true;
        }}
        disabled={selectingTarget || playing}>Select Source</button
      >
      <button
        style="width: 200px;"
        on:click={() => {
          playing = true;
          play();
        }}
        disabled={selectingSource || selectingTarget || playing}>Play</button
      >
      <button
        class="btn-target"
        on:click={() => {
          selectingTarget = true;
        }}
        disabled={selectingSource || playing}>Select Target</button
      >
    {:else}
      <button style="width: 200px;" on:click={() => handleReset(false)}>Reset Blank</button>
      <button style="width: 200px;" on:click={() => handleReset(true)}>Reset Random</button>
    {/if}
  </div>

  <div class="grid" style="width: {GRID_SIZE * 24}px">
    {#each rows as row, i}
      {#each row as cell, j}
        <div
          class="gridCell"
          on:mousedown={() => handleMouseDown(i, j)}
          on:mouseenter={() => handleMouseEnter(i, j)}
          on:mouseleave={() => handleMouseLeave(i, j)}
          style="background-color: {getColor(cell.value)}"
        />
      {/each}
    {/each}
  </div>
</main>

<style>
  main {
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  .btn-panel {
    display: flex;
    justify-content: space-evenly;
  }

  button {
    font-weight: bold;
    padding: 5px 40px;
    /* margin: 0px 10px; */
  }
  .btn-source {
    color: red;
  }
  .btn-target {
    color: green;
  }
  .gridCell {
    border: 1px solid black;
    height: 20px;
    width: 20px;
    margin: 1px;
  }
  .grid {
    display: flex;
    flex-wrap: wrap;
    margin-top: 20px;
  }
</style>
