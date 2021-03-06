---
layout: real-time-plotting
nav_order: 70
title: Web Bluetooth Accelerometer Graphing
---

<html>

<head>
  <title>Web Bluetooth Accelerometer Plotter</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.9.0/p5.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.9.0/addons/p5.dom.js"></script>
  <script src="https://unpkg.com/p5ble@0.0.5/dist/p5.ble.js"></script>

  <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/dygraph/2.1.0/dygraph.min.js"></script>
  <!-- <script src="smooth-plotter.js"></script> -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/dygraph/2.1.0/dygraph.min.css" />
  <meta name="viewport" content="width=device-width,height=device-height,initial-scale=1.0" />
  <link href="https://fonts.googleapis.com/css?family=Ubuntu&display=swap" rel="stylesheet">


</head>
<style>
  h2, h3, p {
    font-family: 'Ubuntu', sans-serif;
  }
</style>


<body>
  <h2 style="text-align: center">Web Bluetooth Accelerometer Plotter</h2>
  <div>
    <ol>
      <li>Upload <a href="https://github.com/armsp/nano-33-ble-gen/tree/master/bluetooth/real_time_plotting/accelerometer_xyz_plotting">this</a> sketch to your Nano 33 BLE</li>
      <li>Open Serial Monitor (it's is important otherwise the data transfer won't take place)</li>
      <li>Click the "Connect" button and follow the prompt</li>
    </ol>
  </div>
  <h3 style="text-align: center; color: red;" id="compatiblity"></h3>
  <script>
    if ("bluetooth" in navigator) {
      console.log("Supports Web Bluetooth");
      // else the browser doesn't support bluetooth
    } else {
      console.log("Browser doesn't support Web Bluetooth");
      alert("WARNING: This browser doesn't support Web Bluetooth. Try using Chrome.");
      document.getElementById("compatiblity").innerHTML = "Your browser doesn't support Web Bluetooth. Try using Chrome.";
    }
  </script>
  <div id="div_g" style="width:150vh; height:40vh; margin: auto;"></div>
  <script>

    const serviceUuid = "19b10000-e8f2-537e-4f6c-d104768a1214";
    //const serviceUuid = "180f0000-0000-0000-0000-000000000000";
    let myCharacteristic;
    let myValue = 0;
    let myBLE;
    var data = [];
    var g;
    var dataPoints = 100;
    //var plotFlag = true;

    //Graphing
    $(document).ready(function () {
      //var data = [];
      var t = new Date();
      for (var i = dataPoints; i >= 0; i--) {
        var x = new Date(t.getTime() - i * 1000);
        data.push([x, NaN, NaN, NaN]);
      }

      g = new Dygraph(document.getElementById("div_g"), data,
        {
          //color: 'red',
          strokeWidth: 2,
          //rollPeriod: 10,
          drawPoints: true,
          //showRoller: true,
          valueRange: [-6, 6],
          labels: ['Time', 'X', 'Y', 'Z'],
          colors: ['#3B3B98', '#ff5e57', '#1dd1a1'],
          fillGraph: true,
          pointSize: 3,
          stepPlot: false,
          ylabel: "X, Y, Z"
          //pixelsPerLabel: 10,
        });
    });

    function plot(ax, ay, az) {
      // if (plotFlag === true){
      //   g.updateOptions({'file': []});
      //   var t = new Date();
      //   for (var i = dataPoints; i >= 0; i--) {
      //     var x = new Date(t.getTime() - i * 1000);
      //     data.push([x, NaN, NaN, NaN]);
      //   }
      //   g.updateOptions({'file': data});
      //   plotFlag = false;
      // }
      //My test
      var d = new Date();  // current time
      var x = ax;
      var y = ay;
      var z = az;
      data.shift();
      // if (dataPoints>0){
      //   dataPoints-=1;
      //   data.shift();
      //   data.push([d, x, y, z]);
      // }
      // else{
      //   data.push([d, x, y, z]);
      // }
      data.push([d, x, y, z]);
      g.updateOptions({ 'file': data });

    }
    // Bluetooth

    function setup() {
      // Create a p5ble class
      myBLE = new p5ble();

      createCanvas(300, 200);
      textSize(20);
      textAlign(CENTER, CENTER);

      // Create a 'Connect' button
      const connectButton = createButton('Connect')
      connectButton.mousePressed(connectToBle);
    }

    function connectToBle() {
      // Connect to a device by passing the service UUID
      myBLE.connect(serviceUuid, gotCharacteristics);
    }

    // A function that will be called once got characteristics
    function gotCharacteristics(error, characteristics) {
      if (error) console.log('error: ', error);
      console.log('characteristics: ', characteristics);
      myCharacteristic = characteristics[0];
      // Read the value of the first characteristic
      myBLE.read(myCharacteristic, 'string', gotValue);
      //plot(myCharacteristic);
    }

    // A function that will be called once got values
    function gotValue(error, value) {
      if (error) console.log('error: ', error);
      console.log('value: ', value);
      myValue = value;
      var acceleration = myValue.split('|');
      console.log('Acceleration: ', acceleration);
      plot(parseFloat(acceleration[0]), parseFloat(acceleration[1]), parseFloat(acceleration[2]));
      // After getting a value, call p5ble.read() again to get the value again
      myBLE.read(myCharacteristic, 'string', gotValue);
    }

    function draw() {
      background(250);
      text(myValue, 150, 100);
    }
  </script>

</body>

</html>