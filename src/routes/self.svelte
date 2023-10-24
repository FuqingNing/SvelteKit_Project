<script lang = "ts">

import { onMount } from 'svelte';
import * as PIXI from 'pixi.js';

let app: PIXI.Application;
interface Cell{
    isHidden: boolean;
      isBomb: boolean;
      hasFlag: boolean; // Use camelCase for property names
      bombsNearby: number; // It seems like a typo: 'bombNearBy' should be 'bombsNearby'
      id: number;
}
interface Game {
      width: number;
      height: number;
      bombsCount: number;
      cells: Cell[]
    }
      
      onMount(()=>{
        app = new PIXI.Application({
          width: 800,
          height: 600,
          backgroundColor: 0xEEEEEE,
        });
      });
const cell = new PIXI.Graphics();
cell.beginFill(0x000000);
cell.drawRect(0, 0, 100, 100);
cell.endFill();

</script>

<style>

</style>