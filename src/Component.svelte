<script>
  // default slot does not work
  // how do i stop the import happing in preview - can i detect preview
  // API is not documented
  // documentation on how to trigger a defineActions?
  import { getContext } from "svelte";
  import Dropzone from "svelte-file-dropzone";
  import Papa from "papaparse";

  export let table;
  export let importText="Import";
  export let resetText="Reset";
  export let copyErrorText = "Copy errors"
  export let onImport= () => {};
  export let dragzoneText;

  const { styleable, Provider, API, builderStore } = getContext("sdk");
  const component = getContext("component");

  let file = {};
  let data = [];
  let parseErrors = [];
  let importErrors = [];
  let isParsed = false;
  let isImporting = false;
  let isImported = false;
  let importedCount = 0;
  $: hasRows = data.length > 0;

  $: dataContext = {
    data,
    file,
    parseErrors,
    importErrors,
    isParsed,
    isImported,
    isImporting,
    importedCount,
  };

  function handleFilesSelect(e) {
    const { acceptedFiles } = e.detail;
    if (acceptedFiles.length === 0) {
      file = null;
      return;
    }

    file = acceptedFiles[0];
    Papa.parse(file, {
      header: true,
      complete: (result) => {
        data = result.data;
        isParsed = true;
        parseErrors = result.errors;
      }
    });
  }

  async function importData() {
    isImporting = true;
    for (let row of data) {
      // do not import in builder mode
      if (!$builderStore.inBuilder) {
        try {
          importedCount++;
          await API.saveRow({
            tableId: table.tableId,
            ...row
          });
        } catch(e) {
          importErrors.push({
            rowNumber:data.indexOf(row) + 1,
            error: e
          });
        }
      }
    }
    isImporting = false;
    isImported = true;
    onImport();
  }

  function reset() {
    data = [];
    file = null;
    parseErrors = [];
    importErrors = [];
    isParsed = false;
    isImported = false;
    isImporting = false;
    importedCount = 0;
  }

  function copyErrors() {
    navigator.clipboard.writeText(JSON.stringify(importErrors));
  }

</script>

<div use:styleable={$component.styles} >
  {#if !isParsed}
    <Dropzone 
      on:drop={handleFilesSelect} 
      multiple=false>
      <p>{dragzoneText}</p>
    </Dropzone>
  {/if}
  <Provider data={dataContext}>
    <slot/>
    {#if !$component.children}
      <div class="info">
        {#if isParsed && !isImported}
          <p>{file.name}&nbsp;-&nbsp;{data.length}&nbsp;rows found</p>
        {/if}
        {#if isImporting}
          <p>{importedCount}/{data.length}</p>
        {/if}
        {#if isImported}
          {#if importErrors.length === 0}
            <p>Import complete!</p>
          {:else}
            <div>
              {`${data.length - importErrors.length} Imported,  ${importErrors.length} Errored`}
            </div>
          {/if}
        {/if}
      </div>
    {/if}
  </Provider>

  <div class="buttons">
    {#if isParsed || importErrors.length > 0}
      <button 
        on:click={reset}
        class="spectrum-Button spectrum-Button--quiet spectrum-Button--secondary spectrum-Button--sizeS">
          {resetText}
      </button>
    {/if}
    {#if isParsed && importErrors.length > 0}
      <button 
        class="spectrum-Button spectrum-Button--fill spectrum-Button--primary spectrum-Button--sizeS"
        on:click={copyErrors}>
          {copyErrorText}
      </button>
    {/if}
    {#if isParsed && hasRows && !isImported}
      <button
        on:click={importData}
        class="spectrum-Button spectrum-Button--fill spectrum-Button--primary spectrum-Button--sizeS">
        {importText}
      </button>
    {/if}
  </div>

</div>

<style>
  .info {
    margin-left: 12px;
  }
  .buttons {
    margin-top: 10px;
  }
</style>
