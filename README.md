# Proxeus external node Golang library: node-go

This repository is part of the [Proxeus](https://proxeus.com) project. Using external nodes are the primary method to 
extend Proxeus functionality.

## Principles

[Proxeus Core](https://github.com/ProxeusApp/proxeus-core) is a workflow engine specialised for the blockchain.  Each
workflow is compose of one or more interconnected nodes.  The core is delivered with a fixed set of core node types
like for example condition, template, form nodes.

Additional functionality is provide by external nodes which are web servers exposing a well defined API.  Each external
node will first register to a running Proxeus Core engine giving its Id, Name, and URL.  On registration, the core will
make the registered node type available to users.  The new node type will appear in the node list alongside the
other nodes.

During workflow design, users can instantiate the new node type in their workflow.  The node will behave the same
way as built-in nodes.  External node will expose a web configuration interface that is used by the workflow designer
to configure the instantiated node. The configuration is then saved in the core and external nodes do not need to have
any storage.  This clearly separate the responsibility between the core and the exteranl nodes.

During execution, when the workflow state reaches the instantiated external node, the core will call the node using 
HTTP and transmit the workflow state.  The external node will query the node instance configuration from the core, 
execute its task and return the new state of the workflow.

## Contract

An external node is an HTTP server that first register to a Proxeus instance and then provides the following API:

* GET /node/:id/config return the configuraton HTML page for the given node instance.
* POST /node/:id/config updated the configuration of the node instance with the give id.
* POST /node/:id/next trigger the execution of a node instance.
* POST /node/:id/remove remove a node instance, called when a node instance is removed from a workflow definition.
* GET /health heartbeat to check that the node is alive.

Proxeus core will provide the following API to external nodes:

* POST /api/admin/external/register to register an external node.
* POST /api/admin/external/config/:id to store the configuration of a node instance.
* GET /api/admin/external/config/:id to read the configuration of a node instance.

## Security

During registration, external node will provide a shared secret to be used by the core when calling the external 
node API, including the configuration page.

Current assumption is that external nodes are run by Proxeus operators on a private network.  Also, since the full 
state of a workflow is currently transmitted to the external node, Proxeus operators must convince themselves that the 
node will use this data as specified.

## Distribution

External nodes are currently distributed as Docker images pulled from well known Docker repositories.  Operators will 
assemble a Proxeus solution using a docker compose configuration file containing a proxeus-core instance and as many 
external node instances as required by their use cases.

## Examples

The following examples are using this library to provide two blockchain external nodes that can be used in any 
Proxeus workflow:

* (node-balance-retriever)[https://github.com/ProxeusApp/node-balance-retriever] which return the ETH and ERC20 balance
for a given Ethereum address,
* (node-crypto-forex-rates)[https://github.com/ProxeusApp/node-crypto-forex-rates] which converts various crypto and 
fiat currencies at current market rates.
