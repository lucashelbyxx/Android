UDP vs. TCP

- UDP: connection-less, unreliable, datagram protocol
  - UDP = IP + port ( + weak checksum)
- TCP: connection-oriented, reliable, byte-stream protocol
- when in doubt, use TCP
- cuncurrent model: how to manage multiple concurrent clients?
  - TCP: one socket fd for each client, not thread safe
  - UDP: usually share one fd for all clients, thread safe