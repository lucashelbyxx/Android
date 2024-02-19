# why netcat?

- read/write two (three) file descriptors simultaneously
- tow simple concurrent modes are examined
  - thread-per-connection with blocking IO
  - IO-multiplexing with non-blocking IO
- how/when to close a TCP connection?
  - ensure all data are sent and received
- why IO-multiplexing must be used with non-blocking IO?



# swiss army knife

- nc < /dev/zero	==	chargen
- nc > /dev/null	==	discard
- dd /dev/zero | nc 	==	poor man's ttcp
- nc < file, nc > file	==	poor man's scp



# how/when to close a TCP connection?

- why my TCP is not reliable?
  - sent all then close, but last few bytes are lost
- wrong: send() + close()
  - when there is data in input buffer, close() will cause RST, terminate conneciton prematurely
  - with or without SO_LINGER? TL; DR: don't use LINGER
- correct sender: send() + shutdown(WR) + read() -> 0 + close()
- correct receiver: read() -> 0 + if nothing more to send + close()
- or, better, design you app protocol to allow safe disconn

