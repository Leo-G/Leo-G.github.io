In this article I will describe how you can visualize the real time Memory and CPU usage of a Linux Server using Socket.io and D3.js gauges.

Before we dive into the code, here is a little background on the Technology.

Socket.io
Socket.io is a javascript library that uses Sockets for fast real time communication. With Socket.io you can create Server and  Client side sockets that will listen for events and transmit JSON formatted Data. In this tutorial we will use it for transmitting Linux Server Resource Usage data.

D3.js
D3.js is a client side javascript library that allows you to visualize data with SVG (Scalable Vector Graphics) a text-based image format. With SVG you can specify what an image should look like by writing simple markup code, For example to draw a circle with SVG you can do so with the following code:


<svg width="50" height="50">
    <circle cx="25" cy="25" r="22" fill="blue" stroke="gray" stroke-width="2"/>
</svg>
Example of the same Circle using D3.js


<script>
// Create an SVG object
var svg = d3.select("body")
            .append("svg")
            .attr("width", 500)
            .attr("height", 50);
// bind a circle to it
svg.append("circle")
                      .attr("cx", 250)
.attr("cy", 50)
.attr("r", 40);
</script>
// Index.html
<html>
<body>
</body>
</html>
With D3.js we will visualize the Linux Resource Usage using Google Style Gauges as seen in the image above.

Getting Started
To get started install Node.js and Socket.io on your Linux Server.
Note: These steps are for Ubuntu 14.04 and socket.io 1.4.5


wget -qO- https://deb.nodesource.com/setup_4.x | sudo bash -
sudo apt-get install -y nodejs
npm install --save socket.io

Creating sockets with Socket.io
Socket.io allows you to create sockets with custom events and data. In the example code below I have created  Two custom events "total" and “server”. When a Socket request comes in these events will send the current CPU and Memory usage in Percentage and GB respectively.


var http = require('http').Server();
var io = require('socket.io')(http);
var os = require('os');
var cpu = require('./cpu.js');

// Listen for incoming socket requests on the special “connection” event
io.on('connection', function(socket){
// Log to console when a user is connected
console.log('a user connected');
// Create the “total” event” and send the total memory data
 socket.on('total', function(){
       var data = {'totalMemory':os.totalmem()};
       io.emit('total', data);});
// Create a “server” event and emit real time cpu and memory usage
 socket.on('server', function(){
           cpu().then(function(cpuPercentage) {
                         var data = {'freeMemory':os.freemem(), 'cpu':cpuPercentage};
                         io.emit('server', data);
 });
 });
});
 http.listen(443, function(){
 console.log('listening on *:443');
});
To calculate the current CPU usage Percentage you will need to Subtract the CPU idle time from the Total CPU time for a particular interval and then convert it into a percentage. I have added this code in a separate file called CPU.js and then imported. The code for CPU.js is as follows:

//via https://gist.github.com/bag-man/5570809
// Explanation at https://github.com/Leo-G/DevopsWiki/wiki/How-Linux-CPU-usage-time-and-Percentage-can-be-calculated
var os = require("os");
//Create function to get CPU information
function cpuAverage() {
//Initialise sum of idle and time of cores and fetch CPU info
 var totalIdle = 0, totalTick = 0;
 var cpus = os.cpus();
//Loop through CPU cores
 for(var i = 0, len = cpus.length; i < len; i++) {
//Select CPU core
 var cpu = cpus[i];
//Total up the time in the cores tick
 for(type in cpu.times) {
 totalTick += cpu.times[type];
 }
//Total up the idle time of the core
 totalIdle += cpu.times.idle;
 }
//Return the average Idle and Tick times
 return {idle: totalIdle / cpus.length, total: totalTick / cpus.length};
}
//Grab first CPU Measure
var startMeasure = cpuAverage();
module.exports = function () {
//https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
 // http://stackoverflow.com/questions/24928846/get-return-value-from-settimeout
 var promise = new Promise(function (resolve, reject) {
//Set delay for second Measure
 setTimeout(function() {
//Grab second Measure
 var endMeasure = cpuAverage();
//Calculate the difference in idle and total time between the measures
 var idleDifference = endMeasure.idle - startMeasure.idle;
 var totalDifference = endMeasure.total - startMeasure.total;
//Calculate the average percentage CPU usage
 var percentageCPU = 100 - ~~(100 * idleDifference / totalDifference);
resolve(percentageCPU);
//Output result to console
 // console.log(percentageCPU + "%");
}, 100);
});
return promise;
}
Note: Besides connect, message and disconnect, you can emit any custom events

D3.js Gauges
You will need to use a web framework to serve D3.js and HTML code. For this tutorial I have used Express.js for it's ease of use. You will need to modify the index.js file with the following code after installing express.js.


npm install --save express@4.10.2
// Serving a single web page with Express
// index.js
var express = require('express');
var app = require('express')();
var http = require('http').Server(app);
app.get('/', function(req, res){
 res.sendFile(__dirname + '/index.html');
});
http.listen(5001, function(){
 console.log('listening on *:5001');
});


#Index.html
<html>
<body>
<h2>Linux Resource Monitor</h2>
</body>
</html>

On starting the server with the command “ node index.js” you should see a page with the header “Linux Resource Usage” at http://localhost:5001.

Google Style Gauges
Tomer Doron has already made available a nice example of Google Style Gauges at http://bl.ocks.org/tomerd/149927. I am going to tweak his example to serve real time Linux and Memory usage.

 // main.js
// Initialize a socket
var socket = io();
var gauges = [];
function createGauge(name, label, min, max) {
var config = {
 size: 220,
 label: label,
 min: undefined != min ? min : 0,
 max: undefined != max ? max : 100,
 minorTicks: 5
 }
var range = config.max - config.min;
 config.yellowZones = [{ from: config.min + range*0.75, to: config.min + range*0.9 }];
 config.redZones = [{ from: config.min + range*0.9, to: config.max }];
gauges[name] = new Gauge(name + "GaugeContainer", config);
 gauges[name].render();
 }
function createGauges()
{
 // Get the Total Memory and Create the Memory Gauge
socket.emit('total', {});
socket.on('total', function(data){
createGauge("memory", "Memory", 0, Math.round(data.totalMemory/(1024*1024*1024)));
});
// Create the CPU gauge with the default range
 createGauge("cpu", "CPU");
}
function updateGauges() {
socket.emit('server', {});
socket.on('server', function(data){
var freeMemory = data.freeMemory/(1024*1024*1024);
var usedMemory = 2 - freeMemory.toFixed(2);
console.log(usedMemory);
var cpuPercentage = data.cpu;
for (var key in gauges)
 {
if (key == "memory") {
gauges[key].redraw(usedMemory);
} else {
gauges[key].redraw(cpuPercentage); }
}
})
}
 // Create the Gauges and Update them every 5 seconds
 function initialize()
 {
 createGauges();
setInterval(updateGauges, 5000);
 }

// Index.html
<!DOCTYPE html>
<html>
<head>
    <title>Linux  Resource Monitor</title>
    <link rel='stylesheet' href='http://fonts.googleapis.com/css?family=Play:700,400' type='text/css'>
    <link rel='stylesheet' href='gauge.css'>
    <script src="/socket.io/socket.io.js"></script>
    <script type="text/javascript" src="http://mbostock.github.com/d3/d3.js"></script>
    <script type="text/javascript" src="gauge.js"></script>
    <script type="text/javascript" src="main.js"></script>
</head>
<body onload="initialize()">
<div >
 <span id="memoryGaugeContainer" style="margin:40px 10px;"></span>
 <span id="cpuGaugeContainer" style="margin-top:40px;"></span>
 </div>
</body>
</html>
Refresh the Page and You should see the Memory and CPU gauges.





 

Video of the working app.



Complete code is available at https://github.com/Leo-G/LMON.

In depth guides on Linux Memory and CPU resource calculation are available at:

Troubleshooting Linux Memory Usage

How Linux CPU usage is calculated
Ref :
http://alignedleft.com/tutorials/d3.
http://socket.io/get-started/chat/
http://www.websocket.org/quantum.html
https://github.com/mbostock/d3/wiki/Gallery
