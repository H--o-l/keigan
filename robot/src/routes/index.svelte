
<script lang='ts'>
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js';
  import './../puck.js';
  import {onMount} from 'svelte';

  // console.time();
  let textDisplay = '';

  onMount(async () => {
    try {
      await new Promise(r => setTimeout(r, 3000));
        textDisplay += (await navigator.bluetooth.getDevices()).length;

      for (const d of await navigator.bluetooth.getDevices()) {
        textDisplay += d.name;
        if (d.name === 'Bangle.js c3f3') {
          // await bangleConnect();
        } else if (d.name === 'KM-1 JPY9#7C4') {
          await kmbRight.connect(d);
        } else if (d.name === 'KM-1 F97V#9A6') {
          await kmbLeft.connect(d);
        }
        console.warn(d)
      }
    } catch (error) {
      textDisplay += 'error ' + error;
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

  let kmbLeft: any; // F97V#9A6
  let kmbRight: any; // JPY9#7C4
  let speed = 0;
  let displayCoords = {x: 50, y: 50};

  kmbRight = new (window as any).KMMotorOneWebBLE();
  kmbLeft = new (window as any).KMMotorOneWebBLE();
  let rightConnected = false;
  let leftConnected = false;

  kmbRight.on(kmbRight.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    kmbRight.cmdDisableIMUMeasurement();
    kmbRight.cmdDisable();
    kmbRight.cmdCurveType(0);
    kmbRight.cmdAcc(0.03);
    kmbRight.cmdDec(0.03);
    kmbRight.cmdSpeed(0);  // rpm
    // kmbRight.cmdMaxTorque(0.15);
    kmbRight.cmdMaxTorque(1);
    kmbRight.cmdMaxSpeed(100);  // rad/s
    kmbRight.cmdLed(1, 0, 0, 255);
    kmbRight.cmdEnable();
    kmbRight.cmdRunForward();
    rightConnected = true;
  });
  kmbLeft.on(kmbLeft.EVENT_TYPE.init, (kMDeviceInfo: any) => {
    console.log('onInit:'+kMDeviceInfo.name);
    kmbLeft.cmdDisableIMUMeasurement();
    kmbLeft.cmdDisable();
    kmbLeft.cmdCurveType(0);
    kmbLeft.cmdAcc(0.03);
    kmbLeft.cmdDec(0.03);
    kmbLeft.cmdSpeed(0);
    // kmbLeft.cmdMaxTorque(0.15);
    kmbLeft.cmdMaxTorque(1);
    kmbLeft.cmdMaxSpeed(100);
    kmbLeft.cmdLed(1, 0, 0, 255);
    kmbLeft.cmdEnable();
    kmbLeft.cmdRunReverse();
    leftConnected = true;
  });
  kmbRight.on(kmbRight.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
    textDisplay += ' == ' + err;
    rightConnected = false;
  });
  kmbLeft.on(kmbRight.EVENT_TYPE.connectFailure,function(kMDeviceInfo: any, err: any){
    textDisplay += ' == ' + err;
    leftConnected = false;
  });
  kmbRight.on(kmbRight.EVENT_TYPE.disconnect,function(kMDeviceInfo: any, err: any){
    textDisplay += ' == ' + err;
    rightConnected = false;
  });
  kmbLeft.on(kmbRight.EVENT_TYPE.disconnect,function(kMDeviceInfo: any, err: any){
    textDisplay += ' == ' + err;
    leftConnected = false;
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


  // Check if the Permissions API is supported
  if ('permissions' in navigator) {
    // Check the current permission status
    navigator.permissions.query({name: 'accelerometer'}).then(result => {
      if (result.state === 'granted') {
        // Permission is already granted, start reading the acceleration data
        textDisplay += 'granted\n';
      } else if (result.state === 'prompt') {
        // Permission is not yet granted, ask for permission
        navigator.permissions.request({name: 'accelerometer'}).then(result => {
          if (result.state === 'granted') {
            // Permission was granted, start reading the acceleration data
            textDisplay += 'granted\n';
          } else {
            textDisplay += 'Permission to access the Accelerometer was denied\n';
          }
        });
      } else {
        textDisplay += 'Permission to access the Accelerometer is not available\n';
      }
    });
  } else {
    textDisplay += 'Permissions API is not supported\n';
  }


  if (!('Accelerometer' in window)) {
    textDisplay += 'Accelerometer not in window\n'
  }
  try {
    // Create a new Accelerometer object
    const accelerometer = new (window as any).Accelerometer({ frequency: 60 });
    // Start reading the acceleration data
    accelerometer.addEventListener('reading', () => {
      newAcc({x: accelerometer.x * 10, y: accelerometer.y * 10, z: accelerometer.z * 10});
    });
    accelerometer.start();
  } catch (error) {
    textDisplay += 'error ' + error;
  }
  function newAcc(coords: {x: number, y: number, z: number}) {
    const x = 100 - ((coords.x + 100) / 2);
    const y = (coords.y + 100) / 2;
    displayCoords = {x, y: 100 - y};
    // console.warn('x', x, 'y', y)
    let speedLeft = 0;
    let speedRight = 0;
    let tmpY = x - 50;
    let tmpX = y / 3;
    if (tmpY > 0.05) {
      speedLeft = Math.max(tmpY - (33 - tmpX), 0) * 0.8;
      speedRight = Math.max(tmpY - tmpX, 0) * 0.8;
    }
    // console.timeLog();
    throttle(() => {
      // textDisplay = " === " + speedLeft + " === ";
      // textDisplay += speedRight;
      // console.warn('left', speedLeft, 'right', speedRight)
      if (leftConnected && rightConnected) {
        kmbLeft.cmdSpeed(speedLeft);
        kmbLeft.cmdRunReverse();
        kmbRight.cmdSpeed(speedRight);
        kmbRight.cmdRunForward();
      }
    }, 100);
  }
  async function connectRight() {
    await kmbRight.connect();
  }
  async function connectLeft() {
    await kmbLeft.connect();
  }
  function stop() {
    kmbRight.cmdSpeed(0);
    kmbRight.cmdRunForward();
    kmbLeft.cmdSpeed(0);
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
    kmbRight.cmdSpeed(speed);
    kmbLeft.cmdSpeed(speed);
    kmbRight.cmdRunForward();
    kmbLeft.cmdRunReverse();
  }
</script>

<p>{textDisplay}</p>
<p>Left connected {leftConnected}</p>
<p>Right connected {rightConnected}</p>
<div class='global'>
  <div class='column'>
    <button on:click={stop}>stop</button>
    <button on:click={connectRight}>connect JPY9#7C4</button>
    <button on:click={connectLeft}>connect F97V#9A6</button>
    <button on:click={disconnect}>disConnect</button>
    <label style='display: block;'>
      Speed:
      <input type='range' bind:value={speed} on:change={onChange} min='0' max='20'>
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
