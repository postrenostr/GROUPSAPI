<script lang="ts">
  import {uniq} from "ramda"
  import {onMount} from "svelte"
  import {generatePrivateKey} from "nostr-tools"
  import {fly} from "svelte/transition"
  import {navigate} from "svelte-routing"
  import {shuffle} from "src/util/misc"
  import {displayPerson} from "src/util/nostr"
  import OnboardingIntro from "src/app/views/OnboardingIntro.svelte"
  import OnboardingProfile from "src/app/views/OnboardingProfile.svelte"
  import OnboardingKey from "src/app/views/OnboardingKey.svelte"
  import OnboardingRelays from "src/app/views/OnboardingRelays.svelte"
  import OnboardingFollows from "src/app/views/OnboardingFollows.svelte"
  import OnboardingNote from "src/app/views/OnboardingNote.svelte"
  import {getFollows, defaultFollows} from "src/agent/social"
  import {getPubkeyWriteRelays, sampleRelays} from "src/agent/relays"
  import {getPersonWithFallback} from "src/agent/db"
  import network from "src/agent/network"
  import user from "src/agent/user"
  import pool from "src/agent/pool"
  import keys from "src/agent/keys"
  import cmd from "src/agent/cmd"
  import {loadAppData} from "src/app/state"
  import {toHex} from "src/util/nostr"
  import {modal} from "src/partials/state"
  import OnboardingGroup from "./OnboardingGroup.svelte"

  export let stage

  const privkey = generatePrivateKey()
  const profile = {}
  const {relays} = user

  let fgroups = []

  if ($relays.length === 0) {
    user.updateRelays(() =>
      (pool.forceUrls.length > 0 ? pool.forceUrls : pool.defaultUrls).map(url => ({
        url,
        write: true,
      }))
    )
  }

  const savefgroups =async (groups) => {
    fgroups = groups;
    if (user.getPetnamePubkeys().length === 0) {
      user.updatePetnames(() =>
        defaultFollows.map(pubkey => {
          const [{url}] = sampleRelays(getPubkeyWriteRelays(pubkey))
          const name = displayPerson(getPersonWithFallback(pubkey))

          return ["p", pubkey, url, name]
        })
      )
    }
    signup(null);
    // modal.replace({type: "onboarding", stage: "note"})
  }

  const signup = async note => {
    await keys.login("privkey", privkey)
    // Re-save preferences now that we have a key
    await Promise.all([
      user.updateRelays(() => user.getRelays()),
      cmd.updateUser(profile).publish(user.getRelays()),
      note && cmd.createNote(note).publish(user.getRelays()),
      user.updatePetnames(() =>
        user.getPetnamePubkeys().map(pubkey => {
          const [{url}] = sampleRelays(getPubkeyWriteRelays(pubkey))
          const name = displayPerson(getPersonWithFallback(pubkey))

          return ["p", pubkey, url, name]
        })
      ),
    ])

    loadAppData(user.getPubkey())

    setTimeout(() => {
      followUsers();
    }, 2000);

    modal.clear()
    navigate("/notes")
  }

  const followUsers = async () => {
    for (let i = 0; i < fgroups.length; i++) {
      const group = fgroups[i];
      const res = await fetch("https://manyworlds.network:3306/groups/" + group, {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
        },
      })
      let data = await res.json()
      if (data.message == "Success") {
        for (let i = 0; i < data.data.length; i++) {
          const npub = data.data[i];
          const pubkey = npub.startsWith("npub") ? toHex(npub) : npub;
          if (user.getPetnames().findIndex(p => p[1] == pubkey) >= 0) continue;
          const [{url}] = sampleRelays(getPubkeyWriteRelays(pubkey));
          await user.addPetname(pubkey, url, displayPerson(getPersonWithFallback(pubkey)));
        }
      }
    }
  }

  // Prime our people cache for hardcoded follows and a sample of people they follow
  onMount(async () => {
    const relays = sampleRelays(user.getRelays())
    const follows = user.getPetnamePubkeys().concat(defaultFollows)

    await network.loadPeople(follows, {relays})

    const others = shuffle(uniq(follows.flatMap(getFollows))).slice(0, 256)

    await network.loadPeople(others, {relays})
  })
</script>

{#key stage}
  <div in:fly={{y: 20}}>
    {#if stage === "intro"}
      <OnboardingIntro />
    {:else if stage === "profile"}
      <OnboardingProfile {profile} />
    {:else if stage === "key"}
      <OnboardingKey {privkey} />
    {:else if stage === "relays"}
      <OnboardingRelays />
    {:else if stage === "follows"}
      <OnboardingFollows />
    {:else if stage === "group"}
      <OnboardingGroup {savefgroups}/>
    {:else if stage === "note"}
      <OnboardingNote {signup} />
    {/if}
  </div>
{/key}
