<script lang='ts'>
  // TODO: https://askubuntu.com/a/1123260
  import {browser} from '$app/env';
  import './../KMConnectorBrowser.js';

  let KMB1: any;
  let KMB2: any;
  let speed = 30;

  if (browser) {  // client-only code here
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

<div class='motor'>
  <button on:click={KMB1.connect()}>connect1</button>
  <button on:click={onMove1}>move1</button>
  <button on:click={stop1}>stop1</button>
  <button on:click={disconnect1}>disConnect1</button>
</div>

<div class='motor'>
  <button on:click={KMB2.connect()}>connect2</button>
  <button on:click={onMove2}>move2</button>
  <button on:click={stop2}>stop2</button>
  <button on:click={disconnect2}>disConnect2</button>
</div>

<label>
  Speed:
  <input type='range' bind:value={speed} on:change={onChange} min='25' max='50'>
</label>

<style>
  .motor {width: 201px; display: inline-block}
  button { width: 200px; height: 200px; }
</style>
