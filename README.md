# HTTP-perf-testing

Other testing tools https://github.com/denji/awesome-http-benchmark

## payload for testing

[payload to sign single PDF document](./payload-SignDocument.json)

## ab - Apache HTTP server benchmarking tool

[Apache HTTP server benchmarking tool](https://httpd.apache.org/docs/2.4/programs/ab.html).

Perf testing using 10 parallel connections, timeout 120s and total number of 1000 signatures:
```
ab -r -n 1000 -s 120  -c 10 -v 2 \
    -T 'application/json' \
    -p payload-SignDocument.json
    https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process 2>&1 | tee /tmp/signserver
...

Server Software:
Server Hostname:        signserver.3key.company
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Server Temp Key:        X25519 253 bits
TLS Server Name:        signserver.3key.company

Document Path:          /signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Document Length:        72030 bytes

Concurrency Level:      10
Time taken for tests:   27.685 seconds
Complete requests:      1000
Failed requests:        723
   (Connect: 0, Receive: 0, Length: 723, Exceptions: 0)
Total transferred:      72365764 bytes
Total body sent:        17947000
HTML transferred:       72028981 bytes
Requests per second:    36.12 [#/sec] (mean)
Time per request:       276.847 [ms] (mean)
Time per request:       27.685 [ms] (mean, across all concurrent requests)
Transfer rate:          2552.66 [Kbytes/sec] received
                        633.07 kb/s sent
                        3185.73 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        3   23  33.9     13     522
Processing:   145  250  97.7    227     874
Waiting:      141  220  87.7    199     835
Total:        150  273 107.3    246     904

Percentage of the requests served within a certain time (ms)
  50%    246
  66%    272
  75%    290
  80%    304
  90%    353
  95%    463
  98%    664
  99%    800
 100%    904 (longest request)
```

Tool reports lots of connections as failed. In test above 723 of 1000. But in recoreded output:
```
semik@lab03:~/HTTP-perf-testing$ grep HTTP /tmp/signserver | head -3
POST /signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process HTTP/1.0
HTTP/1.1 200 OK
HTTP/1.1 200 OK
semik@lab03:~/HTTP-perf-testing$ grep HTTP /tmp/signserver | grep -v POST | grep 'HTTP/1.1 200 OK' | wc -l
1000
```
All tests finished with HTTP status 200. Why Failed? It is possible to print output using `-v 2` option but I was unable to figure out why it is reporting errors.

## wrk - a HTTP benchmarking tool

[wrk](https://github.com/wg/wrk/) - a HTTP benchmarking tool

```
semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c10 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   244.17ms  137.92ms   1.14s    89.09%
    Req/Sec     4.76      2.38    10.00     73.04%
  1298 requests in 30.08s, 89.65MB read
Requests/sec:     43.15
Transfer/sec:      2.98MB

semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c20 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 20 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   241.82ms  100.94ms 777.71ms   88.56%
    Req/Sec     9.19      3.74    20.00     76.96%
  2517 requests in 30.11s, 174.03MB read
Requests/sec:     83.61
Transfer/sec:      5.78MB

semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c40 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 40 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   294.56ms  143.98ms   1.21s    83.88%
    Req/Sec    15.89      8.54    40.00     72.52%
  4150 requests in 30.10s, 286.64MB read
Requests/sec:    137.89
Transfer/sec:      9.52MB

semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c80 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   469.98ms  219.61ms   1.46s    69.13%
    Req/Sec    19.48     11.82    70.00     80.00%
  5099 requests in 30.10s, 352.49MB read
Requests/sec:    169.40
Transfer/sec:     11.71MB

semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c200 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.16s   388.43ms   2.00s    64.36%
    Req/Sec    19.09     12.85    89.00     76.47%
  4698 requests in 30.09s, 324.59MB read
  Socket errors: connect 0, read 0, write 0, timeout 287
Requests/sec:    156.14
Transfer/sec:     10.79MB

semik@lab03:~/HTTP-perf-testing$ wrk -t50 -c200 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  50 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.24s   390.91ms   2.00s    61.74%
    Req/Sec     4.28      4.15    27.00     70.36%
  3968 requests in 30.10s, 274.14MB read
  Socket errors: connect 0, read 0, write 0, timeout 709
Requests/sec:    131.85
Transfer/sec:      9.11MB
```

Looks like peak performance can be achieved by 80 concurent connections no matter how many threads:
```
semik@lab03:~/HTTP-perf-testing$ wrk -t10 -c80 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  10 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   469.98ms  219.61ms   1.46s    69.13%
    Req/Sec    19.48     11.82    70.00     80.00%
  5099 requests in 30.10s, 352.49MB read
Requests/sec:    169.40
Transfer/sec:     11.71MB

semik@lab03:~/HTTP-perf-testing$ wrk -t80 -c80 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  80 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   511.98ms  231.20ms   1.95s    68.91%
    Req/Sec     1.88      1.31    10.00     83.25%
  3714 requests in 30.10s, 256.55MB read
  Socket errors: connect 0, read 0, write 0, timeout 80
Requests/sec:    123.39
Transfer/sec:      8.52MB

semik@lab03:~/HTTP-perf-testing$ wrk -t80 -c80 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  80 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   477.92ms  234.23ms   1.64s    70.67%
    Req/Sec     2.20      1.50    10.00     79.43%
  5079 requests in 30.11s, 351.33MB read
Requests/sec:    168.67
Transfer/sec:     11.67MB

semik@lab03:~/HTTP-perf-testing$ wrk -t1 -c80 -d30s --script=wrk-SignDocument.lua  https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
Running 30s test @ https://signserver.3key.company/signserver/rest/v1/workers/PAdES-Baseline-B-Perf/process
  1 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   491.18ms  248.15ms   1.99s    73.46%
    Req/Sec   179.67     90.35   500.00     69.43%
  4869 requests in 30.05s, 336.45MB read
  Socket errors: connect 0, read 0, write 0, timeout 3
Requests/sec:    162.04
Transfer/sec:     11.20MB
```

## curl & parallel

Create simple script for one test like [./curl-test](./curl-test) and exec it using parallel. `seq ...` defines number of iterations. Argument `-j 200` defines how many tests will be executed in parallel.

```
semik@lab03:~/HTTP-perf-testing$ rm out/* ; time seq 1 5000 | parallel -j 20 ./curl-test

real 1m29.039s
user 2m12.876s
sys	 2m2.750s


semik@lab03:~/HTTP-perf-testing$ rm out/* ; time seq 1 5000 | parallel -j 80 ./curl-test
rm: cannot remove 'out/*': No such file or directory

real 1m9.611s
user 2m3.770s
sys	 1m58.772s


semik@lab03:~/HTTP-perf-testing$ rm out/* ; time seq 1 5000 | parallel -j 200 ./curl-test

real 1m24.871s
user 2m24.346s
sys	 2m26.106s
```

5000/1m9.611s ~ 72 Req/sec. During this testing method I was unable to reach values measured with wrk, it might be because executing curl again and again for each test requires more system resources.

## HSM load

Varies between 30-45%

```
semik@lab07:~$ snmpwalk -v 2c -c public localhost NCIPHER-MIB::loadModule.1 ; snmpwalk -v 2c -c public localhost NCIPHER-MIB::loadAll.0
NCIPHER-MIB::loadModule.1 = Gauge32: 43
NCIPHER-MIB::loadAll.0 = Gauge32: 43
```
