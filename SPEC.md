# The Euphony Project

>  The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
>  NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
>  "OPTIONAL" in this document are to be interpreted as described in
>  [RFC2119](https://tools.ietf.org/html/rfc2119).

## Introduction

The Euphony Project is a chat protocol, inspired by Matrix and Discord. It's fully free and can be implemented by anybody. Servers using the protocol are designed to federate, which allows for multiple self-hosted instances to connect to each other.

The main goal of this project is to create a chat platform that is transparent and easy to implement, while retaining many functions that other software does not have (such as servers - groups with channels in them).

## Keywords

* User - User of an instance.
* Instance - The instance (running a Euphony Project compatible server). **DO NOT CONFUSE WITH SERVERS.**
* Server - A group which contains channels. **DO NOT CONFUSE WITH INSTANCES.**
* Channel - A channel of communication. DMs and group chats are also channels.
* Action - Actions taking place on the server (see the Actions section for more information).
* Message - A single message, placed in a channel.

### Example keywords

Note: if only one user/instance/server/action is mentioned in the example, use the full words. Only use these keywords for examples where multiple objects exist.

* Ux - User. The x is a number assigned to each user in the example.
* Ix - Instance. The x is a number assigned to each instance in the example.
* Sx - server. The x is a number assigned to each server in the example.
* Ax - Action. The x is a number assigned to each action in the example.

## Dates

All dates MUST be stored as time from the UNIX epoch. This prevents date formatting issues and makes them easier to parse.

## Objects and actions

All objects and actions are assigned a numerical ID. Numerical IDs MUST begin at the value of 0, and increase upon every registered action or object. These IDs are shared between objects and actions, so they MUST NOT overlap.

Each object and action, when retrieved, MUST contain the following values in the request returned by the server alongside its own:

- id (number, contains the object/action's ID)
- type (string, object or action)

### Objects

Users, servers, channels and messages are objects. Every object MUST have a numerical ID, which allows it to be quickly located and accessed. You **cannot** change an object's ID once it's been assigned. Object IDs MUST NOT overlap. Object IDs and action IDs MUST NOT overlap.

In the Euphony Project, the main hierarchy of objects is as follows (from largest/most important to smallest/least important):

* Instance (running a Euphony Project compatible server) - this MUST have the numerical ID of 0
* User (somebody who uses the service)
* Role (servers together certain people in a server)
* Server (a group containing channels, compare to Discord's servers/guilds)
* Channel (which contains messages, compare to IRC/Matrix channels or channels in a Discord server/guild)
* Message

An object MUST have the type value set to 'object'.

#### Example

An application wants to get information about an user. In order to do so, it must first find its object ID. Once located, this ID can be used to access information about the user, even if they change their name.

### Actions

If an action modifies the information about any object, it can be classified with its own ID. Examples of actions include: changing profile information; sending, deleting or editing a message; changing a server's prefferences. Each action has its own numerical ID. Action IDs MUST NOT overlap. Object IDs and action IDs MUST NOT overlap.

An action MUST have the type value set to 'action'.

#### Example

An user changes their profile picture. This action gets its own numerical ID, which allows it to categorize it on the server. This action can then be federated to other servers. Upon recieving the action, other servers will assign it its own IDs, while keeping track of the original ID that it had on the server (see Federation), so if it ever needs to be reverted you can notify those servers about it.

### Retrieving information about IDs

In order to get information about an object/action based on its ID, you can query the API:

```
$domain/api/v1/id/<ID>
```

where <ID> is the ID you want to get information on. This will return a request that will look as follows:

```json
{
	"id": <ID>,
	"type": <object/action>,
	...insert the object/action's required values...
}
```

## Federation

Each instance MUST have a public inbox (which can be accessed by other servers) and SHOULD have a private outbox (where requests that will be sent to other instances are stashed).

Federation works on the following principle:

Each channel can be subscribed to, which means that the instance is aware of the channel's existence and will sync it, allowing the users of that instance to recieve and send messages. Instances stash content that has to be sent to other instances' inboxes in a private outbox.

When an object or action gets from the remote instance to the local instance,

### Example

U1 wants to join a server. U1 is on I1, while the server is on I2. I1 isn't subscribed to the server yet, so it sends a request to I2 asking to be subscribed to the server. If I2 doesn't block I1 and I2 is up, I1 will subscribe to the server on I2, thus allowing U1 to communicate with people on the server.

U2 is on I2. U2 sends a message to the server on I2. The server pings every subscribed instance, including I2, sending them the action. I1 gets the action, saves it on itself and sends a confirmation to the server.

### Stashing

When the remote server can't be contacted, the local server creates a stash list, which contains the ID ranges that have to be sent to the remote server. It pings the remote server every X minutes (reccomended default: 10 minutes) and if it gets a connection, it sends it the stashed requests. If a server is down for more than 7 days, it is considered closed, until a request is recieved from the software.

In order to ease this process, servers can send a request to another letting them know that they're back online. This triggers an automatic sync that recieves all requests from a stash without having to wait until the remote server is available. It is reccomended that server software checks for connection dropouts and, once identified, waits for the dropout to end, then send the resync request once the connection is available.

As such, this concept covers two cases:

- one of the servers drops the connection temporarily, but is still running
- one of the servers is shut down

> :warning: Don't send IDs to the remote server that aren't supposed to go it! It's a waste of bandwith and a potential security threat, as bad actors may attempt to intercept information that way.

### Accessing information about remote instances

When an instance begins to federate with another instance, its domain is saved in

```
$domain/api/v1/federated/remote-instance.domain
```

where ``remote-instance.domain`` is the remote instance.

If queried, this will return the equivalent of the remote server's ID 0 (which contains information about the instance).

```json
{
	"type": object,
	"object-type": instance,
	...insert instance-specific information here...
}
```

Assuming an object/action has federated to the instance, it is possible to retrieve its ID on the server it has federated to using

```
$domain/api/v1/federated/remote-instance.domain/ID
```

Where ``ID`` is the ID from the remote domain.

This will automatically locate the right ID and print required information.

```json
{
	"remote-domain": Domain from which the ID was recieved.
	"remote-id": ID on remote server,
	"id": ID on local server,
	...insert other object/action specific information...
}
```

## Servers

> :warning: DO NOT confuse servers with instances or group DMs!

A server is a group comprising of any amount of text and voice channels. Users can join a server, and roles can be given to them. These roles can have certain permissions assigned to them.

## List of objects with properties

> :information_source: The layout of each property is as follows:
> name (type, optional description) {read/write permissions (read - r, write - w)}

### Instance

#### Public values

- object-type (string, MUST be "instance")
- address (string, contains domain name, required for federation) {r}
- server-software (string, set by the server, contains name and version of the Euphony-compatible server) {r}
- name (string, instance-specific name) {r[w]} [modify:instance]
- description (string, instance-specific description) {r[w]} [modify:instance]

### User

#### Public values

- object-type (string, MUST be "user")
- nickname (string) {r[w]} [account.modify]
- status (string) {r[w]} [account.modify]
- bio (string) {r[w]} [account.modify]

#### Private values (require scope for reading)

- email (string) {[rw]} [account.modify]

### Message

#### Public values

- object-type (string, MUST be "message")
- content (string) {r[w]} [message.edit]
- server-id (number, contains ID of the server the channel is present in) {r}
- creator (number, contains ID of the user who sent the message) {r}
- post-date (number, date) {r}
- edit-dates (list with numbers, date, optional) {r}
- attachement (string, contains link to file with https:// or http:// prefix) {r}

### Server

#### Public values

- object-type (string, MUST be "server")
- name (string)
- description (string)
- creation-date (number, date)
- channels (list of numbers, contains IDs for text and voice channels on the server)
- users (list of numbers, contains IDs of users on the server)

#### Private values (require scope)

##### Settings

**Information** [server.info]

- name (string) {r[w]}
- description (string) {r[w]}
- icon (string, link to file with https:// or http:// prefix) {r[w]}
- in-search-index (bool, decides if the server can be found through built-in search tools. The reccomended default is false.) {r[w]}
- owner (number, ID of owner user) {r, rw with server.changeowner scope}

**Roles** [server.roles]

> :information_source: Note: this scope is also required to view information about individual roles.

- roles (list of numbers, contains IDs of roles on the server)

**Individual role information** (``$domain/api/v1/server/role/ID``)

- object-type (string, MUST be "role")
- name (string) {r[w]}
- description (string, small description of role) {r[w]}
- permissions (list - permission map (see Permissions for more information))

#### Closed values (cannot be requested)

- password (contains server password if one is set)