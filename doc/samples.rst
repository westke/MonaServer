
Samples
##############################

This page gives you several examples of what you can do with MonaServer. Others samples are available in the *clients* directory of MonaServer. To use them you can copy or create a symbolic link of this directory in *www*.
For more information about functionalities see `Server Application`_ page.

The samples below are hosted on a `Raspberry Pi <http://www.raspberrypi.org/>`_ running MonaServer :

.. image:: img/Raspberry_Pi_-_Model_A.jpg
  :width: 300
  :height: 300
  :alt: Raspberry Pi
  :align: center

.. contents:: Table of Contents

Pull Data
******************************

This part describes how to write a simple pull data client/server that supports Websocket, Flash, and Http with RPC-type communication. The real amazing part is the 7 lines of server application which support each protocol :

Server part (lua)
==============================

.. code-block:: lua

  function onConnection(client,...)
    INFO("Connection of a new client "..client.protocol)
    function client:onMethod(data)
      INFO("Reception Message : "..mona:toJSON(data))
      client.writer:writeInvocation("onReception",data.." received")
    end
  end
  
Source code : `PullData/main.lua <http://78.199.204.75/clients/samples/PullData/main.lua>`_

WebSocket client
==============================

.. code-block:: html

  <html>
  <head>
      <title>Pull Websocket/JSON Client Test</title>
      <script type="text/javascript">
        var socket;      
        function createWebSocket() {
          host = "ws://" + window.location.host + "/" + window.location.pathname;
          
          if(window.MozWebSocket)
            window.WebSocket=window.MozWebSocket;
          if(!window.WebSocket) {
            window.document.getElementById("error").value = "Your browser don't support webSocket!";
            return false;
          }
          socket = new WebSocket(host);
          socket.onopen = function() { alert('socket opened'); }
          socket.onclose = function() { alert('socket closed'); }
          socket.onerror = function() { alert('An error occurs'); }
          socket.onmessage = onMessage;
        }
         
        function onMessage(msg){ 
          var response = JSON.parse(msg.data);
          if (response[0] == "onReception") alert(response[1]);
        }
         
        function sendMessage() { 
          var data = ["onMethod", "websocket msg"];
          socket.send(JSON.stringify(data));
        }
         
        createWebSocket();
      </script>
  </head>
  <body>
    <input type="button" value="Send" onclick="sendMessage();" />
  </body>
  <html>

Sample : `PullData/websocket.html <http://78.199.204.75/clients/samples/PullData/websocket.html>`_

Flash client
==============================

.. code-block:: as3

  <?xml version="1.0" encoding="utf-8"?>
  <mx:Application xmlns:fx="http://ns.adobe.com/mxml/2009" 
          xmlns:mx="library://ns.adobe.com/flex/mx" layout="absolute" minWidth="955" minHeight="600">
    <fx:Script>
      <![CDATA[
        import mx.controls.Alert;
        import mx.utils.URLUtil;
        
        private var _netConnection:NetConnection;
        
        // connect button handler
        private function connectAndSend():void {
          
          // Generate dynamic url
          var address:String = "rtmfp://localhost/clients/samples/PullData";
          var url:String = this.loaderInfo.url;
          var domainNPath:Array = url.match(/(:\/\/.+)\//);
          if (URLUtil.getProtocol(url) != "file")
            address = "rtmfp" + domainNPath[1];
          
          // make a new NetConnection and connect
          _netConnection = new NetConnection();
          _netConnection.connect(address);
          _netConnection.client = this;
          // send the request
          _netConnection.call("onMethod", null, "amf message");
        }
        
        public function onReception(result:String):void { Alert.show(result); }
      ]]>
    </fx:Script>
    <mx:Button x="10" y="10" label="Connect and Send" click="connectAndSend()"/>
  </mx:Application>
  
Sample : `PullData/flash.html <http://78.199.204.75/clients/samples/PullData/flash.html>`_

Http JSON-RPC client
======================================

.. code-block:: html

  <html>
  <head>
    <title>HTTP JSON Client Test</title>
    <script type="text/javascript">
      function sendMessage() {
        var xmlhttp = new XMLHttpRequest();
        xmlhttp.open('POST', "", true);
        
        // Manage the response
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            var response = JSON.parse(xmlhttp.response);
            if (response[0] == "onReception") alert(response[1]);
          }
        }
        // Send the POST request
        xmlhttp.setRequestHeader('Content-Type', 'application/json');
        var data = ["onMethod", "http json msg"];
        xmlhttp.send(JSON.stringify(data));
      }
    </script>
  </head>
  <body>
      <input type="button" value="Send" onclick="sendMessage();" />
  </body>
  <html>
  
Sample : `PullData/httpjson.html <http://78.199.204.75/clients/samples/PullData/httpjson.html>`_

Http XML-RPC client
======================================

Mona supports both json and XML-RPC formats, so just replace the response and request with the lines below to have an XML-RPC_ sample :

.. code-block:: js

  // Manage the response
  xmlhttp.onreadystatechange = function () {
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
      var xml = xmlhttp.response;
      alert(xml);
    }
  }
  // Send the POST request
  xmlhttp.setRequestHeader('Content-Type', 'text/xml');
  xmlhttp.send('<?xml version="1.0"?><methodCall>' +
                '<methodName>onMethod</methodName><params>' +
                '<param><value><string>http XML-RPC msg</string></value></param>' +
                '</params></methodCall>');

Sample : `PullData/httpxml.html <http://78.199.204.75/clients/samples/PullData/httpxml.html>`_

Push Data
******************************

This chapter presents an example of push client/server in Websocket and Flash (HTTP support only long polling method). 
Brief description : When the flash client send a message to the server, this message is sent to the websocket client and conversely message from websocket is sent to the other client.

Server part
==============================

.. code-block:: lua

  clientWS = nil
  clientAMF = nil

  function onConnection(client,...)
    
    INFO("Connection of a new client to push (protocol:"..client.protocol..")")
    
    if client.protocol == "WebSocket" then
      clientWS = client
    else
      if client.protocol == "RTMFP" then
        clientAMF = client
      end
    end
    
    function client:onMessage(data)
      INFO("Reception Message : "..mona:toJSON(data))
      
      if client == clientAMF then
        clientWS.writer:writeInvocation("onReception", data)
      else
        clientAMF.writer:writeInvocation("onReception", data)
      end
    end
  end

Source code : `PushData/main.lua <http://78.199.204.75/clients/samples/PushData/main.lua>`_

Websocket part
==============================

.. code-block:: html

  <html>
  <head>
      <title>Push Websocket/JSON Client Test</title>
      <script type="text/javascript">
        var socket;      
        function createWebSocket() {
          host = "ws://" + window.location.host + "/" + window.location.pathname;
          
          if(window.MozWebSocket)
            window.WebSocket=window.MozWebSocket;
          if(!window.WebSocket) {
            window.document.getElementById("error").value = "Your browser don't support webSocket!";
            return false;
          }
          socket = new WebSocket(host);
          socket.onopen = function() { alert('socket opened');}
          socket.onclose = function() { alert('socket closed'); }
          socket.onerror = function() { alert('socket in error'); }
          socket.onmessage = onMessage;
        }
         
        function onMessage(msg){
          var response = JSON.parse(msg.data);
          if (response[0] == "onReception")
            alert(response[1] + " received");
        }
         
        function sendMessage() { socket.send([["message from websocket"]]); }
         
        createWebSocket();
      </script>
  </head>
  <body>
    <input type="button" value="Send" onclick="sendMessage();" />
  </body>
  <html>

Sample : `PushData/websocket.html <http://78.199.204.75/clients/samples/PushData/websocket.html>`_

Flash client
==============================

.. code-block:: as3

  <?xml version="1.0" encoding="utf-8"?>
  <mx:Application xmlns:fx="http://ns.adobe.com/mxml/2009" 
          xmlns:mx="library://ns.adobe.com/flex/mx" layout="absolute" minWidth="955" minHeight="600" updateComplete="init()">
    <fx:Script>
      <![CDATA[
        import mx.controls.Alert;
        import mx.utils.URLUtil;
        
        private var _netConnection:NetConnection;
        
        // connect button handler
        private function init():void {
          
          // Generate dynamic url
          var address:String = "rtmfp://localhost/clients/samples/PushData";
          var url:String = this.loaderInfo.url;
          var domainNPath:Array = url.match(/(:\/\/.+)\//);
          if (URLUtil.getProtocol(url) != "file")
            address = "rtmfp" + domainNPath[1];
          
          // make a new NetConnection and connect
          _netConnection = new NetConnection();
          _netConnection.connect(address);
          _netConnection.client = this;
        }
        
        // send the request
        private function send():void {
          _netConnection.call("onMessage", null, "message from amf");
        }
        
        // Receive a message asynchronously
        public function onReception(result:String):void { Alert.show(result + " received"); }
      ]]>
    </fx:Script>
    <mx:Button x="10" y="10" label="Send" click="send()"/>
  </mx:Application>

Sample : `PushData/flash.html <http://78.199.204.75/clients/samples/PushData/flash.html>`_

Read/Write Files
******************************

For now writing files is only possible by a scripting way, if you need more please contact us (Support_).

Read Files
==============================

To read a file you just have to put it in the *www* directory and access it from HTTP. To have more functionnalities please contact us (Support_).

Communication channels
******************************

Publish live
==============================

Now we are about to create a sample of publication with a flash publisher. For the server part just create a directory “publish” in the root directory. For the publisher use the code below :

.. code-block:: as3

  <?xml version="1.0" encoding="utf-8"?>
  <mx:Application xmlns:fx="http://ns.adobe.com/mxml/2009" 
          xmlns:mx="library://ns.adobe.com/flex/mx" layout="absolute" minWidth="955" minHeight="600" updateComplete="startCam()">
    <fx:Script>
      <![CDATA[
        import mx.controls.Alert;
        import mx.core.FlexGlobals;
        import mx.utils.URLUtil;
        private var _cam:Camera;
        private var _connection:NetConnection;
        private var _outstream:NetStream;
        
        // init camera
        private function startCam():void {
          
          // Generate dynamic url
          var url:String = this.loaderInfo.url;
          var domainNPath:Array = url.match(/(:\/\/.+)\//);
          if (URLUtil.getProtocol(url) != "file")
            address.text = "rtmfp" + domainNPath[1];
          
          _cam = Camera.getCamera();
          player.attachCamera(_cam);
        }
        
        // net status handler for the NetConnection : connect the netstream and publish
        private function onStatus(evt:NetStatusEvent):void { 
          
          status.text = evt.info.code; 
          _outstream = new NetStream(_connection);
          _outstream.addEventListener(NetStatusEvent.NET_STATUS, onStatusOutstream);
          _outstream.attachCamera(_cam);
          _outstream.publish("file");
        }
        
        // net status handler for the NetStream
        private function onStatusOutstream(evt:NetStatusEvent):void {       
          statusOutstream.text = evt.info.code; 
        }
        
        // Connect
        private function send():void {
          
          _connection = new NetConnection();
          _connection.connect(address.text);
          _connection.addEventListener(NetStatusEvent.NET_STATUS, onStatus);
        }
      ]]>
    </fx:Script>  
    <mx:TextInput x="10" y="10" width="400" text="rtmfp://localhost/clients/samples/PublishLive" id="address"/>
    <mx:Button x="450" y="10" label="Send" click="send()"/>
    <mx:Label x="10" y="40" text="Net Status Code: "/>
    <mx:Text x="150" y="40" id="status" width="200"/>
    <mx:Label x="10" y="70" text="OutStream Status Code: "/>
    <mx:Text x="150" y="70" id="statusOutstream" width="200"/>
    <mx:VideoDisplay x="10" y="100" width="320" height="240" id="player"/>
  </mx:Application>


Sample : `PublishLive/flash.html <http://78.199.204.75/clients/samples/PublishLive/flash.html>`_

To play the video you an use a flash player or vlc for example connected to the following url : `rtmp://78.199.204.75/file <rtmp://78.199.204.75/file>`_

If you need support for other type of clients or devices please contact us (Support_).

P2P channel
=====================================

This sample shows P2P file transfert and NetGroup usage over Object Replication functionality. 
Here follows an illustration of the P2P NetGroup :

.. image:: img/NetgroupP2PChannel.png
  :width: 639
  :height: 336
  :align: center
  
1. In this sample the first user can browse and share a file from is filesystem,
2. When a second user arrives he can download the file from the first one,
3. And then each new user download the file from any other user, flash decides by itself who is the nearest peer.

Server part
-------------------------------------

Source below is the lua application. During the test you should take attention to the sender of the file which could be any peer among the providers.

.. code-block:: lua

  peers = {}

  function onConnection(client, name)
    
    INFO("User connected on p2p sharing app : ", name)
    peers[client] = name
    
    function client:onInfoSend(file, index)
      
      INFO("User "..peers[client].." is sending file "..file.." ("..index..")")
    end
    
    function client:onInfoRequest()
      
      INFO("User "..peers[client].." has requested file")
    end
  end

  function onDisconnection(client)
    name = peers[client]

    if name then
      INFO("User disconnecting: "..name)
      peers[client] = nil
    end
  end
  
Source code : `P2PChannel/main.lua <http://78.199.204.75/clients/samples/P2PChannel/main.lua>`_

Flash client
-------------------------------------  
  
And the flash client source is cutted in three files. 
Here is the file *P2PSharedObject.as*, the class file for objects that will be exchanged :

.. code-block:: as3

  package {
    import flash.utils.ByteArray;

    public class P2PSharedObject {
      
      public var fileName:String;
      public var size:Number = 0;
      public var packetLength:uint = 0;
      public var actualFetchIndex:Number = 0;
      public var data:ByteArray;
      public var chunks:Object = new Object();
      
      public function P2PSharedObject(){}
    }
  }
  
Next file is *LocalFileLoader.As*, the class for reading files and chunking them :

.. code-block:: as3

  package {
    import flash.events.Event;
    import flash.events.EventDispatcher;
    import flash.events.IOErrorEvent;
    import flash.events.ProgressEvent;
    import flash.events.SecurityErrorEvent;
    import flash.events.StatusEvent;
    import flash.net.FileReference;
    import flash.utils.ByteArray;
    import mx.controls.Alert;
    
    [Event(name="complete",type="flash.events.Event")]
    [Event(name="status",type="flash.events.StatusEvent")]
    public class LocalFileLoader extends EventDispatcher {
      
      public function LocalFileLoader(){}
     
      private var file:FileReference;
      public var p2pSharedObject:P2PSharedObject;
      public const SIZE_CHUNKS:Number=64000; ///< 64k per chunks
      
      public function browseFileSystem():void {
        
        file = new FileReference();
        file.addEventListener(Event.SELECT, selectHandler);
        file.addEventListener(IOErrorEvent.IO_ERROR, ioErrorHandler);
        file.addEventListener(ProgressEvent.PROGRESS, progressHandler);
        file.addEventListener(SecurityErrorEvent.SECURITY_ERROR, securityErrorHandler)
        file.addEventListener(Event.COMPLETE, completeHandler);
        file.browse();
      }
      
      protected function selectHandler(event:Event):void {
        writeText("fileChosen : " + file.name+" | " + file.size);
        file.load();
      }
      
      protected function ioErrorHandler(event:IOErrorEvent):void {
        Alert.show("ioErrorHandler: " + event);
      }
      
      protected function securityErrorHandler(event:SecurityErrorEvent):void {
        Alert.show("securityError: " + event);
      }
      
      protected function progressHandler(event:ProgressEvent):void {
        var file:FileReference = FileReference(event.target);
        writeText("progressHandler: bytesLoaded=" + event.bytesLoaded + "/" +event.bytesTotal);
        
      }
      
      protected function completeHandler(event:Event):void {
        writeText("completeHandler");
        
        p2pSharedObject = new P2PSharedObject();
        p2pSharedObject.size = file.size;
        p2pSharedObject.data = file.data;
        p2pSharedObject.fileName = file.name;

        p2pSharedObject.chunks = new Object();
        p2pSharedObject.packetLength = 2;
        
        // Write each chunked part of file
        var size:Number = 0;
        while((size = p2pSharedObject.data.bytesAvailable) > 0) {
          
          p2pSharedObject.chunks[p2pSharedObject.packetLength] = new ByteArray();
          if (size >= SIZE_CHUNKS)
            p2pSharedObject.data.readBytes(p2pSharedObject.chunks[p2pSharedObject.packetLength],0,SIZE_CHUNKS);
          else // last bytes
            p2pSharedObject.data.readBytes(p2pSharedObject.chunks[p2pSharedObject.packetLength],0,p2pSharedObject.data.bytesAvailable);
          p2pSharedObject.packetLength += 1;
        }
        p2pSharedObject.chunks[0] = p2pSharedObject.packetLength;
        p2pSharedObject.chunks[1] = p2pSharedObject.fileName;
        
        writeText("packetLength: "+(p2pSharedObject.packetLength));
        dispatchEvent(new Event(Event.COMPLETE));
      }
      
      protected function writeText(str:String):void{
        var e:StatusEvent = new StatusEvent(StatusEvent.STATUS,false,false,"status",str);
        
        dispatchEvent(e);
      }
    }
  }

And the last one is the mxml main file which connect the peer to MonaServer and share/receive file among the peers :

.. code-block:: as3

  <?xml version="1.0" encoding="utf-8"?>
  <mx:Application xmlns:fx="http://ns.adobe.com/mxml/2009" 
          xmlns:mx="library://ns.adobe.com/flex/mx" layout="absolute" minWidth="955" minHeight="600" applicationComplete="getName();">
    <fx:Script>
      <![CDATA[			
        import mx.containers.TitleWindow;
        import mx.controls.Alert;
        import mx.events.CloseEvent;
        import mx.managers.PopUpManager;
        import mx.utils.URLUtil;
        private var _fileLoader:LocalFileLoader;
        private var _netConnection:NetConnection;
        private var _netGroup:NetGroup;
        private var _namePopup:TitleWindow;
        private var _nameUser:String;
        public 	var p2pSharedObject:P2PSharedObject;
        
        // Try to identificate
        private function sendName(event:Event):void {
          
          // Accept only <ENTER> key
          if (event is KeyboardEvent) {
            var eventKey:KeyboardEvent = event as KeyboardEvent;
            if (eventKey.keyCode != 13)
              return;
          }
          
          var userName:TextInput = _namePopup.getChildByName("userName") as TextInput;
          if (userName != null && userName.text!="") {
            
            _nameUser = userName.text;
            PopUpManager.removePopUp(_namePopup);
          }
        }
        
        private function closeNamePopup(event:CloseEvent):void {
          
          PopUpManager.removePopUp(_namePopup);
        }
        
        // net status handler for the NetConnection
        private function getName():void {
          
          // create and configure the Identification Window
          _namePopup = new TitleWindow();
          _namePopup.title = "Please enter your name :";
          _namePopup.showCloseButton = true;
          _namePopup.addEventListener(CloseEvent.CLOSE, closeNamePopup);
          
          // create and configure a Label
          var userName:TextInput = new TextInput();
          userName.text = "User";
          userName.name = "userName";
          _namePopup.addChild(userName);
          // add buttons OK and Cancel
          var btOK:Button = new Button();
          btOK.label = "OK";
          btOK.addEventListener(MouseEvent.CLICK, sendName);
          btOK.addEventListener(KeyboardEvent.KEY_DOWN, sendName);
          _namePopup.addChild(btOK);
          
          // open the Identification Window as a modal popup window
          PopUpManager.addPopUp(_namePopup, this, true);
          PopUpManager.centerPopUp(_namePopup);
          userName.setFocus();
          
          // Generate dynamic url
          var url:String = this.loaderInfo.url;
          var domainNPath:Array = url.match(/(:\/\/.+)\//);
          if (URLUtil.getProtocol(url) != "file")
            address.text = "rtmfp" + domainNPath[1];
        }
        
        private function connect():void{
          _fileLoader = new LocalFileLoader();
          _fileLoader.addEventListener(StatusEvent.STATUS, onStatusLoad);
          _fileLoader.addEventListener(Event.COMPLETE, startSharing);
          
          _netConnection = new NetConnection();
          _netConnection.addEventListener(NetStatusEvent.NET_STATUS, netStatus);
          _netConnection.connect(address.text, _nameUser);
        }
        
        private function onStatusLoad(event:StatusEvent):void{
          writeText("Load : " + event.level);
        }
        
        protected function netStatus(event:NetStatusEvent):void{
          
          switch(event.info.code){
            case "NetConnection.Connect.Success": // Connected to server => NetGroup connection 
              var spec:GroupSpecifier = new GroupSpecifier("myGroup");
              spec.serverChannelEnabled = true;
              spec.objectReplicationEnabled = true;
              
              _netGroup = new NetGroup(_netConnection,spec.groupspecWithAuthorizations());
              _netGroup.addEventListener(NetStatusEvent.NET_STATUS,netStatus);
              
              writeText("Netconnection OK");
              break;
            
            case "NetGroup.Connect.Success": // Connected to group
              _netGroup.replicationStrategy = NetGroupReplicationStrategy.LOWEST_FIRST;
              btStartReceiving.enabled = true;
              btBrowse.enabled = true;
              writeText("NetGroup Connection OK");
              break;
            
            case "NetGroup.Replication.Fetch.Result": // Reception of file
              
              // Share the chunk downloaded
              _netGroup.addHaveObjects(event.info.index,event.info.index);
              p2pSharedObject.chunks[event.info.index] = event.info.object;
              _fileLoader.p2pSharedObject = p2pSharedObject;
              
              // Size
              if(event.info.index == 0){
                p2pSharedObject.packetLength = Number(event.info.object);
                writeText("p2pSharedObject.packetLenght: "+p2pSharedObject.packetLength);
              } 
              // FileName
              else if (event.info.index == 1) {
                p2pSharedObject.fileName = String(event.info.object);
                writeText("p2pSharedObject.fileName: "+p2pSharedObject.fileName);
              }
              // File Reception Complete!
              else if (event.info.index+1 >= p2pSharedObject.packetLength) {
                writeText("Receiving DONE: "+p2pSharedObject.packetLength);
                
                p2pSharedObject.data = new ByteArray();
                for(var i:int = 2;i<p2pSharedObject.packetLength;i++){
                  p2pSharedObject.data.writeBytes(p2pSharedObject.chunks[i]);
                }
                btSave.enabled = true;
                return;
              }
              receiveObject(event.info.index+1);
              
              break;
            
            case "NetGroup.Replication.Request": // File requested
              _netConnection.call("onInfoSend", null, _fileLoader.p2pSharedObject.fileName, event.info.index);
              _netGroup.writeRequestedObject(event.info.requestID, _fileLoader.p2pSharedObject.chunks[event.info.index]);
              break;
            
            case "NetGroup.Replication.Fetch.SendNotify":
              break;
            
            default:
              writeText(event.info.code);
              break;
          }
        }
        
        private function saveFile():void {
          
          var file:FileReference = new FileReference();
          file.save(p2pSharedObject.data, p2pSharedObject.fileName);
        }
        
        // Request one chunked object
        protected function receiveObject(index:Number):void{
          
          _netGroup.addWantObjects(index,index);
          p2pSharedObject.actualFetchIndex = index;
        }
        
        private function startReceiving():void{
          writeText("startReceiving");
          
          p2pSharedObject = new P2PSharedObject();
          p2pSharedObject.chunks = new Object();
          receiveObject(0);
          _netConnection.call("onInfoRequest", null);
        }
        
        private function startSharing(event:Event):void{
          writeText("File loaded, startSharing - " + _fileLoader.p2pSharedObject.packetLength + " chunks");
          
          _netGroup.addHaveObjects(0, _fileLoader.p2pSharedObject.packetLength);
          btStartReceiving.enabled = false;
          btBrowse.enabled = false;
        }
        
        private function writeText(txt:String):void {
          txtHistory.text += txt + "\n";
        }
      ]]>
    </fx:Script>
    <mx:VBox x="10" y="10" height="100%" paddingBottom="10">
      <mx:HBox>
        <mx:TextInput width="400" text="rtmfp://localhost/clients/samples/P2PChannel" id="address"/>
        <mx:Button label="Connect" click="connect()"/>
      </mx:HBox>
      <mx:HBox>
        <mx:Button id="btBrowse" label="Browse and Share" click="_fileLoader.browseFileSystem();" enabled="false"/>
        <mx:Button id="btStartReceiving" label="Receive" click="startReceiving();" enabled="false"/>
        <mx:Button id="btSave" label="Save" click="saveFile();" enabled="false"/>
      </mx:HBox>
      <mx:TextArea id="txtHistory" width="400" height="100%"/>
    </mx:VBox>
  </mx:Application>


Sample : `P2PChannel/index.html <http://78.199.204.75/clients/samples/P2PChannel/index.html>`_

Others
*************************************

Meeting sample
=====================================

The sources are available here: http://www.adobe.com/devnet/flashmediaserver/articles/real-time-collaboration.html

Use only the client part of these sources, and for server side create the file MonaServer/www/meeting/main.lua with the following content:

.. code-block:: lua

  meeters = {}

  function onConnection(client, userName, meeting)
    
    if client.protocol == "RTMFP" or client.protocol == "RTMP" then
      meeter = {}
      meeter.userName = userName
      meeter.meeting = meeting

      INFO("User connected: ", meeter.userName , " meeting: ", meeter.meeting)
      
      sendParticipantUpdate(meeter.meeting)
      meeters[client] = meeter -- Add participant to the list
    end
    
    function client:onRead(file)
      if file == "" and client.protocol == "HTTP" then -- If file empty => return VideoMeeting.html
        return "VideoMeeting.html"
      end
    end
    
    function client:getParticipants(meeting)
      result = {}
      i = 0;
      for cur_client, cur_meeter in pairs(meeters) do
        if (cur_meeter.meeting == meeting) then
          i = i+1;
          if cur_client.id then
            cur_meeter.protocol = 'rtmfp'
          end
          cur_meeter.farID = cur_client.id;    
          result[i] = cur_meeter
        end
      end  
      return result
    end
      
    function client:sendMessage(meeting, from, message)
    
      for cur_client, cur_meeter in pairs(meeters) do
        if (cur_meeter.meeting == meeting) then
          cur_client.writer:writeInvocation("onMessage", from, message)
        end
      end
    end
  end

  function onDisconnection(client)
    meeter = meeters[client]

    if meeter then
      INFO("User disconnecting: "..meeter.userName)
      meeters[client] = nil
      sendParticipantUpdate(meeter.meeting)
    end
  end

  function sendParticipantUpdate(meeting)
    for cur_client, cur_meeter in pairs(meeters) do
      if (cur_meeter.meeting == meeting) then
        cur_client.writer:writeInvocation("participantChanged")
      end
    end
  end

Sample : `Meeting/VideoMeeting.html <http://78.199.204.75/clients/samples/Meeting/VideoMeeting.html>`_

.. _Server Application: ./serverapp.html
.. _Support: ./contacts.html
.. _XML-RPC : http://xmlrpc.scripting.com/spec.html
