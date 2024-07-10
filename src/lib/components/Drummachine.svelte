<script lang="ts">
  // @ts-nocheck

  import * as Tone from "tone";
  import Drummixer from "$lib/components/Drummixer.svelte";
  import EffectList from "$lib/components/EffectList.svelte";
  import { bpmStore, effectDrumStore, beatStore, playStore, moveDrumEffectUp, moveDrumEffectDown, removeDrumEffect, addDrumEffect, activeDrumStore, soloDrumStore, muteDrumStore, updateDrumState, panDrumStore, gainDrumStore, delayParams, distortionParams, reverbParams, chorusParams } from '$lib/stores';
  import { createReverb } from '$lib/effects';
  import { get } from 'svelte/store';
  import { onDestroy } from 'svelte';
 

  const samples = [
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/perc_2.wav"
      },
    }),
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/perc_1.wav"
      },
    }),
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/hat.wav"
      },
    }),
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/clap.wav"
      },
    }),
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/snare.wav"
      },
    }),
    new Tone.Sampler({urls: {
        C3: "/samples/blackpearl/kick.wav"
      },
    }),
  ];

  let beatIndicators = Array.from({ length: 16}, (_, i) => i);
/*
  effectDrumStore.subscribe(effectList => {
    samples.forEach((sample, index) => {
      sample.disconnect();

      if (effectList.length > 0) {
        let currentNode = sample;
        effectList.forEach(effect => {
          currentNode.connect(effect.toDestination());
          currentNode = effect;
        });
        sample.chain($panDrumStore[index], $gainDrumStore[index], Tone.Destination);
      } else {
        sample.chain($panDrumStore[index], $gainDrumStore[index], Tone.Destination);
      }

    });
  });

*/
  effectDrumStore.subscribe(effectList => {
    samples.forEach((sample, index) => {
      sample.disconnect();
      const pan = get(panDrumStore)[index];
      const gain = get(gainDrumStore)[index];

      sample.chain(
        gain,
        pan,
        ...effectList,
        Tone.Destination
      );
    });
  });

  Tone.Transport.scheduleRepeat(time => {
   $activeDrumStore.map((row, index) => {
  let sample = samples[index];
    let currentBeat = row[$beatStore];
      const soloIndex = $soloDrumStore.findIndex(value => value === true);

      if (currentBeat) {
        if (soloIndex >= 0 && samples.length - 1 - soloIndex === index) {
          if (!($muteDrumStore[samples.length - index - 1])) {
            sample.triggerAttackRelease("C3", "8n", time);
          }
        } else if (soloIndex === -1) {
          if (!($muteDrumStore[samples.length - index - 1])) {
            sample.triggerAttackRelease("C3", "8n", time);
          }
        }
      }

    });

    beatStore.update((beat) => (beat + 1) % 16);
  }, '16n');

  const handleNoteClick = (rowIndex: number, colIndex: number) => {
    updateDrumState(rowIndex, colIndex, !($activeDrumStore[rowIndex][colIndex]));
  }

  const handleClearClick = () => {
    for (let i = 0; i < $activeDrumStore.length; i++) {
      for (let j = 0; j < $activeDrumStore[i].length; j++) {
        updateDrumState(i, j , false);
      }
    }
  }

    // Mouse click und drag events
  let isDragging = false;
  let lastClickedRowIndex = -1;
  let lastClickedNoteIndex = -1;
  let initialActiveState = false; 

    // Funktion zum draggen und ändern des zustands
    const handleStartDrag = (rowIndex, colIndex) => {
    isDragging = true;
    lastClickedRowIndex = rowIndex;
    lastClickedNoteIndex = colIndex;
    initialActiveState = $activeDrumStore[rowIndex][colIndex].active; 
    handleNoteClick(rowIndex, colIndex); 
    };

  // Funktion zum handeln von mouseover events während des draggen
  const handleDragOver = (rowIndex, noteIndex) => {
    if (isDragging) {
      // Ensure it only toggles between active or inactive based on initial state
      if (initialActiveState === $activeDrumStore[rowIndex][noteIndex].active) {
        handleNoteClick(rowIndex, noteIndex);
      }
      lastClickedRowIndex = rowIndex;
      lastClickedNoteIndex = noteIndex;
    }
  };

  const handleStopDrag = () => {
    isDragging = false;
  };


  $: if ($playStore) {
    Tone.Transport.bpm.value = $bpmStore;
  }

  onDestroy(() => {
    samples.map(sample => {
      sample.disconnect();
      sample.dispose();
    });
    Tone.Transport.cancel();
    Tone.Transport.stop();
  });

</script>


<div class="controls">
  <button on:click={handleClearClick}> CLEAR DRUM </button>
</div>


<div class="container">

  <div class="beat-indicators">
    {#each beatIndicators as beatIndicator, bi}
      <div class="beat-indicator {bi === $beatStore -1 ? 'live' : ''}"></div>
    {/each}
  </div>

<!-- svelte-ignore a11y-mouse-events-have-key-events -->
  <div class="drummachine">
    {#each $activeDrumStore as row, i}    
      {#each row as beat, j}
        <button
        on:mousedown={() => handleStartDrag(i, j)}
        on:mouseover={() => handleDragOver(i, j)}
        on:mouseup={handleStopDrag}
        class="beat {$activeDrumStore[i][j] ? 'active' : ''}"
        ></button>
      {/each}
    {/each}

  </div>


  <Drummixer />
  <div>
    <EffectList moveEffectUpProp={moveDrumEffectUp} moveEffectDownProp={moveDrumEffectDown} removeEffectProp={removeDrumEffect} addEffectProp={addDrumEffect} effects={effectDrumStore}/>
  </div>


</div>






<style>

  #Drummachine {
    padding-top: 5rem;
    grid-column: 2/3;
    grid-row: 2;
  }

  #Komponente {
    max-width: 1280px;
    margin: 0 auto;
    padding: 2rem;
    text-align: center;
  }

  .controls {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
    margin-top: 20px;
  }

  .controls button {
    background-color: #ce0505; /* Green background */
    border: 1px solid rgb(43, 43, 43); /* Green border */
    color: white; /* White text */
    padding: 17px 30px; /* Some padding */
    cursor: pointer; /* Pointer/hand icon */
    float: left; /* Float the buttons side by side */
    font-family: "Inter", sans-serif;
    border-radius: 30px;
  }

  .container {
    display: grid;
    position: relative;
    justify-content: center;
    align-items: center;
    gap: 1em;
    margin-top: 3em;
    margin-bottom: 3em;
    z-index: -100;
  }

  .beat-indicators {
    display: grid;
    grid-template-columns: repeat(16, 1fr);
    gap: 4px;
    margin-left: 13px;
    width: 100%;
    place-items: center;
  }

  .beat-indicator {
    background: #ccc;
    width: 15px;
    height: 15px;
    border: 1px solid #ccc;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-sizing: border-box;
    margin: 0.5em;
  }

  .live {
    background: #2BE127;
  }
  
  .drummachine {
    display: grid;
    grid-template-columns: repeat(16, 1fr);
    margin-left: 15px;
    gap: 5px;
    width: 100%;
  }

  .beat {
    background: #ccc;
    width: 3em;
    height: 3em;
    border: 1px solid #ccc;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0;
    box-sizing: border-box;
  }

  .beat.active {
    background-color: #2BE127;
    border-color: #2BE127;
  }

  .beat:hover {
    opacity: 0.8;
    cursor: pointer;
  }

  .beat:nth-child(4n - 3) {
    border-color: #2BE127;
  }

  
</style>
