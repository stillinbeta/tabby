Key:

- *bus message type*
- **Private/Secret data**

User Registration
-----------------

1. User choose Username, and **Password** in app
2. App sends Username and Password to Websocket Cache (http/ssl)
3. **Password** used to encrypt large **Secret**
4. Encrypted **Secret** stored in DB along with Username and **password** hash
   (for authenticating user)
5. "OK" returned to user in app (http/ssl)

Logging in to App
-----------------

1. User enters Username, **Password** into app
2. App sends Username and **Password** to Websocket Cache (http/ssl)
3. App checks User's credentials against hash in database
    1. If incorrect, return 401 (http/ssl)
4. Generate a **session key**
5. Store **session key** in database with expiration date[a]
6. Send **session key** to app (http/ssl)
7. Send a *Connection Request* to clients for each account (rabbitmq/ssl
    1. Request includes **Password**[b], Account
    2. Client Retrieves Account Information from DB
    3. Client decrypts **Account Password** with **Secret** and **Password**
    4. Client connects to server

Adding an IM account
--------------------

1. User enters Account Username, **Account Password**, Account Type,
   Account Server into app
2. Account data and **Password** is sent to Websocket Cache (http/ssl)
3. **Secret** is decrypted with **Password**
4. **Account Password** is encrypted with **Secret**
5. Encrypted **Account Password** and Account information stored in database
6. "OK" returned to user in app (http/ssl)

Receiving a message
-------------------

1. Client receives message (jabber/ssl?|irc/ssl)
2. *Message Notification* sent to bus (rabbitmq/ssl)
3. GCM receives message
    1. GCM checks notification preference for account/message type
    2. If notification requested, retrieve **current session** key from database
    3. Encrypt message with **session key**
    4. Send message to Google
    5. Phone receives message, decrypts using stored **session key**
4. Websocket Cache receives message
    1. Message is stored in Cache until requested[c]
    2. App requests cached results (http/ssl)
    3. Messages sent to app (http/ssl)
    4. Messages cleared from Redis

Sending a Message
-----------------

1. User sends message using a given account
2. App sends message, username, account, and **session key** (http/ssl)
3. Websocket Cache receives message
4. Websocket Cache checks session key for validity and expiration
    1. If invalid, return 401
5. *Outgoing Message* sent to bus (http/ssl)
6. appropriate client sends message to remote server (ssl?)




[a]Ellie Frost:
How is rollover handled?
[b]Ellie Frost:
This precludes reconnection
[c]Ellie Frost:
Persistance?
