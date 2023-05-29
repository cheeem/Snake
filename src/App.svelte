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
  const delay: number = 150;
  const init_snake_size: number = 3;
  const init_snake_x: number = 2;
  const init_snake_y: number = Math.floor(board_size / 2);
  const init_direction: Direction = "RIGHT";
  const top_score_storage_location: string = "snake_top_score";

  let snake: Position[];
  let direction: Direction;
  let food: Position;
  let start: boolean;
  let score: number;
  
  let top_score: number = parseInt(localStorage.getItem(top_score_storage_location)) || 0;

  init();

  $: if(start) {

    const interval = setInterval(() => {

      const alive = move();

      if(alive) {
        return;
      }

      clearInterval(interval);

      init();

    }, delay);

  }

  $: if(score > top_score) {

    top_score = score;

    localStorage.setItem(top_score_storage_location, top_score.toString());

  }

  function init() {
    
    snake = [];

    for(let i = init_snake_size; i; i--) {
      snake.push({
        x: init_snake_x + i,
        y: init_snake_y,
      });
    }

    direction = init_direction;

    food = random_position();

    start = false;

    score = 0;

  }

  function move(): boolean {

    const length: number = snake.length;

    let new_head_position: Position = get_new_head_position(direction, snake[0]);
    let new_food_position: Position = random_position();
    let previous_position: Position;

    const out_of_bounds = new_head_position.x < 0 
      || new_head_position.y < 0 
      || new_head_position.x >= board_size 
      || new_head_position.y >= board_size;
    const consumes_food = same_position(food, new_head_position);

    if(out_of_bounds) {
      return false;
    }

    if(consumes_food && same_position(new_food_position, new_head_position)) {
      new_food_position = random_position();
    }

    for(let i = 0; i < length; i++) {

      const position = snake[i];
      const last: boolean = i === snake.length-1;

      snake[i] = previous_position ?? new_head_position;

      const hits_itself = i && same_position(snake[i], new_head_position);

      if(hits_itself) {
        return false;
      }

      if(consumes_food && same_position(new_food_position, position)) {
        new_food_position = random_position();
      }

      if(last && consumes_food) {
        eat(position, new_food_position);
      }

      previous_position = position;

    }

    snake = snake;

    return true;

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

    const new_direction = key_directions[event.detail.direction ?? event.key];

    if(!new_direction) {
      return;
    }

    if(forbidden_direction_changes[direction] === new_direction) {
      return;
    }

    start = true;

    direction = new_direction;

  }

  function eat(position: Position, new_food_position: Position) {
    snake.push(position);
    score++;
    food = new_food_position;
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

<main use:swipe={{ timeframe: 200, minSwipeDistance: 0, touchAction: 'none', }} on:swipe={change_direction}>

  <div id="board" style={`
    grid-template-columns: repeat(${board_size}, 1fr);
  `}> 

    <div class="score">
      {score}
    </div>

    <div class="score top">
      {top_score}
    </div>

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
    --speed: 200ms;

    display: grid;
    place-items: center;

    font-size: min(1rem, 2.5vw);

    height: 100vh;

    background: var(--darkgrey);
  }

  #board {
    position: relative;

    display: grid;
  }

  .score {
    position: absolute;
    top: calc(var(--block-size) * -1);

    font-weight: 600;
    font-size: var(--block-size);
  }

  .top {
    right: 0;
  }

  .block {
    width: 1em;
    height: 1em;

    font-size: var(--block-size);
  }

  .snake {
    position: absolute;

    background-color: var(--color7);

    transition: top var(--speed), left var(--speed);
  }

  .food {
    position: absolute;

    background-color: var(--color3);
  }

</style>
