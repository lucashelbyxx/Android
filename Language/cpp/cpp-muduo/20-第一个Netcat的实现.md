# Thread-per-connection with blocking IO

- those two parallel loops map to two thread natively
  - thread-per-half-conneciton, to be exact
  - go uses this model universally
- two direcitons are not affecting each other
  - how to tell the other thread to exit, when it's blocking on read or write?
- blocking IO throttles traffic by default