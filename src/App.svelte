<script lang="ts">

  import { scale } from 'svelte/transition';
  import { swipe } from 'svelte-gestures';

  type Position = {
    x: number
    y: number
  }

  type Direction = "UP" | "DOWN" | "LEFT" | "RIGHT";

  const key_directions = {
    // arrow directions
    "ArrowUp": "UP",
    "ArrowDown": "DOWN",
    "ArrowLeft": "LEFT",
    "ArrowRight": "RIGHT",
    // keyboard directions
    "w": "UP",
    "s": "DOWN",
    "a": "LEFT",
    "d": "RIGHT",
    // swipe directions
    "top": "UP",
    "bottom": "DOWN",
    "left": "LEFT",
    "right": "RIGHT",
  }

  const forbidden_direction_changes = {
    "LEFT": "RIGHT",
    "RIGHT": "LEFT",
    "UP": "DOWN",
    "DOWN": "UP",
  }

  const board_size: number = 21;
  const delay: number = 200;

  let snake: Position[] = [
    { x: 4, y: Math.floor(board_size / 2), },
    { x: 3, y: Math.floor(board_size / 2), },
    { x: 2, y: Math.floor(board_size / 2), },
  ];

  let direction: Direction = "RIGHT";
  let food: Position = random_position();

  let start: boolean = false;

  $: if(start) {

    const interval = setInterval(() => {

      const lost = move();

      if(lost) {
        clearInterval(interval);
      }

    }, delay);
  }

  function move() {

    const length: number = snake.length;

    let new_head_position: Position = get_new_head_position(direction, snake[0]);
    let previous_position: Position;

    if(new_head_position.x < 0) return false;
    if(new_head_position.y < 0) return false;
    if(new_head_position.x >= board_size) return false;
    if(new_head_position.y >= board_size) return false;

    for(let i = 0; i < length; i++) {

      const position = snake[i];
      const last: boolean = i === snake.length-1;

      if(last && same_position(food, new_head_position)) {
        eat(position);
      }

      snake[i] = previous_position ?? new_head_position;
      
      if(i && same_position(snake[i], new_head_position)) {
        return true;
      }

      previous_position = position;

    }

    snake = snake;

    return false;

  }

  function get_new_head_position(direction: Direction, { x, y }: Position): Position {

    switch(direction) {
      case "UP": y--; break;
      case "DOWN": y++; break;
      case "LEFT": x--; break;
      case "RIGHT": x++; break;
    }

    return { x, y };

  }

  function change_direction(event: Event) {

    const new_direction = key_directions[event.key];

    if(!new_direction) {
      return;
    }

    if(forbidden_direction_changes[direction] === new_direction) {
      return;
    }

    start = true;

    direction = key_directions[event.key];

  }

  function eat(position: Position) {
    snake.push(position);
    food = random_position();
  }

  function random_position(): Position {
    return {
      x: Math.floor(Math.random() * board_size),
      y: Math.floor(Math.random() * board_size),
    }
  }

  function same_position(position_1: Position, position_2: Position): boolean {
    return position_1.x === position_2.x && position_1.y === position_2.y;
  }

</script>

<svelte:window on:keydown|preventDefault={change_direction} />

<main use:swipe={{ timeframe: 1000, minSwipeDistance: 10, touchAction: 'none', }} on:swipe={change_direction}>

  <div id="board" style={`
    grid-template-columns: repeat(${board_size}, 1fr);
  `}> 

    {#each { length: board_size ** 2 } as _, i}
      <div class="block" style={`
        background-color: var(--${i % 2 ? "white" : "lightgrey"});
      `}> </div>
    {/each}

    {#each snake as { x, y }}
      <div class="block snake"
        style={`
          top: ${y}em;
          left: ${x}em;
      `}> </div>
    {/each}

    {#key food}
      <div class="block food" in:scale
        style={`
          top: ${food.y}em;
          left: ${food.x}em;
      `}> </div>
    {/key}

  <div>

</main>

<style>

  main {
    --block-size: 1.5em;

    display: grid;
    place-items: center;

    height: 100vh;

    background: var(--darkgrey);
  }

  #board {
    position: relative;

    display: grid;
  }

  .block {
    width: 1em;
    height: 1em;

    font-size: var(--block-size);
  }

  .snake {
    position: absolute;

    background-color: var(--color1);

    transition: top 300ms, left 300ms;
  }

  .food {
    position: absolute;

    background-color: var(--color3);
  }

</style>
