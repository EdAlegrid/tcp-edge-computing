# node-edge

#### A very lightweight brokerless TCP communication library for edge computing. It has a simple API for pub/sub and client/server communication model. Ideal for iot and m2m applications.

<br>

For a quick tour, we will setup three endpoint devices communicating with each other through a local area network using a publish/subsribe and a client/server communication pattern. 

![](assets/m2m-edge.svg)

## Edge Server 1 Setup

### 1. Create a project directory and install node-edge.
```js
$ npm install node-edge
```
### 2. Save the code below as edge.js in your project directory.
```js
const edge = require('node-edge')

/* simulated voltage data source */
function voltageSource(){
  return 50 + Math.floor(Math.random() * 10)
}

/***
 * tcp edge server 1
 */
 
// you may need to open this port from your endpoint    
let port = 8125, host = '192.168.0.113'  // use the actual ip of your endpoint
edge.createServer(port, host, (server) => {
// or
// edge.createServer({ port:port, host:host }, (server) => {
// edge.createServer(port, (server) => { // localhost
  console.log('tcp server 1 :', host, port)
  
  // publish a data using the topic 'edge-voltage'
  server.publish('edge-voltage', (data) => {
     // by default, data value is polled every 5 secs for changes 
     let vs = voltageSource()
     data.send(vs)
  })
  
  // set a data source using the topic 'current-voltage' for client/server communication model
  server.dataSource('current-voltage', (data) => {
     let ts = voltageSource()
     let pl = null
     if(data.payload === 'current'){
        pl = JSON.stringify({method:'sendData', value:ts})
     }
     else{ 
        pl = JSON.stringify({method:'getData', value:ts})
     }
     data.send(pl)
  })
})

```
### 3. Start your application.
```js
$ node server.js
```
<br>

## Edge Server 2 Setup

### 1. Create a project directory and install node-edge.
```js
$ npm install node-edge
```
### 2. Save the code below as edge.js in your project directory.
```js
const edge = require('node-edge')

/* simulated temperature data source */
function tempSource(){
  return 20 + Math.floor(Math.random() * 4)
}

/***
 * tcp edge server 2
 */

// you may need to open this port from your endpoint
let port = 8125, host = '192.168.0.142'  // use the actual ip of your endpoint

edge.createServer({ port:port, host:host }, (server) => {
  console.log('tcp server 2 :', host, port)
  
  // publish a data using the topic 'edge-temperature'
  // data value is polled every 3 secs as set by data.polling
  server.publish('edge-temperature', (data) => {
     data.polling = 3000 	// set polling interval as 3000 msec or 3 secs instead of 5 secs default
     let ts = tempSource()
     data.send(ts)
  })
  
  // set a data source using the topic 'current-temperature' for client/server communication model
  server.dataSource('current-temperature', (data) => {
     let ts = tempSource()
     let pl = null
     if(data.payload === 'current'){
        pl = JSON.stringify({method:'sendData', payload:data.payload, value:ts})
     }
     else{ 
        pl = JSON.stringify({method:'getData', value:ts})
     }
     data.send(pl)
  })
})

```
### 3. Start your application.
```js
$ node edge.js
```

<br>

## Edge Client Setup

### 1. Create a project directory and install node-edge.
```js
$ npm install node-edge
```
### 2. Save the code below as client.js in your project directory.
```js
const edge = require('node-edge') 

/***
 * tcp edge clients ec1 and ec2
 *
 * communication with edge servers is through a private local network
 */
 
let ec1 = new edge.client(8125, '192.168.0.113')

ec1.getResources((data) => {
  console.log('edge-server 1 resources', data)
})

ec1.subscribe('edge-voltage', (data) => {
  console.log('edge-server 1 edge-voltage', data)
})

ec1.getData('current-voltage', (data) => {
  console.log('edge-server 1 current-voltage', data)
})

/*let ec2 = new edge.client(8125, '192.168.0.142')

// access the data from edge server 2 using the 'edge-temperature' topic
ec2.subscribe('edge-temperature', (data) => {
  console.log('edge server 2 edge-temperature', data)
})

ec2.sendData('current-temperature', 'current', (data) => {
  console.log('edge server 2 current-temperatue', data)
})*/

// or access client methods from a callback

let ec2 = new edge.connect(8125, '192.168.0.142', (client) => {

  client.getResources((data) => {
    console.log('edge-server 2 resources', data)
  })

  client.subscribe('edge-temperature', (data) => {
    console.log('edge-server 2 edge-temperature', data)
  })

  client.sendData('current-temperature', 'current', (data) => {
  // or
  // client.sendData({topic:'current-temperature', payload:'current'}, (data) => {
    console.log('edge-server 2 current-temperatue', data)
  })
})

```
### 3. Start your application.
```js
$ node client.js
```

### 4. The expected output should be similar as shown below.
```js
$
edge-server 1 resources [
  { topic: 'edge-voltage', type: 'publish' },
  { topic: 'current-voltage', type: 'dataSource' }
]
edge-server 1 edge-voltage 55
edge-server 1 current-voltage { method: 'getData', value: 50 }
edge-server 2 resources [
  { topic: 'edge-temperature', type: 'publish' },
  { topic: 'current-temperature', type: 'dataSource' }
]
edge-server 2 edge-temperature 21
edge-server 2 current-temperatue { method: 'sendData', payload: 'current', value: 23 }
edge-server 2 edge-temperature 20
edge-server 1 edge-voltage 58
edge-server 2 edge-temperature 23


```

