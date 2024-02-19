# IO-multiplexing

- IO-multiplexing is also called event-driven, event-based, reactor
  - it is not aysnchronous!
- use only one thread
  - stock netcats are implemented in this way
- with blocking IO: one direction could block the other
- demo
  - server: chargen.py, a white hole
  - client: nc, nc < /dev/zero

