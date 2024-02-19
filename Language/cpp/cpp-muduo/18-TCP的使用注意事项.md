# ignore SIGPIPE in any concurrent TCP server

- SIGPIPE is sent to writer when the other end of "pipe" is closed
  - default action is terminating the process, good for command line piping
    - gunzip -c huge.log.gz | grep ERROR | head
- what if client closes a socket when server is still writing to it?
  - client could be offensive or misbehaving, server should be defensive
- the default action will kill the server and affect all clients
  - where is my server process after Ctrl-C a client?
- so ignore SIGPIPE in network programs
  - if a program writes to stdout, check return value of printf() then exit()



# Nagle's algorithm, TCP_NODELAY

- affect latency of request-response protocol
- write() will not send data if there is any unacked TCP segment
- for write-write-read, the second write will be delayed by one RTT
  - solution: buffering, make it write-read
  - however, it still affect request pipelining
- i recommend to disable Nagle's algorithm by default
  - go does this for  every TCP connection



# trilogy of TCP client/server

- SO_REUSEADDR
  - so that a TCP server can restart immediately after crash/kill
  - also needed for fork-per-connection model
- ignore SIGPIPE
- TCP_NODELAY