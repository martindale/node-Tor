node-Tor
===

Javascript implementation of a Tor (or Tor like) anonymizer network (The Onion Router https://www.torproject.org/)

For a quick look, see the demo video on [Peersm] (http://www.peersm.com)

And try:

[Peersm app] (http://peersm.com/peersm)

The minified code for browsers is in the min directory.

You can install:

* [Peersm client](https://github.com/Ayms/node-Tor/tree/master/install)
* [node-Tor Bridge WebSocket server](https://github.com/Ayms/node-Tor/tree/master/install)

node-Tor nodes and bridges are live here:

* [ORDB1](https://atlas.torproject.org/#details/E0671CF9CB593F27CD389CD4DD819BF9448EA834)
* [ORDB2](https://atlas.torproject.org/#details/2679B51C906158F3DF4C59AFD73E2B1FDA6535E1)
* [ORDB3](https://atlas.torproject.org/#details/179B10784BF8955C73313CCB195904AE133E5F53)

Example of implementations:

* Peersm (http://www.peersm.com) : Anonymous P2P serverless network inside browsers, no installation, encrypted and untrackable

* iAnonym :Anonymity into your browser everywhere from any device, see https://www.github.com/Ayms/iAnonym and http://www.ianonym.com
 
## Presentation:

This is an unofficial and extended implementation of the Tor (or Tor like) protocol (Onion Proxy and Onion Router) which anonymizes communications via the Tor (or Tor like) network. This allows to simply connect to the Tor (or Tor like) network and use it, as well as creating and adding nodes into the network, creating complementary and/or parallel networks, implementing completely, partially or not the Tor protocol or a derived one, using completely, partially or not the Tor network, it can be used to create separated Tor like networks.

There are numerous possibilities of uses for node-Tor

**The most challenging goals are now to put the OP and the OR inside the browsers.**

**This is done, see the 3 phases of [Peersm project](http://www.peersm.com) to achieve this.**

## License:

Only the initial code in the lib directory is under the MIT license.

The complete minified versions are subject to the following modified MIT license for now (which removes the rights to modify, merge, sublicense, and sell):

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, publish, and/or distribute copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

## Anonymous serverless P2P inside browsers - Peersm specs

#### Architecture (with servers):

									--- Node2 --- Node3 --- ORDB1
									--- Node5 --- Node6 --- ORDB2
	A (the peer)---	Node/Bridge(ws)	...
									--- Nodey --- Nodez --- ORDBw

			----- ORDB2
	ORDB1	....
			----- ORDBn

#### Final Architecture (serverless for P2P / WS Bridges for direct download):

													--- A1(Peer + Node + ORDB)
	A(Peer + Node + ORDB)	(webRTC + Tor protocol)	...
			|										--- Z1(Peer + Node + ORDB)
			ws (direct download)
			|
			Bridge 	--- Nodea --- Nodeb --- Web site
					...
					--- Nodey --- Nodez --- Web site

Each peer is implementing the Tor protocol (Onion proxy and Onion router) and the ORDB function.

Option 1:

Each peer generates a public/private key, its fingerprint (or ID) is the hash of the DER format of its public key. In what follows 'modulus' is the modulus of the public key.

Option 2:

Each peer generates its ID (160 bits). In what follows 'modulus' does not apply and CREATE is a CREATE_FAST.

Peers are implementing a Kadmelia DHT using their IDs (160 bits), each routing table is composed of 160 buckets of 8 peers max where for bucket j 2^j <=distance(peer,other peer)< 2^(j+1)

If a peer is new (A), it can know how to connect to other peers asking to some servers (the WebSocket bridges used for direct download) that know about the peers.

If the servers are blocked, the peer introduction can be performed by other means : mirror servers or social networks, A just needs to know about one peer first.

Some facilitators running as background processes are doing the same than browsers in order to keep some peers alive and to share files if the peers close their browsers. They can run on PC, Mac, servers and ADSL boxes/routers.

A connects to one of them (CREATE) and sends a FIND_NODE [ID, modulus], it receives up to 8 peers [ID,IP,port,modulus] closest to it. Then it does this (CREATE + FIND_NODE) to closer and closer nodes until it cannot find any closer or until it has at least 6 circuits. When A has 6 circuits it continues to discover the peers the same way just sending a FIND_NODE message.

Each peer connected to A adds A in its routing table (Note: if CREATE is used A must send its fingerprint and modulus too)

The first 5 peers will act as the ORDBs.

The peers can leave the network without telling the others (the peer closes his browser for example), so peers are testing the peers they know with a PING every 15mn (question: how many peers in average in bittorrent routing tables?). They associate to each peer its live time and sort the bucket from the older to the newer, if the bucket is full no new peer can be added.

If a peers disconnects from A, A will establish a new circuit (CREATE) with a peer randomely chosen taking the first one of the selected bucket.

A sends to the ORDBs precisely what it has: db_info 'abcd',N,nb,size,type --> I have chunks N (0 if A has all the chunks) to N+nb of hash_name 'abcd' whose total size is size (0 for a continuous stream) and type MIME-type.

The list is maintained by OR_files['abcd'][N] variable and OR_stream['abcd'][N] for a continuous stream.

If A does not know the size, the parameters size and type are missing in the request.

If N is 0, OR_files['abcd'][1 to Nb chunks] is populated.

The ORDBs are peers too, so they are connected to other ORDBs, they tell them globally what they know other peers have: 'abcd',size,type, the list is maintained by OR_ORDB['abcd'] variable.

A stores the received chunks every block and advertises the ORDBs for each chunk.

A advertises the ORDBs of what they have when a file is uploaded too.

Each time A has a new hash_name 'abcd' it sends a STORE message ['abcd',ID,IP,port,modulus] to the closest node from the hash_name.

Then the closest node sends the same STORE message to the closest node it knows from the hash_name.

Chunk size : 15936 B (32x498 B, payload of Tor protocol cells of 512 B).

Or for WebRTC :

Chunk size : 996 B (2x498 B, < payload of IP, UDP, DTLS, and SCTP protocols ~1150 B - unreliable mode)

Window size: 1035840 B - 65 blocks

A requests 'abcd' :

* A selects 5 ORDBs among the (at least) 6 he is connected to.

* GET [hash_name][Chunk nb][Nb of chunks][Counter] --> 'abcd' N n 0

* 5 GET on 5 circuits : GET1 1 (1-13), GET2 2 (14-26), GET3 3 (27-39),GET4 4 (40-52),GET5 5 (53-65)

* If the size of the file is less than 65 blocks, the ORDBs close the useless requests.

* The ORDB receives the request:

	* If the counter is equal to 5, send db_end (to avoid loops between ORDBs)

		* For m in N to N+n

			* The ORDB checks OR_files['abcd'][m] if it exists, the result is an array of [circ,size,type]
			
				* if the result exists, the ORDB chooses the first one that has a valid circuit (and remove from the lists those that are not valid) and sends the request, the circuit is removed from the list and put at the end.

				* if the result does not exist

					* if chunk nb is 0, the ORDB checks OR_Stream['abcd'], the result is an array of chunks indexes.

						* if the result exists, the ORDBs chooses the index M of number of elements of the result minus 4 times the window size, the result is an array of [circ,type]

							* The ORDB chooses the first one that has a valid circuit and send the request 'abcd' N 0, A will know N in the db_data answer, the ORDB removes the first from the list and put it at the end.

					* if chunk nb is not 0

						* the ORDB checks OR_Stream['abcd'][chunk nb], the result is an array of [circ,type]

							* if the result exists, the ORDB chooses the first one that has a valid circuit and send the request 'abcd' N 0, the ORDB removes the first from the list and put it at the end.

							* if the result does not exist

								* the ORDB checks OR_ORDB, the result is an array of circ

									* if one corresponds, the ORDB chooses the first one that has a valid circuit and sends the request with the counter incremented, remove it from the list and put it at the end.

									* if no result, the ORDB sends a FIND_VALUE ['abcd'] to the 4 closest peer from 'abcd'.
										* as soon as it receives a [ID,IP,port,modulus] answer it connects to the node ID (CREATE) and inserts the new circuit in OR_ORDB['abcd']
										* if the answer is a list of nodes (8 max), these are nodes closest from 'abcd' for the queried node, it continues to send FIND_VALUE['abcd'] to these nodes and implement the same process on reply.

									The reason to do this is to avoid that the download is performed only from the first peer discovered that has the value.

* Note: if for example only one peer has the requested chunks, all the requests will finally end up to him since the ORDBs will send the requests to one of the ORDBs he is connected to.

* A computes tm for every GETm, the time between the request (db_query) and the answer (db_data). Example: 250ms so 31250 B if rate of 1 Mbps, 2 blocks.

* A computes the effective rate for each GETm.

* A wait for the two first GET to end and sends next request on the circuit that showed the best rate, then next one on the second that has the best rate and idem for each finished requests.

* A computes now for each requests sent when he must send a new GET using tm and the effective rate (for example A will compute that he must send a new GET after having received the 10th block)

* It's a bit approximative since the ORDB is rotating the peers by putting them at the end of the lists each time they are used, we suppose that the delay is more related to the connexion between A and the ORDBS.

* If the value is superior to 13 blocks, A sends a new GET after the 10th block.

* And so on.

* If a circuit has a too slow rate compared to others (slow node in the path), it is destroyed and replaced by one of the circuits not used, a new circuit is established.

* If a GET does not end it is resumed from where it was.

#### Handling the lists in the ORDBs:

OR_files['abcd'][N] an array of : [circ,size,type] where circ is a circuit with a peer, N the chunk number, size the total size, type the MIME-type of the file.

OR_files is used for files or finished streaming.

OR_streams['abcd'][N] an array of : [circ,type] where circ is a circuit with a peer, size the total size, type the MIME-type of the file.

OR_streams is used for continuous streaming

OR_ORDB['abcd'] an array of : [circ] where circ is a circuit with an ORDB

If 'abcd' is a continuous streaming, peers and ORDBs periodically remove chunks older than 4 times the window size (4143360 B)

The peers do not advertise the ORDBs of the removed chunks and the ORDBs do not update the lists if circuits break, this is to avoid to continuously sort the lists.

The lists are always manipulated as the same objects, no copy/duplication/clone

#### Streaming:

Add a stream button.

Continuous Streaming:

* Connection to the stream: http://mytv.fr hash_name efgh

* Direct download if nobody has chunks for efgh.

* A saves chunks from 0.

* ...

* TODO: how to correlate chunks Nbs with the source if we must reconnect to it?

#### Messages format:

DB_QUERY [hash_name length 1B][hash_name][Chunk nb length 1B][Chunk nb][nb chunks length 1B][nb chunks][Counter 0 to 5 1B][file info optional 1B value 1]

DB_CONNECTED removed

DB_DATA
* answer to file_info 1:[size length 1B][file size][type length 1B][MIME-type][chunk nb length 1B][chunk nb][end 0 or 1 1B][data]
* answer to no file_info: [end 0 or 1 1B][chunk nb length 1B][chunk nb][data]

* the end field is used in case the requester does not know the total size of the file (unlikely but it can happen)

DB_INFO
	[hash_name length][hash_name][chunk length][chunk nb][nb of chunks length][nb of chunks][size length][size][type length][type]

DB_INFO_ORDB
	[hash_name length][hash_name][size length][size][type length][type]

DB_END
	[Reason 1B]
		0 UNAVAILABLE
		1 FINISHED (aborted by requesting party)
		2 DESTROYED (destroyed by serving party)
		3 DO_NOT_RETRY

## Tests : 

See the demo video on [Peersm] (http://www.peersm.com), the first release is available.

## Install (initial version in lib directory) :

Install node.js on supported platforms : Unix, Windows, MacOS
	
Then as usual :

	npm install node-Tor

or

    git clone http://github.com/Ayms/node-Tor.git
    cd node-Tor
    npm link
	
If you encounter installation problems, you might look at :

	https://github.com/joyent/node/issues/3574 (openssl)
	https://github.com/joyent/node/issues/3504 (python)
	https://github.com/joyent/node/issues/3516 (node.js)

To launch it, you need to be in the lib directory (some small inconvenient that will be fixed) :

	node node-tor.js

## node-Tor Goals and possible Future :

The intent of this project is to provide Tor mechanisms in a web language, so it might open the Tor (or Tor like) network to web languages interfaces.

It is easy to install and to scale, does not depend on specific applications and can interact with js modules, then it is possible to easily build web/js applications on top of it (chat, etc).

node-Tor's nodes can be used to create complementary and/or parallel networks, implementing completely, partially or not the Tor protocol or a derived one, using completely, partially or not the Tor network, it can be used to create separated Tor like networks.
	
## Related projects :

* [Ayms/iAnonym](https://github.com/Ayms/iAnonym)
* [Interception Detector](http://www.ianonym.com/intercept.html)
* [Ayms/abstract-tls](https://github.com/Ayms/abstract-tls)
* [Ayms/websocket](https://github.com/Ayms/websocket)
* [Ayms/node-typedarray](https://github.com/Ayms/node-typedarray)

node-Tor can advantageously be coupled with :

* [Ayms/node-dom](https://github.com/Ayms/node-dom)
* [Ayms/node-bot](https://github.com/Ayms/node-bot)
* [Ayms/node-gadgets](https://github.com/Ayms/node-gadgets)

## Support/Sponsors :

If you like this project you can contact us and/or possibly donate : contact at peersm.com or via PayPal.

## Some words :

The disparition of Aaron Swartz begining of 2013 was a shock for us as for everybody. We did not know each other but exchanged a few emails where he suggested briefly just "to implement all of Tor in JavaScript" while our intent at that time was only to access the network using server side javascript. Apparently Aaron meant to put it inside the browser recognizing a kind of technical challenge. With this idea in mind we did node-Tor and came up with iAnonym and Peersm for the browser implementation, Aaron was aware of part of the result, hopefully this might help serving the causes he defended that we support too.




