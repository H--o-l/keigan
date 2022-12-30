
<script lang='ts'>
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js';
  import './../puck.js';
  import {onMount} from 'svelte';

  // console.time();

  onMount(async () => {
    await new Promise(r => setTimeout(r, 3000));
    for (const d of await navigator.bluetooth.getDevices()) {
      if (d.name === 'Bangle.js c3f3') {
        await bangleConnect();
      }
    }
  });

  let lastCall = 0;
  function throttle<T extends (...args: any[]) => void>(fn: T, interval: number): void {
    const currentTime = Date.now();
    if (currentTime - lastCall >= interval) {
      lastCall = currentTime;
      fn(...arguments);
    }
  }

  let kmbLeft: any; // F9
  let kmbRight: any; // JPY9
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
    // kmbRight.cmdMaxTorque(0.15);
    kmbRight.cmdMaxTorque(50);
    kmbRight.cmdMaxSpeed(100);
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
    // kmbLeft.cmdMaxTorque(0.15);
    kmbLeft.cmdMaxTorque(50);
    kmbLeft.cmdMaxSpeed(100);
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
    const x = 100 - ((coords.x + 100) / 2);
    const y = (coords.y + 100) / 2;
    displayCoords = {x, y};
    // console.warn('x', x, 'y', y)
    let speedLeft = 0;
    let speedRight = 0;
    let tmpY = y - 50;
    let tmpX = x / 3;
    if (tmpY > 0) {
      speedLeft = Math.max(tmpY - (33 - tmpX), 0) * 3;
      speedRight = Math.max(tmpY - tmpX, 0) * 3;
    }
    // console.timeLog();
    throttle(() => {
      console.warn('left', speedLeft, 'right', speedRight)
      kmbLeft.cmdSpeed_rpm(speedLeft);
      kmbRight.cmdSpeed_rpm(speedRight);
      kmbLeft.cmdRunReverse();
      kmbRight.cmdRunForward();
    }, 100);
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
    kmbLeft.cmdSpeed_rpm(speed);
    kmbRight.cmdRunForward();
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
