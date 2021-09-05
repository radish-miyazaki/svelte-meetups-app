<script lang="ts">
  import customMeetupsStore from "./Meetups/meetups-store";
  import Header from "./UI/Header.svelte";
  import MeetupGrid from "./Meetups/MeetupGrid.svelte";
  import EditMeetup from "./Meetups/EditMeetup.svelte";
  import MeetupDetails from "./Meetups/MeetupDetails.svelte";
  import LoadingSpinner from "./UI/LoadingSpinner.svelte";
  import Error from "./UI/Error.svelte";
  let error;

  let editMode: string;
  let page: string = 'overview';
  let editedId: string;
  let pageId: string;
  let isLoading = true;

  fetch(process.env.FIREBASE_REALTIME_DATABASE_URI + '.json')
    .then(res => {
      if (!res.ok) {
        throw new Error("Fetching meetups failed, please try again later!")
      }
      return res.json();
    })
    .then(data => {
      const loadedMeetups = [];
      for (const key in data) {
        loadedMeetups.push({
          ...data[key],
          id: key,
        });
      }
      setTimeout(() => {
        isLoading = false;
        // 逆順にする
        customMeetupsStore.setMeetups(loadedMeetups.reverse());
      }, 1000)
    })
    .catch(err => {
      error = err;
      isLoading = false;
      console.log(err)
    })

  const savedMeetup = () => {
    editMode = null;
    editedId = null;
  }

  const cancelEdit = () => {
    editMode = null;
    editedId = null;
  }

  const showDetails = (event) => {
    page = 'details'
    pageId = event.detail
  }

  const closeDetails = () => {
    page = 'overview'
    pageId = null
  }

  const startEdit = (e) => {
    editMode = 'edit';
    editedId = e.detail;
  }

  const clearError = () => {
    error = null;
  }
</script>

<style>
  main {
    margin-top: 5rem;
  }
</style>

{#if error}
  <Error message={error.message} on:cancel={clearError} />
{/if}

<Header />

<main>
  {#if page === 'overview'}
    {#if editMode === 'edit'}
      <EditMeetup on:save={savedMeetup} on:cancel={cancelEdit} id={editedId} />
    {/if}
    {#if isLoading}
      <LoadingSpinner />
    {:else}
      <MeetupGrid
        meetups={$customMeetupsStore}
        on:add={() => editMode = 'edit'}
        on:showDetails={showDetails}
        on:edit={startEdit}
      />
    {/if}
  {:else if page === 'details'}
    <MeetupDetails id={pageId} on:close={closeDetails} />
  {/if}
</main>
