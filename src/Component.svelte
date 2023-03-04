<script>
  import { getContext } from "svelte"

  export let label

  const { styleable } = getContext("sdk")
  const component = getContext("component")
</script>

<div use:styleable={$component.styles}>
  This is a custom component. The text setting is: {label}.
</div>
