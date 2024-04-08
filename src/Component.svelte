<script>
  import { getContext, onDestroy } from "svelte";

  import CKEditor from "ckeditor5-svelte";
  import CustomEditor from "ckeditor5-custom";

  import "./custom.css";

  export let field;
  export let label;

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

  let editor = CustomEditor;
  let editorInstance = null;
  let initialData = "";

  setTimeout(() => {
    initialData = fieldState.value;
  }, 100);

  let editorConfig = {
    removePlugins: ["Title"],
    autosave: {
      save(editor) {
        return saveData(editor.getData());
      },
    },
    toolbar: {
      items: [
        "selectAll",
        "|",
        "heading",
        "|",
        "bold",
        "italic",
        "strikethrough",
        "underline",
        "|",
        "alignment",
        "removeFormat",
        "|",
        "fontColor",
        "fontBackgroundColor",
        "fontSize",
        "|",
        "bulletedList",
        "numberedList",
        "|",
        "HorizontalLine",
        "specialCharacters",
        "link",
        "blockQuote",
        "|",
        "undo",
        "redo",
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

  function saveData(data) {
    displayStatus(true);

    return setTimeout(() => {
      displayStatus();

      fieldApi.setValue(data);
    }, 200);
  }

  function displayStatus(saved = false) {
    const statusIndicator = document.querySelector("#save");

    if (saved) {
      statusIndicator.style.display = "block";
    } else {
      statusIndicator.style.display = "none";
    }
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
        bind:value={initialData}
      />

      <span id="save">&#128190;</span>
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
  #save {
    display: none;
    position: absolute;
    top: 10px;
    right: -28px;
    font-size: 18px;
    transition: 0.5s;
    opacity: 0.5;
  }
</style>
