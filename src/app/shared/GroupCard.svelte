<script lang="ts">
  import cx from "classnames"
  import {fly} from "svelte/transition"
  import {stringToHue, hsl} from "src/util/misc"
  import Anchor from "src/partials/Anchor.svelte"

  export let group
  export let theme = "gray-8"
  export let showStatus = false
  export let hideActions = false
  export let followGroup
  export let unfollowGroup

  let hasgroup = false
  let statusHover = false
</script>

<div
  class={cx(
    `bg-${theme}`,
    "flex flex-col justify-between gap-3 rounded border border-l-2 border-solid border-gray-6 py-3 px-6 shadow"
  )}
  style={`border-left-color: ${hsl(stringToHue(group))}`}
  in:fly={{y: 20}}>
  <div class="flex items-center justify-between gap-2">
    <div class="flex items-center gap-2 text-xl">
      <i class="fa fa-user-group" />
      <Anchor type="unstyled" href={`/groupusers/${group}`}>
        {group}
      </Anchor>
      {#if showStatus}
        <span
          on:mouseout={() => {
            statusHover = false
          }}
          on:mouseover={() => {
            statusHover = true
          }} />
        <p
          class="hidden text-sm text-gray-1 transition-all sm:block"
          class:opacity-0={!statusHover}
          class:opacity-1={statusHover} />
      {/if}
    </div>
    {#if !hideActions}
      <slot name="actions">
        {#if !hasgroup}
          <button
            class="flex items-center gap-3 text-gray-1"
            on:click={() => {
              followGroup(group)
              hasgroup = true
            }}>
            <i class="fa fa-right-to-bracket" /> Follow
          </button>
        {:else}
          <button
            class="flex items-center gap-3 text-gray-1"
            on:click={() => {
              unfollowGroup(group)
              hasgroup = false
            }}>
            <i class="fa fa-right-from-bracket" /> Unfollow
          </button>
        {/if}
      </slot>
    {/if}
  </div>
</div>
