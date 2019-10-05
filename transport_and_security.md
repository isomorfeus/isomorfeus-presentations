Transcipt of Slack conversation:
Jan Biedermann 09:01 Uhr
So lets look at isomorfeus transport security and security for rpc and drb, maybe we can innovate something.
(ill put my mumblings from here in a doc afterwards)
First of all, for isomorfeus being a isomorphic system, security is absolutely critical and is a top concern and i implemented it to the best of my knowledge. I found absolutely horrible remote code execution problems in other isomorphic systems already. I learned from them :zwinkern:
The transport -> execution model of isomorfeus has several layers or levels security.

First layer is the transport itself. For authenticated access isomorfeus requires a secure socket.

Next comes the transport protocol, or how information is transferred over the wire. Isomorfeus uses JSON. For security i would have preferred net strings https://cr.yp.to/proto/netstrings.txt
But for interoperability with the world i chose JSON.
(If some day wants to have a super secure version of Isomorfeus, i will be happy to implement netstrings)
(In fact, in the beginning a had a version with netstrings)

(to make bare drb secure, the first things would be to use a secure channel and use netstrings for serialized transport)
The JSON of isomorfeus is "used" to form a tree of commands and data, representing the next layers of security.
As a client may have several entities requesting something from the server, at the top of the tree is a identification of the entity requesting the information:
```
RequestAgent with id
```
Server side the same, there are many possbile entities handling requests, LucidHandler, for Data, for Operations etc. So we have:
```
RequestAgent with id
|
Handler
```
RequestAgent: https://github.com/isomorfeus/isomorfeus-framework/blob/master/ruby/isomorfeus-transport/lib/isomorfeus/transport/request_agent.rb
Handler: https://github.com/isomorfeus/isomorfeus-framework/blob/master/ruby/isomorfeus-transport/lib/lucid_handler/mixin.rb
Before Isomorfeus calls into the handler, the first check applied is, if the Handler is actually a valid Handler. Only if that succeeds, the next level is available. A Handler than may provide to many things, may it be different Operation Classes, Object, methods, whatever. If the Handler is valid, control is passed on to it, along with the remaining request information. (bearbeitet) 
The Handler then checks if the requested resource is valid. Lets use Operation as example.
```
RequestAgent with id
|
valid? Handler
|
valid? Operation
```
https://github.com/isomorfeus/isomorfeus-framework/blob/41c2f0d618644c42801f699b01d9e0fefdb405a2/ruby/isomorfeus-operation/lib/isomorfeus/operation/handler/operation_handler.rb#L10 (bearbeitet) 
Next, for operations, its clear that the requested calls is to :promise_run, now Policy comes in to play.
https://github.com/isomorfeus/isomorfeus-framework/blob/41c2f0d618644c42801f699b01d9e0fefdb405a2/ruby/isomorfeus-operation/lib/isomorfeus/operation/handler/operation_handler.rb#L17 (bearbeitet) 
```
RequestAgent with id
|
valid? Handler
|
valid? Operation
|
authorized? :promise_run with props
|
+--->  actual execution
```
The user information is taken from the connection, as we have sockets, everything is bound to the long running connection.
Other Handlers in the system may divert from that and apply more or less checks, depending on purpose.
