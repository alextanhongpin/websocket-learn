# websocket-learn
Knowledge base for what I learn with Websocket and how to apply them in production


websocket learn

- there are several logic for the websocket session. First, a user can have multiple sessions on different devices or browser tab. This is normally the expected behaviour. But when we broadcast a message, we need to consider if this is what we want, since we will be duplicating the requests to multiple devices for the same user, which can be spammy.
- we can choose to limit the user (identified by the user id) to be connected to one websocket connection only. When the user opens another tab, we can either kill the old session and only use the new one, or maintain the old one and kill the new one.
- in the case where the user is online with multiple sessions, the message that the user send must be send to all the users sessions too.
- If the server restarts, the web socket connection will be terminated. The old connections needs to be removed and the client has to attempt to reconnect again.
- Authorization. User can attempt to obtain a ticket first, (which is a one time use only access token) to initiate the connection. For every other connection, they have to request a new ticket.
- The users session may be distributed across different machines. In order to detect which machine the session may be located, we need a pub/sub system to notify whenever a new message is send by one client. We can use redis pub/sub for this, and register the user based on the user_id:machine_id:session_id, or just store the machine_id:session_id set in the user sorted set/hash map. we need to store and inverted index of this because we need to be able to find both machine id and the session id based on the user. Then we need to publish the message to the machines because only the message will have the session for the websocket and is able to send to the client.


Register the user’s session and machine. 
hash_map[user_id]: [session1_id: machine1_id, session2_id: machine2_id]

To send to another user, we just need to know the user’s id. Then we can find the machine the user belongs to and publish the message to that machine.

## Consistent Hashing Application


## PubSub

```go
package main

import (
	"fmt"
)

func main() {
	// conn is of type redis.Conn.
	pubsubConn := &redis.PubSubConn{Conn: conn}
	defer pubsubConn.Close()
	
	server := os.Hostname()
	err := pubsubConn.Subscribe(server)
	if err != nil {
		log.Fatal(err)
	}
	go subscribe(pubsubConn)
}

func subscribe(conn *redis.PubSubConn) {
	for {
		switch v := conn.Receive().(type) {
			case redis.Message:
				// Do something with message.
				fmt.Println(v.Channel, string(v.Data))
				// Send to the broadcast channel.
			case redis.Subscription:
				fmt.Println(v.Channel, v.Kind, v.Count)
			case error:
				fmt.Println("error pub/sub, delivery has stopped")
				return
		}
	}
}

// server:xyz:user:john: true
// user:john -> server:xyz
```
