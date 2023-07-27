<script lang="ts">
  import {onMount} from "svelte"
  import Anchor from "src/partials/Anchor.svelte"
  import Heading from "src/partials/Heading.svelte"
  import Content from "src/partials/Content.svelte"
  import GroupCard from "../shared/GroupCard.svelte"
    import Spinner from "src/partials/Spinner.svelte"

  export let savefgroups

  let groups = []
  let fgroups = []
  let loading = true

  onMount(async () => {
    // Get groupnames
    const gres = await fetch("https://manyworlds.network:3306/groups", {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
      },
    })
    let gdata = await gres.json()
    if (gdata.message == "Success") {
      groups = gdata.data
    }
    loading = false
  })

  const followGroup = async group => {
    if (fgroups.findIndex(g => g == group) >= 0) return
    fgroups.push(group)
  }

  const unfollowGroup = async group => {
    if (fgroups.findIndex(g => g == group) == -1) return
    fgroups = fgroups.filter(g => g != group)
  }
</script>

<Content size="lg">
  <Heading class="text-center">Groups</Heading>
  <h1 class="text-xl">
    Click follow on some groups which interest you. This will make your first NOSTR experience
    better!
  </h1>
  {#if loading}
    <Spinner delay={0} />
  {:else}
    {#each groups as group}
      <div>
        {#if group}
          <GroupCard {followGroup} {unfollowGroup} {group} />
        {/if}
      </div>
    {/each}
  {/if}
  <Anchor type="button-accent" class="text-center" on:click={() => savefgroups(fgroups)}>Let’s Go!</Anchor>
</Content>
