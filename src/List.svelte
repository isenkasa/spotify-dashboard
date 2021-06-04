<script>
  import { onMount } from "svelte";
  import { slide } from "svelte/transition";
  import { token, timeRange, tokenExpired } from "./stores.js";

  export let collectionType, collectionMap;

  let collection;

  async function getCollection() {
    const accessToken = $token;

    const url = new URL(`https://api.spotify.com/v1/me/top/${collectionType}?`);
    const params = new URLSearchParams({
      time_range: $timeRange,
      limit: 10,
      offset: 0,
    });

    if (accessToken) {
      const res = await fetch(url + params, {
        headers: {
          Authorization: "Bearer " + accessToken,
        },
      });

      if (res.ok) {
        const data = await res.json();
        console.log(data);
        collection = data.items.map(collectionMap);
      } else {
        tokenExpired.set(true);
      }
    }
  }

  onMount(getCollection);

  function caps(string) {
    return string.charAt(0).toUpperCase() + string.slice(1);
  }

  // Collapsable menu
  let expanded = false;
  function collapse() {
    expanded = !expanded;
  }
</script>

<div class="items-container">
  <button class="expand-btn" on:click={collapse}>
    <span class="expand-btn-name">{caps(collectionType)}</span>
    {#if !expanded}
      <span class="expand-btn-icon">+</span>
    {:else}
      <span class="expand-btn-icon">-</span>
    {/if}
  </button>

  {#if collection && expanded}
    <div class="list-container" transition:slide>
      {#each collection as { name, art, info, link }, i}
        <div class="item-info">
          <div class="item-text">
            <div class="item-details">
              <h2>{i + 1}. {name}</h2>
              <span style="white-space: pre-line">{info}</span>
              <!-- <p>{titleCase(info)}</p> -->
            </div>
          </div>
          <div class="img-container">
            <a class="img-link" href={link}>
              <img class="item-img" src={art} alt={name} />
            </a>
          </div>
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  .expand-btn {
    width: clamp(20rem, 70%, 40rem);
    font-size: 2rem;
    font-weight: 500;
    margin-top: 1em;
    border-style: solid;
  }

  .expand-btn-icon {
    float: right;
    padding-bottom: 0.2rem;
  }

  .list-container {
    margin-left: auto;
    margin-right: auto;
    display: flex;
    width: clamp(20rem, 70%, 40rem);
    flex-direction: column;
  }

  .item-info {
    background: #282e2b;
    padding: 1em;
    margin: 0.25em;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 10em;
    border-style: solid;
    border-radius: 50px;
  }

  .item-text {
    display: flex;
    text-align: left;
    width: 65%;
    height: 100%;
    /* margin: auto; */
  }

  .item-details {
    /* margin: 1.5em; */
    /* margin: auto; */
    padding: 1rem;
  }

  .img-container {
    max-width: 40%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .img-link {
    max-width: 100%;
    max-height: 100%;
  }

  .item-img {
    border-radius: 10px;
    width: 10rem;
    /* max-width: 90%; */
    /* max-height: 90%; */
  }
</style>