# Notes...

## Thursday, Nov 16, 2017
- need to include "discover." in front of "table" (namespacing...)
- geth.go doesn't implement the udp layer, it implements a node  that creates a 
server that implements the networking layer
- server starts a UDP Listener. The listener returns a Table (which presumably 
is updated as the listener hears of new UDP connections)
- Table implements the discoverTable interface allowing the basic functionality
used by the dialer
  - Self()
  - Close()
  - Resolve
  - Lookup
  - ReadRandomNodes
- Dialer uses this interface to find/discover new nodes. 
- All outbound UDP connections are run through the table object... 
- Table also keeps track of the whole "Kademlia binning" situation
- Adding a peer adds them first at the TCP layer and then if they aren't known,
will attempt to add them through the UDP layer

Remember: 
- kademlia buckets are tiered, each tier represents how close an address is to 
you
- addresses further away are less like you but will be more likely to have entries


### TODO
- get a findnode request running
  - print and manipulate response
  - generate our own list of nodeid's

EX: 
Given a list of nodes 
```
for node in nodelist
    while new nodes from node
        findnode(more nodes)
    add found new nodes to nodelist
print all unique nodes received from every node
```

## Thursday, Nov 09, 2017
- Two levels of networks
  - UDP level: Kademila Protocol
  - TCP level: Ethereum Peer Protocol
The admin.addPeer(<address>) creates connections over TCP, not UDP.
Therefore, since, the bootnode doesn't have a TCP Listener, we cannot add it
as an Ethereum Peer.

### TODO
- Stage 1: Explore UDP protocol via bootnode -> create the UDP crawler
  - Initialize with one node to start contacting peers
  - Write helper fucntions to do following....
    - Ask peers for neighbors
    - Receive neighbors from peers, keep track of who knows who
    - Dial all new receieved neighbors. 
- Stage 2: connect to peoplve via the TCP connection layer

## Friday, Oct 27, 2017
- Hardcoded our bootnode into the default bootnodes as the only available bootnode
- "enode://a78e6b94394f4f27f0396b3bd48395db621e64357ce1e7747256ff24ee010734388f0f1c25a9c7b2820d1cc231db730705d1be95db1d878c6b6ce4ea5a1e1675@54.204.229.16:30301"
- Remote Procedure Call (RPC): run procedures on other machines or devices 
connected to a network
- Open ports
    - 30301-30303
- Commented out all code pertaining to accounts and ethereum block chain in
$GETH\_HOME/cmd/geth/main.go -> still builds, can acces console. 
- Running into issue where we still can't connect to nodes? Conection refused?

### TODO
End Goal: start up a private newtork of 3 nodes.
- 1 bootnode
- 2 geth nodes
1. Have each geth node add the bootnode as a peer. The bootnode will know about 
everyone in the newtork. The two geth nodes only know about the one bootnode. 
2. From one geth node, request information from the bootnode about what other
nodes it knows about.
3. Use the bootnodes response to add the new (unknown) node as a peer. Now, 
everyone in the network should know about everyone else in the network. 

## Thursday, Oct 26, 2017
- Fixed "syntax error issue" when adding peers via geth console. prompt is
*admin.addPeer('enode://numbers@ipaddress:portnumber')*
- Was able to connect two nodes from AWS together and also connect via outside node
  - Needed to have Alan open up TCP/UDP ports from AWS instance service
  - Use public facing ip address to connect from outside: 54.204.229.161
  - AWS Internal IP address is: 172.30.0.120
  - Noticed that the admin.addPeer cmd makes a dial over TCP but Kademlia operates
  over UDP? How does that work??
  - Connection between instances in the AWS server required both ports to be
  unblocked. I.e. if using port 30301 for one instance and 30302 for another 
  instance, both must be open from the AWS side....
  - Useful Linux Commands
    - nmap ipAddress -p portNumber: allows you to scan from a remote computer
    - netstat -tupn: scan local computer for network interfaces. 
        - t flag: tcp
        - u flag: udp
        - n flag: don't resolve names (no DNS lookup?)
        - p flag: show PID/Program name using each socket
  - Useful Geth Commands
    - admin.addPeer('enode://numbers@ipaddress:portnumber'): manually specifcy 
    a peer to add.
    - debug.verbosity(level): specify what level of debug statements you want 
    printed out. 
    - net.peerCount: number of connected peers

## Monday, Oct 23, 2017
- Forked geth repo to private repo
- Made new branch for crawler code
- Installed go1.9.1 in /usr/local/go/bin/go
- Updated Makefile to be able to run "make bootnode" to build bootnode code. 
- Commented out all bootnodes in params/bootnodes.go

### TODO
1. Copy the go-ethereum directory so we have an unaltered version to run the 
boot node from
*just pull down another clone of the repo*
2. In both directories comment out the boot node addresses so neither nodes are 
able to connect to anyone else
—> note: if we don’t do this and either connect to some outside node, then 
that node will save our ip address and port # and will try to connect to us even 
if we erase the boot node info later on. If this happens we have to switch which 
port we use so that no one can connect to us.
*looking for this now... they seem to have removed cmd/utils/bootnode.go... trying 
to find where the hardcoded values have been moved to. 
I also don't think there's an issue with our bootnode attempting to connect to 
the other nodes since it only runs the UDP listener?*
*update: located in $GETH_HOME/params/bootnode.go*
3. Start running the bootnode
4. On another screen start the geth node as a console and using a different port 
than the boot node (boot node uses port 30301). we can run this with the geth 
console “build/bin/geth —port 30305 console”
5. The first line the boot node outputs is it’s node address (without the ip address). 
Give geth this address but put the ip address where you see [::]. You can either 
hardcode this address into the boot node list or tell the console to connect to 
this peer using the following command: 
*admin.addPeer('enode://numbers@ipaddress:portnumber')*
6. Check if we have any peers by typing net.peerCount; into the console 
7. Can also see peer info with admin.peers;

Other console commands: https://ethereum.gitbooks.io/frontier-guide/content/web3.html look at sections 4.2, 4.3, 4.4
