<script lang="ts">
  import { onMount } from "svelte";
  import {
    addFinalState,
    addTransition,
    FSM,
    removeFinalState,
  } from "$stores/FSM";
  import { instance } from "@viz-js/viz";
  import { toDotScript } from "$lib/converter";
  import { Button, GradientButton, Input, Label, Select } from "flowbite-svelte";
  import Table from "$lib/Table.svelte";
  import type { FSMJSON } from "../app";
  import {TrashBinOutline} from "flowbite-svelte-icons";

  let Graph: HTMLElement;
  let finalState: string | null = null;
  let FSM_JSON: FSMJSON = $FSM;
  let currentNodes = FSM_JSON.acceptStates.map(state => ({ name: state, value: state }));

  onMount(() => {
    currentNodes = FSM_JSON.acceptStates.map(state => ({ name: state, value: state }));
    return FSM.subscribe((value) => {
      FSM_JSON = value;
      renderGraph();
    });
  });

  function renderGraph() {
    const dotString = toDotScript(FSM_JSON);
    console.log(dotString);
    instance().then((viz) => {
      Graph.innerHTML = viz.renderSVGElement(dotString).outerHTML;
    });
  }

  function handleSubmit(event: Event) {
    const form = event.target as HTMLFormElement;
    const formData = new FormData(form);
    const fromState = formData.get("FromState") as string;
    const inputSymbol = formData.get("InputSymbol") as string;
    const toState = formData.get("ToState") as string;
    console.log(fromState, inputSymbol, toState);
    addTransition(fromState, toState, inputSymbol);
    form.reset();
  }
</script>

<div class="flex flex-row justify-evenly gap-10">
  <div class="w-[40%] flex flex-col gap-4">
    <div class="flex flex-row gap-10">
      <Label class="block mb-2 text-lg pt-2" for="first_name"
        >Initial state</Label
      >
      <!--<Input type="text" placeholder="initial state" required bind:value={FSM_JSON.initialState} on:input={renderGraph}/>-->
      <Input
        class="w-36"
        bind:value={$FSM.initialState}
        on:input={renderGraph}
        placeholder="initial state"
        required
        type="text"
      />
    </div>
    <div class="flex flex-row gap-10">
      <Label class="block mb-2 text-lg pt-2" for="first_name">End state</Label>
      <Input
        class="w-36 ml-5"
        bind:value={finalState}
        placeholder="add final state"
        required
        type="text"
      />
      <Button
        color="none"
        class="bg-fontColor"
        on:click={() => {
          addFinalState(finalState);
          finalState = null;
        }}
        >Add
      </Button>
    </div>
      <div class="text-black dark:text-white text-2xl text-center">End States</div>
        {#if FSM_JSON.acceptStates.length > 0}
      <div class="final-states-container flex flex-row flex-wrap gap-4 text-center">
        {#each FSM_JSON.acceptStates as state}
          <div class="final-state-item flex  items-center mb-4 bg-primary-dark rounded-lg">
            <h5
              class="pl-4 text-2xl font-bold tracking-tight text-gray-900 dark:text-white"
            >
              {state}
            </h5>
            <Button
            class="p-2"
              color="none"
              on:click={() => removeFinalState(state)}
            >
              <TrashBinOutline class="text-black dark:text-white" />
            </Button>
          </div>
        {/each}
      </div>
    {/if}
    <div class="relative bg-fontColor p-8 min-h-52 text-black text-center rounded-xl">
        <div class="text-xl font-bold mb-4">Add Transitions</div>
        <form on:submit|preventDefault={handleSubmit}>
          <div class="flex flex-row justify-evenly">
              <div class="flex flex-col justify-center items-center">
                  <Label class="mb-2 dark:text-black font-bold" for="inputSymbol">From State</Label>
                  <Input class="w-24" name="FromState" placeholder="eg. a, b" required type="text" />
              </div>
              <div class="flex flex-col justify-center items-center">
                  <Label class="mb-2 dark:text-black font-bold" for="inputSymbol">Input Symbol</Label>
                  <Input class="w-24 bg-primary-dark" name="InputSymbol" placeholder="eg. 0, 1" required type="text" />
              </div>
              <div class="flex flex-col justify-center items-center">
                  <Label class="mb-2 dark:text-black font-bold" for="toState">To State</Label>
                  <Input class="w-24 bg-primary-dark" name="ToState" placeholder="eg. c, d" required type="text" />
              </div>
          </div>
          <Button color="none" class="absolute bg-primary-dark text-white rounded-none rounded-br-xl rounded-tl-xl bottom-0 right-0" type="submit"
            >Add Transition</Button
          >
        </form>
    </div>
    <Table />
  </div>
  <div class="bg-primary-dark flex-1 rounded-xl flex flex-col">
    <p class=" text-white p-5 m-0 text-3xl">Final Output    :</p>
    <div class="flex-1 text-center flex justify-center items-center">
      <div bind:this={Graph} class="sample" />
    </div>
  </div>
</div>
