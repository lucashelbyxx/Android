Benchmarking results on Celeron 1037U

- IO intensive
  - short connectinos: 5000 requests per second
  - keep-alive: 12780 RPS (<100us)
  - keep-alive, 2 connections: 17668 RPS
- cpu.png    CPU intensive    plot_test.cc
  - short connections: 571 requests per second
  - keep-alive: 670 RPS (1.5ms)
  - keep-alive, 2 connections: 720 RPS

