<script lang="ts">
    import { JSONEditor } from 'svelte-jsoneditor';
    import { getContext, onDestroy } from "svelte";

    export let fieldText;
    export let fieldJSON;
    export let fieldType;
    export let label;
    export let defaultValue;
    export let readOnly;
    export let validationText;

    const { styleable } = getContext("sdk");
    const component = getContext("component");
    const formContext = getContext("form");
    const formStepContext = getContext("form-step");
    const fieldGroupContext = getContext("field-group");

    let fieldState;
    let fieldApi;
    let errors;
    let content;
    let unsubscribe;

    const formApi = formContext?.formApi;
    const labelPos = fieldGroupContext?.labelPosition || "above";
    let configErrors = "";

    $: {
        configErrors = "";

        if (!fieldText && !fieldJSON) {
            configErrors = "Please select a form field\n";
        }

        if (fieldType !== 'text' &&  fieldType !== 'json') {
            configErrors = configErrors + "Select a field type\n";
        }

        if (fieldType !== 'text' && validationText) {
            configErrors = configErrors + "Can only use Text validaton with Text typed field\n";
        }
    }

    $: formStep = formStepContext ? $formStepContext || 1 : 1;
    $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;


    $: if (formApi && (fieldText || fieldJSON) && !configErrors) {
        const formField = formApi.registerField(
            fieldType === 'text' && fieldText || fieldType === 'json' && fieldJSON,
            fieldType,
            defaultValue,
            false,
            validationText,
            formStep
        );

        unsubscribe = formField?.subscribe((value) => {
            fieldState = value?.fieldState;
            fieldApi = value?.fieldApi;
        });
    }

    $: {
        if (fieldState?.value !== undefined && content === undefined) {
            let value;

            if (fieldType === 'text') {
                try {
                    value = JSON.parse(fieldState.value);
                }
                catch (err) {
                    contentErrors = err.toString();
                }
            }

            if (value !== undefined) {
                content = {
                    json: value
                };
            }
        }
    }

    onDestroy(() => {
        fieldApi?.deregister();
        unsubscribe?.();
    })

    function handleChange(updatedContent, previousContent, { contentErrors, patchResult }) {
        let value;
        if (typeof updatedContent.text !== 'undefined') {
            try {
                    value = JSON.parse(updatedContent.text);
                }
                catch (err) {
                    contentErrors = err.toString();
                }
        }
        else {
            value = updatedContent.json;
        }

        if (!contentErrors) {
            if (fieldType === 'text') {
                fieldApi.setValue(JSON.stringify(value))
            }
            if (fieldType === 'json') {
                fieldApi.setValue(value);
            }
        }

        errors = contentErrors;
    }
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
    {#if !formContext}
        <div class="placeholder">Form components need to be wrapped in a form</div>
    {:else}
        {#if !configErrors}
            <label
              class:hidden={!label}
              for={fieldState?.fieldId}
              class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
            >
              {label || " "}
            </label>
            <div class="spectrum-Form-itemField">
                <div class="container">
                    <JSONEditor {content} {readOnly} onChange="{handleChange}" />
              </div>
            </div>
            {#if errors}
                <div class="error">{errors}</div>
            {/if}
            {#if fieldState?.error}
                <div class="error">{fieldState.error}</div>
            {/if}
        {:else}
            <div class="error">{configErrors}</div>
        {/if}
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
