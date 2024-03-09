<script lang="ts">
    import { JSONEditor } from 'svelte-jsoneditor';
    import { getContext, onDestroy } from "svelte";

    export let field;
    export let label;
    export let defaultValue;
    export let readOnly;
    export let validation;

    const { styleable } = getContext("sdk");
    const component = getContext("component");
    const formContext = getContext("form");
    const formStepContext = getContext("form-step");
    const fieldGroupContext = getContext("field-group");

    let fieldState;
    let fieldApi;
    let errors
    let content;
    let unsubscribe;

    const formApi = formContext?.formApi;
    const labelPos = fieldGroupContext?.labelPosition || "above";
    $: formStep = formStepContext ? $formStepContext || 1 : 1;
    $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

    $: if (formApi && field) {
        const formField = formApi.registerField(
            field,
            "text",
            defaultValue,
            false,
            readOnly,
            validation,
            formStep
        );

        unsubscribe = formField.subscribe((value) => {
            fieldState = value?.fieldState;
            fieldApi = value?.fieldApi;
        });
    }

    $: {
        if (fieldState?.value !== undefined && content === null) {
          content = {
            text: fieldState?.value
          }
        }
    }

    onDestroy(() => {
        unsubscribe?.();
    })

    function handleChange(updatedContent, previousContent, { contentErrors, patchResult }) {
        if (!contentErrors) {
            fieldApi.setValue(updatedContent.text);
        }

        errors = contentErrors;
    }
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      <div class="container">
        <JSONEditor {content} onChange="{handleChange}" />
      </div>
      {#if !field}
        <div class="error">Please select a form field</div>
      {/if}
      {#if errors}
        <div class="error">{errors}</div>
      {/if}
      {#if fieldState?.error}
        <div class="error">{fieldState.error}</div>
      {/if}
    </div>
  {/if}
</div>

<style>
  .container {
    display: flex;
    min-height: 42px;
    align-items: center;
    resize: both;
    overflow: hidden;
  }
  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }
  label {
    white-space: nowrap;
    display: block;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-item {
    display: block;
  }
  .spectrum-Form-itemField {
    width: 100%;
    align-items: center;
    display:block;
  }
  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>
