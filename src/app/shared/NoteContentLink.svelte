<script lang="ts">
  import {annotateMedia} from "src/util/misc"
  import Anchor from "src/partials/Anchor.svelte"
  import Media from "src/partials/Media.svelte"

  export let value
  export let showMedia

  const close = () => {
    hidden = true
  }

  let hidden = false
</script>

{#if showMedia && value.isMedia}
  <div class="py-2">
    <Media link={annotateMedia(value.url)} onClose={close} />
  </div>
{:else}
  <Anchor external href={value.url}>
    {value.url.replace(/https?:\/\/(www\.)?/, "")}
  </Anchor>
{/if}
