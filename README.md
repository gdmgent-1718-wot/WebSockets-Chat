# WebSockets-Chat
WebSockets-Chat | New Media Development | Graphic and Digital Media | Artevelde University College Ghent


In deze demo demonstreren wij het gebruik van websocket samen met nodejs server en expressjs. 




Documentatie
-------------
**stap1**
Maak een folder aan met de volgende files:
	*index.html
	*server.js
	
**Stap2**
Initialiseer npm => 'npm init'

**Stap3**
Voeg de volgende packages toe aan je 'package.json':
		"dependencies": {
			"socket.io": "*",
			"express": "*"
		}
**Stap4**
Initialiseer toegevoegde dependencies => 'npm install'
**Stap5**
maak een nieuwe file aan in de rootmap van het project met de naam "server.js"
**Stap6**
importeer socket.io , http en express 
```
var express = require('express');
var app = express();
var server = require('http').createServer(app);
var io = require('socket.io').listen(server);
```
**Stap7**
maak twee variabelen aan waar we de online users en de connecties gaan bijhouden.
```
users = [];
connections = [];
```
**Stap8**
Server laten luisteren op een bepaalde poort , in dit geval poort 3000 + ophalen index op de root van de server
```
server.listen(process.env.PORT || 3000);
console.log("Server draait");

app.get('/', function(req, res) {
  res.sendFile(__dirname + '/index.html');
});

```
> **Note:**

>- Voor de server te laten lopen =>  ``` node server ```
>- Als je de server laat lopen moet je in de console de logs krijgen

**Stap9**
luisteren naar alle mogelijke serverevents. Alle verdere code zal in deze callback function moeten!!
```
io.sockets.on('connection', function(socket){
});

```
**Note:**

>- voor meer info over de socket.io server api  => https://socket.io/docs/server-api]

**Stap10**
Alle connecties toevoegen aan de array
```
// connect
connections.push(socket);
console.log(' %s connecties', connections.length);
```
**Stap11**
Vanaf het moment iemand disconnect laat je zien hoeveel socket connections er nog actief zijn
```
  // Disconnect
  socket.on('disconnect', function(data){
    users.splice(users.indexOf(socket.username), 1);
    updateUsernames();
    connections.splice(connections.indexOf(socket), 1);
    if(connections.length >= 1){
      console.log('nog %s connecties actief', connections.length);
    }else{
      console.log('geen toestellen meer geconnecteerd');
    }   
  })

```
**Stap12**
 we gaan de data (array users ) gaan verspreiden via het hevent 'get users', 
```
  function updateUsernames() {
    io.sockets.emit('get users', users);
  }  
```
**Stap13**
na het uizenden van de data via emit gaan we de data in de index.html (client) gaan receiven via de sockets.on function
```
   <script>
	$(function(){
		socket.on('get users', function(data) {
			var html = '';
				for(i = 0; i < data.length; i++) {
				html += '<li class="list-group-item">' + data[i] + 
				'</li>';
				}
			$users.html(html);
			});
		});
    </script>
```

**Stap14**
Versturen van een een bericht over socket io naar de server in de server.js
en deze opnieuw  versturen naar de client (want socket io werkt bidirectioneel)
```
//send message
socket.on('send message', function(data){
    console.log(data);
    io.sockets.emit('Bericht', {msg:data, user: socket.username});
  }) 
```
**Stap15**
Toevoegen van een nieuwe user  aan de array users en deze opnieuw emitten naar de client via updateUsernames.
```
  //New User
  socket.on('new user', function(data, callback) {
    callback(true);
    socket.username = data;
    users.push(socket.username);
    updateUsernames();
  });
```
**Stap16**
toevoegen van variabelen aan de client in het script in index.html 
```
 	var socket = io.connect();
	var $messageForm = $('#messageForm');
	var $message = $('#message');
	var $chat = $('#chat');
	var $messageArea = $('#messageArea');
	var $userFormArea = $('#userFormArea');
	var $userForm = $('#userForm');
	var $users = $('#users');
	var $username = $('#username');
```
html layout
```
  <head>
    <title>IO Chat</title>
    <link rel="stylesheet" href="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"></link>
    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script src="/socket.io/socket.io.js"></script>
    <style>
	#messageArea {
	  display: none;
	}
    </style>
  </head>
  <body>

    <div class="container">
      <br />
      <div id="userFormArea" class="row">
	<div class="col-md-12">
	  <form id="userForm">
	    <div class="form-group">
		<label>Enter Username</label>
		<input class="form-control" id="username" />
		<br />
		<input type="submit" class="btn btn-primary" value="Login">
	    </div>
	  </form>
	</div>
      </div>

      <div id="messageArea" class="row">
	<div class="col-md-4">
	  <div class="well">
	    <h3>Online Users</h3>
	    <ul class="list-group" id="users"></ul>
	  </div>
	</div>
	<div class="col-md-8">
	  <div class="chat" id="chat"></div>

	  <form id="messageForm">
	    <div class="form-group">
	      <label>Bericht</label>
	      <textarea class="form-control" id="message"></textarea>
	      <br />
      	      <div id="chat"></div>
      	      <input class="btn btn-primary" type="submit"></input>
	    </div>
    	  </form>
	</div>
      </div>
    </div>
   </body
```
**Stap17**
Als de client zijn bericht invoert zal hij submitten en dus het bericht opnieuw gaan verzenden naar alle socketconnetions vie emit
```
			$messageForm.submit(function(e){
				e.preventDefault();
				console.log('Submitted!');
				socket.emit('send message', $message.val());
				$message.val('');
			});
```
**Stap18**
weergeven van de verzonden berichten + user over de sockets, deze berichten worden toegevoegd aan de div #chat
```
			socket.on('new message', function(data){
				$chat.append('<div class="well"><strong>' + data.user + ': </strong>'+ data.msg + '</div>');
			});
```

**stap19**
Voor men berichten kan sturen moet metn eerst de naam ingeven zodat men weet wie de berichten verstuurt
```
			$userForm.submit(function(e){
				e.preventDefault();
				socket.emit('new user', $username.val(), function(data) {
					if(data) {
						$userFormArea.hide();
						$messageArea.show();
					}
				});
				$username.val('');
			});
```



 Gemaakt door   en **Lode Muylaert** en **Julien Lemoine**
