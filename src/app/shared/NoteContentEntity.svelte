<script lang="ts">
  import Anchor from "src/partials/Anchor.svelte"

  export let value
</script>

<Anchor killEvent href={"/" + value.entity}>
  {value.entity.slice(0, 16) + "..."}
</Anchor>
