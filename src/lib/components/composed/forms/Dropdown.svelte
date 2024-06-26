<script lang="ts" context="module">
  import type { Item } from "../../../components/simple/forms/Autocomplete.svelte";
  export type { Item }
</script>

<script lang="ts">
  import lodash from "lodash";
  import Autocomplete from "../../../components/simple/forms/Autocomplete.svelte";
  import Button from '../../simple/buttons/Button.svelte'
  import Icon from '../../simple/media/Icon.svelte'
  import { createEventDispatcher } from "svelte";

  let dispatch = createEventDispatcher<{
    change: {
      unselect: Item | undefined;
      select: Item | undefined;
      selection: Item[];
    }
  }>()

  export let items: Item[] = [],
    values: Item[] = [],
    multiple: boolean = false,
    lang: 'it' | 'en' = 'en',
    searchText: string | undefined = undefined,
    maxVisibleChips: number | undefined = undefined,
    placeholder: string = lang == 'en' ? "Select" : "Seleziona",
    clearable: boolean = true,
    mandatory: boolean = true,
    icon: string | undefined = undefined,
    menuOpened: boolean = false,
    openingId: string | undefined = undefined,
    width: string | undefined = undefined,
    minWidth: string | undefined = undefined,
    menuWidth: string | undefined = width,
    mobileDrawer: boolean = false,
    disabled: boolean = false

  $: generatedLabel = values.length == 1 ? values[0].label : `${values.length} Selezionati`

  function handleCloseClick(event: MouseEvent) {
    event.preventDefault()
    event.stopPropagation()
    let valuesBefore = lodash.cloneDeep(values)
    values = []

    dispatch('change', {
      unselect: valuesBefore[0],
      select: undefined,
      selection: []
    })
  }
</script>

<Autocomplete
  bind:items
  bind:values
  bind:searchText
  bind:multiple
  bind:maxVisibleChips
  bind:mandatory
  bind:disabled
  searchFunction={() => true}
  on:change
  bind:menuOpened
  bind:openingId
  bind:width
  {mobileDrawer}
  bind:minWidth
  bind:menuWidth
>
  <svelte:fragment slot="selection-container" let:openMenu let:handleKeyDown>
    <Button
      --button-default-background-color="transparent"
      --button-default-focus-background-color="rgb(var(--global-color-primary-400), .3)"
      --button-default-focus-color="rgb(var(--global-color-contrast-900))"
      --button-default-border="2px solid rgb(var(--global-color-primary-400))"
      --button-default-color="rgb(var(--global-color-contrast-800))"
      on:click={openMenu}
      on:keydown={(event) => {
        handleKeyDown(event.detail.nativeEvent)
        if(event.detail.nativeEvent.key == 'ArrowDown' || event.detail.nativeEvent.key == 'ArrowUp') {
          event.detail.nativeEvent.stopPropagation()
          event.detail.nativeEvent.preventDefault()
        }
      }}
    >
      <slot
        name="label"
        {values}
        {items}
        {searchText}
        {generatedLabel}
        {placeholder}
        {clearable}
        {handleCloseClick}
      >
        <div class="label">
          {#if !!icon}
            <Icon name={icon}></Icon>
          {/if}
          {#if values.length == 0}
            <div class="space-between">
              <div>{placeholder}</div>
              <Icon name="mdi-chevron-down"></Icon>
            </div>
          {:else}
            <div class="space-between">
              <div>{generatedLabel}</div>
              {#if clearable}
                <Icon
                  name="mdi-close"
                  click
                  on:click={handleCloseClick}
                ></Icon>
              {/if}
            </div>
          {/if}
        </div>
      </slot>
    </Button>
  </svelte:fragment>
  <svelte:fragment slot="item-label" let:item >
    <slot name="item-label" {item}>
      {item.label}
    </slot>
  </svelte:fragment>
</Autocomplete>

<style>
  .label {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    gap: 10px;
    width: 100%;
    text-overflow: ellipsis;
    white-space: pre;
  }

  .space-between {
    flex-grow: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
  }
</style>