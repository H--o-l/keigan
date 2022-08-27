
<svelte:head>
  <script src="https://www.puck-js.com/puck.js"></script>
</svelte:head>

<script lang='ts'>
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js';

  let KMB1: any;
  let KMB2: any;
  let speed = 30;
  let displayCoords = {x: 50, y: 50};

  KMB1 = new (window as any).KMMotorOneWebBLE();
  KMB2 = new (window as any).KMMotorOneWebBLE();

  KMB1.on(KMB1.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    KMB1.cmdDisableIMUMeasurement();
    KMB1.cmdDisable();
    KMB1.cmdMaxTorque(0.15);
    KMB1.cmdMaxSpeed(50);
    KMB1.cmdLed(1, 0, 0, 255);
  });
  KMB2.on(KMB2.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    KMB2.cmdDisableIMUMeasurement();
    KMB2.cmdDisable();
    KMB2.cmdMaxTorque(0.15);
    KMB2.cmdMaxSpeed(50);
    KMB2.cmdLed(1, 0, 0, 255);
  });
  KMB1.on(KMB1.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
    console.warn('error: ', err);
  });
  KMB2.on(KMB1.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
    console.warn('error: ', err);
  });

  var connection: any;
  function bangleConnect() {
    if (connection) {
      connection.close();
      connection = undefined;
    }
    (Puck as any).connect((c: any) => {
      if (!c) {
        alert("Couldn't connect!");
        return;
      }
      connection = c;
      connection.on("data", (data: any) => {
        if (typeof data === 'string' &&
            data.startsWith('s') &&
            data.endsWith('e')) {
          const array = data.slice(1, -1).split(',');
          const coords = {
            x: parseInt(array[0]),
            y: parseInt(array[1]),
            z: parseInt(array[2]),
          }
          newAcc(coords)
        }
      });
    });
  }
  function newAcc(coords: {x: number, y: number, z: number}) {
    displayCoords.x = 100 - ((coords.x + 100) / 2);
    displayCoords.y = (coords.y + 100) / 2;
  }
  function onMove1() {
    KMB1.cmdEnable();
    KMB1.cmdSpeed_rpm(speed);
    KMB1.cmdRunForward();
  }
  function onMove2() {
    KMB2.cmdEnable();
    KMB2.cmdSpeed_rpm(speed);
    KMB2.cmdRunReverse();
  }
  function stop1() {
    KMB1.cmdFree();
  }
  function stop2() {
    KMB2.cmdFree();
  }
  async function disconnect1() {
    KMB1.cmdFree();
    KMB1.cmdLed(1, 255, 0, 0);
    await new Promise(r => setTimeout(r, 1000));
    KMB1.disConnect();
  }
  async function disconnect2() {
    KMB2.cmdFree();
    KMB2.cmdLed(1, 255, 0, 0);
    await new Promise(r => setTimeout(r, 1000));
    KMB2.disConnect();
  }
  function onChange() {
    KMB1.cmdSpeed_rpm(speed);
    KMB1.cmdRunForward();
    KMB2.cmdSpeed_rpm(speed);
    KMB2.cmdRunReverse();
  }
</script>

<div class='global'>
  <label style='display: block;'>
    Speed:
    <input type='range' bind:value={speed} on:change={onChange} min='25' max='50'>
  </label>
  <div class='column'>
    <button on:click={KMB1.connect()}>connect1</button>
    <button on:click={onMove1}>move1</button>
    <button on:click={stop1}>stop1</button>
    <button on:click={disconnect1}>disConnect1</button>
  </div>
  <div class='column'>
    <button on:click={KMB2.connect()}>connect2</button>
    <button on:click={onMove2}>move2</button>
    <button on:click={stop2}>stop2</button>
    <button on:click={disconnect2}>disConnect2</button>
  </div>
  <div class='column'>
    <button on:click={bangleConnect}>bangle</button>
    <div class="rectangle">
      <div class="box" style="left: {displayCoords.x * 1.35}px; bottom: {displayCoords.y * 1.35}px"></div>
    </div>
  </div>
</div>

<style>
  .global {display: flex;}
  .column {width: 150px;}
  button {width: 150px; height: 150px;}
  .rectangle {
    box-sizing: border-box;
    width: 150px;
    height: 150px;
    border-color: #000;
    border-style: solid;
    position: relative;
  }
  .box {
    position: absolute;
    width: 10px;
    height: 10px;
    border-radius: 4px;
    background-color: #ff3e00;
  }
</style>
