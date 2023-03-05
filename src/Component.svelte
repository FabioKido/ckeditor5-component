<script>
  import { getContext, onDestroy } from "svelte";

  import CKEditor from "ckeditor5-svelte";
  import ClassicEditor from "@ckeditor/ckeditor5-build-classic/build/ckeditor";

  export let field;
  export let label;
  export let background;

  let fieldApi;
  let fieldState;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  const formApi = formContext?.formApi;

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    null,
    formStep
  );

  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  let editor = ClassicEditor;
  let editorInstance = null;

  let editorConfig = {
    toolbar: {
      items: [
        "selectAll",
        "|",
        "heading",
        "|",
        "bold",
        "italic",
        "strikethrough", // precisa de lib
        "underline",  // precisa de lib
        "|",
        "bulletedList",
        "numberedList",
        "|",
        "undo",
        "redo",
        "fontSize", // precisa de lib
        "fontFamily", // precisa de lib
        "fontColor",  // precisa de lib
        "fontBackgroundColor",  // precisa de lib
        "|",
        "alignment",  // precisa de lib
        "|",
        "link",
        "blockQuote"
      ],
    },
  };

  function onReady({ detail: editor }) {
    editorInstance = editor;
    editor.ui
      .getEditableElement()
      .parentElement.insertBefore(
        editor.ui.view.toolbar.element,
        editor.ui.getEditableElement()
      );
  }

  function onHandleChange() {
    fieldApi.setValue();
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
      <CKEditor
        bind:editor
        on:ready={onReady}
        bind:config={editorConfig}
        bind:value={fieldState.value}
      />
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
