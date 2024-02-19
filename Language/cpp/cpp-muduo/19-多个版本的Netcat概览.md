# Operation modes

- server
- client
- bind + listen + accept
- resolve address + connect
- once connection has been established, c/s behave the same
- two parallel loops serve two directions:
  - read from stdin, write to TCP socket
  - read from TCP socket, write to stdout



# Not all netcats are created equal

- recipes / tpc / netcat.cc		thread-per-connection
- recipes / python / netcat.py		IO-multiplexing
- recipes / python / netcat-nonblock.py		IO-multiplexing
- load generator: chargen
  - recipes / tpc / chargen.cc
  - recipes / python / chargen.py
  - muduo / examples / simple / chargen / *