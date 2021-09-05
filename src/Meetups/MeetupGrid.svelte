<script lang="ts">
  import MeetupItem from "./MeetupItem.svelte";
  import MeetupFilter from "./MeetupFilter.svelte";
  import customMeetupsStore from "./meetups-store";
  import {createEventDispatcher} from "svelte";
  import Button from "../UI/Button.svelte";
  import {flip} from "svelte/animate";
  import {scale} from "svelte/transition";

  export let meetups;
  let favsOnly = false;

  $: filteredMeetups = favsOnly ? meetups.filter(m => m.isFavorite) : meetups;

  const setFilter = (e) => {
    favsOnly = e.detail === 1;
  }

  const dispatch = createEventDispatcher();
</script>

<style>
  section {
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
    margin-top: 1rem;
  }

  #meetup-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  #no-meetups {
    padding: 1rem;
  }

  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>

<section id="meetup-controls">
  <MeetupFilter on:select={setFilter} />
  <Button on:click={() => dispatch('add')}>New Meetup</Button>
</section>
{#if filteredMeetups.length === 0}
  <p id="no-meetups">No meetups found, you can start adding some.</p>
{/if}
<section id="meetups">
  {#each filteredMeetups as meetup (meetup.id)}
    <div transition:scale animate:flip={{duration: 300}}>
      <MeetupItem
        id={meetup.id}
        title={meetup.title}
        subtitle={meetup.subtitle}
        description={meetup.description}
        imageUrl={meetup.imageUrl}
        email={meetup.contactEmail}
        address={meetup.address}
        isFavorite={meetup.isFavorite}
        on:showDetails
        on:edit
      />
    </div>
  {/each}
</section>