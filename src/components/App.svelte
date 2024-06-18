<script lang="ts">
  import Card from './Card.svelte';
  import { onMount } from 'svelte';
  import '../styles/global.css';

  interface Coffee {
    id: number,
    uid: string,
    blend_name: string;
    origin: string;
    variety: string;
    notes: string;
    intensifier: string;
  }

  const initialColorPalette = ['#FFB3BA', '#FFDFBA', '#FFFFBA', '#BAFFC9', '#BAE1FF', '#E1BAFF'];
  let colorPalette = [...initialColorPalette];
  let tagColors: { [key: string]: string } = {};

  function getRandomColor(): string {
    const letters = '0123456789ABCDEF';
    let color = '#';
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  function getTagColor(tag: string): string {
    if (!tagColors[tag]) {
      if (colorPalette.length > 0) {
        const randomIndex = Math.floor(Math.random() * colorPalette.length);
        tagColors[tag] = colorPalette[randomIndex];
        colorPalette.splice(randomIndex, 1);
      } else {
        tagColors[tag] = getRandomColor();
      }
    }
    return tagColors[tag];
  }

  let coffees: Coffee[] = [];
  let isLoading = false;
  let isAnimating = false;
  let errorCount = 0;

  let lastActionTime = Date.now();
  const throttleTime = 30000;

  function resetInactivityTimer() {
    lastActionTime = Date.now();
  }

  function checkInactivity() {
    const currentTime = Date.now();
    if (currentTime - lastActionTime > throttleTime) {
      fetchCoffee();
      lastActionTime = Date.now();
    }
    requestAnimationFrame(checkInactivity);
  }

  function addCard() {
    fetchCoffee().then(() => {
      const container = document.querySelector('.container');
      const lastCard = container?.lastElementChild;
      lastCard?.scrollIntoView({ behavior: 'smooth' });
    });
  }

  function addCardDueToInactivity() {
    fetchCoffee().then(() => {
      const container = document.querySelector('.container');
      const lastCard = container?.lastElementChild;
      lastCard?.scrollIntoView({ behavior: 'smooth' });
    });
  }

  onMount(() => {
    fetchCoffee();
    document.addEventListener('mousemove', resetInactivityTimer);
    document.addEventListener('keypress', resetInactivityTimer);
    document.addEventListener('touchstart', resetInactivityTimer);
    document.addEventListener('touchmove', resetInactivityTimer);
    requestAnimationFrame(checkInactivity);
  });

  async function fetchCoffee() {
    if (isLoading) return;
    isLoading = true;
    isAnimating = true;

    try {
      const response = await fetch('https://random-data-api.com/api/coffee/random_coffee');

      if (!response.ok) {
        if (response.status === 429 && errorCount < 3) {
          errorCount++;
          setTimeout(fetchCoffee, 2000);
        } else if (response.status >= 500) {
          console.error(`Server error: ${response.status}`);
          setTimeout(fetchCoffee, 5000);
        } else {
          console.error(`Error: ${response.status}`);
          throw new Error(`Error: ${response.status}`);
        }
      } else {
        const coffee = await response.json();
        coffees = [...coffees, coffee];
        errorCount = 0;
      }
    } catch (error) {
      console.error(error);
    } finally {
      isLoading = false;
      isAnimating = false;
    }
  }
</script>

<style>
  .button-wrapper {
    position: fixed;
    bottom: 20px;
  }

  .add-button {
    width: 56px;
    height: 56px;
    background-color: #FFB13B;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 4px 16px rgba(0 0 0 / 7%);
    cursor: pointer;
    border: none;
    outline: none;
    padding: 0;
    appearance: none;
  }

  .add-button span {
    font-size: 32px;
    color: #FFF;
    line-height: 1;
  }

  .add-button.loading::after {
    content: '';
    position: absolute;
    width: 24px;
    height: 24px;
    border: 3px solid #FFF;
    border-top: 3px solid transparent;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }

  @media (width <= 320px) {
    .button-wrapper {
      bottom: 10px;
    }

    .add-button {
      width: 48px;
      height: 48px;
    }

    .add-button span {
      font-size: 24px;
    }
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    50% {
      transform: rotate(360deg);
    }
    100% {
      transform: rotate(240deg);
    }
  }
</style>

<div class="container">
  {#each coffees as coffee (coffee.id)}
    <Card {coffee} {getTagColor} />
  {/each}
</div>
<div class="button-wrapper">
  <button class="add-button {isAnimating ? 'loading' : ''}" on:click={addCard} disabled={isLoading}>
    <span>+</span>
  </button>
</div>