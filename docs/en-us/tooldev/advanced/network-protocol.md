# Network Structure: P2P

NEO uses P2P network structure and TCP/IP protocol for transmission.
There are 2 types of node in the network, ordinary node and consensus node. The former can perform transaction/block broadcasting, receiving and relaying, while the later can furthermore perform block construction.
Neo's network protocol standard is similiar to that of Bitcoin, but varies a lot in block / transaction data structure.

> [!NOTE]
>
> - In NEO network, seed node is not equal to consensus node. It's a kind of ordinary node which provides node list query service to other nodes.
> - NEO network also supports WebSocket connection and supports constructing node in a LAN by UPnP protocol using IGD (Optional).

## Transmission Protocol and Port

| Protocol | Port (Main net) | Port (Test net) |
| --- | --- | --- |
| TCP/IP | 10333 | 20333 |
| WebSocket | 10334 | 20334 |

> [!Note]
>
> Port can be set to any unused one when constructing NEO private chain. Ports of nodes in a private chain can also be different.

## Message

Message's basic format is as follows:

| Type | Name | Description |
| --- | --- | --- |
| uint | Magic | Magic number to avoid network conflict |
| string(12) | Command | String message name (Fill with zeros if shorter than 12 byte) |
| int | Payload length | Payload length |
| uint | Payload Checksum | Payload verification to avoid falsification and transmission errors |
| byte[] | Payload | Message text, varying by message type |

## Command List

| <nobr>Name</nobr> | <nobr>Uniqueness</nobr> | <nobr>High priority</nobr> | <nobr>Reserved</nobr> | Description |
| --- | --- | --- | --- | --- |
| getaddr | 〇 | 〇 |  | Queries address and port numer of other nodes. |
| addr | 〇 |  |  | Answers getaddr message with at most 200 records of succesfully connected node addresses and port numbers. |
| filteradd |  | 〇 |  | Adds data to bloom_filter for SPV wallet. |
| filterclear |  | 〇 |  | Remove bloom_filter for SPV wallet. |
| filterload |  | 〇 |  | Initialize bloom_filter for SPV wallet. |
| getblocks | 〇 |  |  | Specify the start and end hash values ​​to get the details of several consecutive blocks. |
| getdata |  |  |  | Queries other nodes for Inventory objects of specified type and hash. <br>Current usage: <br>1)Sending get-transaction query during consensus process. <br>2)Sending getdata message upon receiving inv message. |
| block |  |  |  | Answers getdata message with Block of specified hash. |
| consensus |  | 〇 |  | Answers getdata message with consensus data of specified hash. |
| tx |  |  |  | Answering getdata message for a transaction with specified hash. |
| getheaders | 〇 |  |  | Node with fewer information queries for block head after two nodes establish connection. |
| headers |  |  |  | Answers getheaders message with at most 2000 block header items. |
| inv |  |  |  | Send Inventory hash array of specified type and hash (only hash value rather than complete information). Inventory types include Block, Transaction, and Consensus). Currently inv message is used in these scenarios: <br>1)Sending transaction in consensus process. <br>2)Replying getblocks message with no more than 500 blocks. <br>3) Replying mempool message with all transactions in memory pool. <br>4) Relaying an Inventory. <br>5) Relaying a batch of transactions. |
| mempool | 〇 | 〇 |  | Query for all transactions in the memory pool of the connected node. |
| version | - | 〇 | - | First instruction: with information like block header height, etc. |
| verack | - | 〇 | - | Sencond instruction: handshake Version |
| merkleblock |  |  | 〇 | Sending process is implemented while receiving is not. It's for SPV wallet. |
| ping |  |  |  | Standard ping message |
| pong |  |  |  | Response to ping message |
| alert |  | 〇 | 〇 | Unimplemented |
| notfound |  |  | 〇 | Unimplemented |
| reject |  |  | 〇 | Unimplemented |
| others |  |  | 〇 | Neglected |


> [!NOTE]
> - Uniqueness：Only one such message in message queue at the same time.
> - High priority：System need to guarantee higher priority to transmit for a message with higher priority.

## Conversation protocol

1. Firstly neo node tries connecting with seed nodes.

2. Sends firstly version message upon connecting successfully and waits for version message. Local block height is contained in version message.

3. Sends verack message and waits for verack message to complete handshake.

4. If local block height is lower than the other side, send getheaders message.

5. The other side will send headers message which contains no more than 2000 block heads upon receiving getheaders message.

6. If block head information has been synchronized and local block height is lower than the other side, send getblocks message.

7. The other side will send inv message which contains no more than 500 block hashes upon receiving getblocks message.

8. Determine whether received inv message is needed upon receiving. If so, send getdata message querying for block information.

9. The other side sends block message which contains complete block information upon receiving getdata message.

10. Neo node check the number of connections every 5 seconds. Connect backup nodes initiatively if connection is lower than 10. Will send getaddr message to connected nodes to query information about other nodes in network if backup nodes are not enough.

11. The other side sends addr message which contains address and port information of no more than 200 nodes upon receiving getaddr message.

12. Use hash to manage relatively big information like consensus messages / block / transaction data, to avoid receiving duplicated message from different nodes.

13. Nodes is responsible for relaying received consensus messages / block / transaction data to other nodes by inv message.

## Conversation Sequence Instance

| Message direction | Message type | Description |
| --- | --- | --- |
| send | version | Send version to perform 1st handshake |
| receive | verack | Receive verack to perform 2nd handshake |
| send | getheaders | Send getheaders to retrieve block head |
| receive | headers | Receive block headers |
| send | getblocks | Send getblocks to get blocks |
| receive | inv(blocks) | Receive inv hash of some blocks |
| send | getdata(blocks) | Send getdata to retrieve complete block |
| receive | inv(consensus) | Receive inv hash of consensus data |
| send | getdata(consensus) | Send getdata to retrieve consensus data of specified hash |
| receive | consensus | Receive complete consensus data |
| send | inv(consensus) | Relay received consensus data's hash to other nodes |
| receive | block | Receive a complete block |
| send | inv(block) | Relay block hash |
| receive | block | Receive a complete block |
| send | inv(block) | Relay block hash |
| receive | block | Receive a complete block |
| send | inv(block) | Relay block hash |
| ... | ... | ... |



