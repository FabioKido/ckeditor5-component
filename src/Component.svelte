<script>
  import { getContext, onDestroy } from "svelte"

  export let field
  export let label
  export let background

  let fieldApi
  let fieldState

  const { styleable } = getContext("sdk")
  const component = getContext("component")
  const formContext = getContext("form")
  const formStepContext = getContext("form-step")
  const fieldGroupContext = getContext("field-group")

  const formApi = formContext?.formApi

  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    null,
    formStep
  )

  const labelPos = fieldGroupContext?.labelPosition || "above"

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState
    fieldApi = value?.fieldApi
  })

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })



  function onHandleChange() {
    fieldApi.setValue()
  }

</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form.</div>
  {:else if !field}
    <div class="error">Please select a field</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>

    <div class="spectrum-Form-itemField">
      <!-- <div
        on:change={() => onHandleChange()}
        style:background-color={background}
        id="editorjs"
      /> -->
        Valor atual no field: {fieldState?.value}
    </div>

    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>