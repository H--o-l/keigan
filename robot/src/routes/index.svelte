
<svelte:head>
  <script src="https://www.puck-js.com/puck.js"></script>
</svelte:head>

<script lang='ts'>
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js';

  let kmbRight: any; // JPY9
  let kmbLeft: any; // F9
  let leftOk = false;
  let speed = 0;
  let displayCoords = {x: 50, y: 50};

  kmbRight = new (window as any).KMMotorOneWebBLE();
  kmbLeft = new (window as any).KMMotorOneWebBLE();

  kmbRight.on(kmbRight.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    kmbRight.cmdDisableIMUMeasurement();
    kmbRight.cmdDisable();
    kmbRight.cmdSpeed_rpm(0);
    kmbRight.cmdMaxTorque(0.15);
    kmbRight.cmdMaxSpeed(50);
    kmbRight.cmdLed(1, 0, 0, 255);
    kmbRight.cmdEnable();
    kmbRight.cmdRunForward();
    leftOk = true;
  });
  kmbLeft.on(kmbLeft.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    kmbLeft.cmdDisableIMUMeasurement();
    kmbLeft.cmdDisable();
    kmbLeft.cmdSpeed_rpm(0);
    kmbLeft.cmdMaxTorque(0.15);
    kmbLeft.cmdMaxSpeed(50);
    kmbLeft.cmdLed(1, 0, 0, 255);
    kmbLeft.cmdEnable();
    kmbLeft.cmdRunReverse();
  });
  kmbRight.on(kmbRight.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
    console.warn('error: ', err);
  });
  kmbLeft.on(kmbRight.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
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
  async function connectRight() {
    await kmbRight.connect();
  }
  async function connectLeft() {
    await kmbLeft.connect();
  }
  function stop() {
    kmbRight.cmdSpeed_rpm(0);
    kmbRight.cmdRunForward();
    kmbLeft.cmdSpeed_rpm(0);
    kmbLeft.cmdRunReverse();
  }
  async function disconnect() {
    kmbRight.cmdFree();
    kmbRight.cmdLed(1, 255, 0, 0);
    kmbLeft.cmdFree();
    kmbLeft.cmdLed(1, 255, 0, 0);
    await new Promise(r => setTimeout(r, 1000));
    kmbRight.disConnect();
    kmbLeft.disConnect();
  }
  function onChange() {
    kmbRight.cmdSpeed_rpm(speed);
    kmbRight.cmdRunForward();
    kmbLeft.cmdSpeed_rpm(speed);
    kmbLeft.cmdRunReverse();
  }
</script>

<div class='global'>
  <div class='column'>
    <button on:click={stop}>stop</button>
    <button on:click={connectRight}>connectRight</button>
    <button on:click={connectLeft}>connectLeft</button>
    <button on:click={disconnect}>disConnect</button>
    <label style='display: block;'>
      Speed:
      <input type='range' bind:value={speed} on:change={onChange} min='0' max='50'>
    </label>
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
