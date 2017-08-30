Go HTTP Router Benchmark
========================

This benchmark suite aims to compare the performance of HTTP request routers for [Go](https://golang.org) by implementing the routing structure of some real world APIs.
Some of the APIs are slightly adapted, since they can not be implemented 1:1 in some of the routers.

Of course the tested routers can be used for any kind of HTTP request → handler function routing, not only (REST) APIs.


#### Tested routers & frameworks:

 * [Beego](http://beego.me/)
 * [go-json-rest](https://github.com/ant0ine/go-json-rest)
 * [chi](https://github.com/go-chi/chi)
 * [Denco](https://github.com/naoina/denco)
 * [Gocraft Web](https://github.com/gocraft/web)
 * [Goji](https://github.com/zenazn/goji/)
 * [Gorilla Mux](http://www.gorillatoolkit.org/pkg/mux)
 * [http.ServeMux](http://golang.org/pkg/net/http/#ServeMux)
 * [HttpRouter](https://github.com/julienschmidt/httprouter)
 * [HttpTreeMux](https://github.com/dimfeld/httptreemux)
 * [Kocha-urlrouter](https://github.com/naoina/kocha-urlrouter)
 * [Martini](https://github.com/go-martini/martini)
 * [Pat](https://github.com/bmizerany/pat)
 * [Possum](https://github.com/mikespook/possum)
 * [R2router](https://github.com/vanng822/r2router)
 * [TigerTonic](https://github.com/rcrowley/go-tigertonic)
 * [Traffic](https://github.com/pilu/traffic)


## Motivation

Go is a great language for web applications. Since the [default *request multiplexer*](http://golang.org/pkg/net/http/#ServeMux) of Go's net/http package is very simple and limited, an accordingly high number of HTTP request routers exist.

Unfortunately, most of the (early) routers use pretty bad routing algorithms. Moreover, many of them are very wasteful with memory allocations, which can become a problem in a language with Garbage Collection like Go, since every (heap) allocation results in more work for the Garbage Collector.

Lately more and more bloated frameworks pop up, outdoing one another in the number of features. This benchmark tries to measure their overhead.

Be aware that we are comparing apples and oranges here. We compare feature-rich frameworks to packages with simple routing functionality only. But since we are only interested in decent request routing, I think this is not entirely unfair. The frameworks are configured to do as little additional work as possible.

If you care about performance, this benchmark can maybe help you find the right router, which scales with your application.

Personally, I prefer slim and optimized software, which is why I implemented [HttpRouter](https://github.com/julienschmidt/httprouter), which is also tested here. In fact, this benchmark suite started as part of the packages tests, but was then extended to a generic benchmark suite.
So keep in mind, that I am not completely unbiased :relieved:


## Results

Benchmark System:
 * Intel Core i5-2500K (4x 3,30GHz + Turbo Boost), CPU-governor: performance
 * 2x 4 GiB DDR3-1333 RAM, dual-channel
 * go version go1.3rc1 linux/amd64
 * Ubuntu 14.04 amd64 (Linux Kernel 3.13.0-29), fresh installation


### Memory Consumption

Besides the micro-benchmarks, there are 3 sets of benchmarks where we play around with clones of some real-world APIs, and one benchmark with static routes only, to allow a comparison with [http.ServeMux](http://golang.org/pkg/net/http/#ServeMux).
The following table shows the memory required only for loading the routing structure for the respective API.
The best 3 values for each test are bold. I'm pretty sure you can detect a pattern :wink:

| Router       | Static    | GitHub     | Google+   | Parse     |
|:-------------|----------:|-----------:|----------:|----------:|
| HttpServeMux |__18064 B__|         -  |        -  |        -  |
| Beego        |  79472 B  |  497248 B  |  26480 B  |  38768 B  |
| Denco        |  44752 B  |  107632 B  |  54896 B  |  36368 B  |
| Gocraft Web  |  57976 B  |   95736 B  |   8024 B  |  13120 B  |
| Goji         |  32400 B  | __58424 B__| __3392 B__| __6704 B__|
| Go-Json-Rest | 152608 B  |  148352 B  |  11696 B  |  13712 B  |
| Gorilla Mux  | 685152 B  | 1557216 B  |  80240 B  | 125480 B  |
| HttpRouter   |__26232 B__| __44344 B__| __3144 B__| __5792 B__|
| HttpTreeMux  |  75624 B  |   81408 B  |   7712 B  |   7616 B  |
| Kocha        | 130336 B  |  811744 B  | 139968 B  | 191632 B  |
| Martini      | 312592 B  |  579472 B  |  27520 B  |  50608 B  |
| Pat          |__21272 B__| __18968 B__| __1448 B__| __2360 B__|
| TigerTonic   |  85264 B  |   99392 B  |  10576 B  |  11008 B  |
| Traffic      | 649568 B  | 1124704 B  |  57984 B  |  98168 B  |

The first place goes to [Pat](https://github.com/bmizerany/pat), followed by [HttpRouter](https://github.com/julienschmidt/httprouter) and [Goji](https://github.com/zenazn/goji/). Now, before everyone starts reading the documentation of Pat, `[SPOILER]` this low memory consumption comes at the price of relatively bad routing performance. The routing structure of Pat is simple - probably too simple. `[/SPOILER]`.

Moreover main memory is cheap and usually not a scarce resource. As long as the router doesn't require Megabytes of memory, it should be no deal breaker. But it gives us a first hint how efficient or wasteful a router works.


### Static Routes

The `Static` benchmark is not really a clone of a real-world API. It is just a collection of random static paths inspired by the structure of the Go directory. It might not be a realistic URL-structure.

The only intention of this benchmark is to allow a comparison with the default router of Go's net/http package, [http.ServeMux](http://golang.org/pkg/net/http/#ServeMux), which is limited to static routes and does not support parameters in the route pattern.

In the `StaticAll` benchmark each of 157 URLs is called once per repetition (op, *operation*). If you are unfamiliar with the `go test -bench` tool, the first number is the number of repetitions the `go test` tool made, to get a test running long enough for measurements. The second column shows the time in nanoseconds that a single repetition takes. The third number is the amount of heap memory allocated in bytes, the last one the average number of allocations made per repetition.

The logs below show, that http.ServeMux has only medium performance, compared to more feature-rich routers. The fastest router only needs 1.8% of the time http.ServeMux needs.

[HttpRouter](https://github.com/julienschmidt/httprouter) was the first router (I know of) that managed to serve all the static URLs without a single heap allocation. Since [the first run of this benchmark](https://github.com/julienschmidt/go-http-routing-benchmark/blob/0eb78904be13aee7a1e9f8943386f7c26b9d9d79/README.md) more routers followed this trend and were optimized in the same way.

```
BenchmarkHttpServeMux_StaticAll            50000             22378 ns/op              96 B/op          8 allocs/op

BenchmarkAce_StaticAll                    100000             14959 ns/op               0 B/op          0 allocs/op
BenchmarkBeego_StaticAll                   10000            147814 ns/op           55264 B/op        471 allocs/op
BenchmarkBear_StaticAll                    20000             67145 ns/op           20336 B/op        461 allocs/op
BenchmarkBone_StaticAll                    30000             53677 ns/op               0 B/op          0 allocs/op
BenchmarkChi_StaticAll                     10000            130823 ns/op           67824 B/op        471 allocs/op
BenchmarkDenco_StaticAll                  300000              5251 ns/op               0 B/op          0 allocs/op
BenchmarkEcho_StaticAll                    50000             36602 ns/op            5024 B/op        157 allocs/op
BenchmarkGin_StaticAll                    100000             15566 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_StaticAll              10000            102703 ns/op           46440 B/op        785 allocs/op
BenchmarkGoji_StaticAll                    50000             36484 ns/op               0 B/op          0 allocs/op
BenchmarkGojiv2_StaticAll                  10000            402730 ns/op          205984 B/op       1570 allocs/op
BenchmarkGoJsonRest_StaticAll              10000            159874 ns/op           51653 B/op       1727 allocs/op
BenchmarkGoRestful_StaticAll                1000           1302223 ns/op          314936 B/op       3144 allocs/op
BenchmarkGorillaMux_StaticAll               2000           1127755 ns/op          150816 B/op       1421 allocs/op
BenchmarkHttpRouter_StaticAll             200000              8546 ns/op               0 B/op          0 allocs/op
BenchmarkHttpTreeMux_StaticAll            200000              8657 ns/op               0 B/op          0 allocs/op
BenchmarkKocha_StaticAll                  100000             11607 ns/op               0 B/op          0 allocs/op
BenchmarkLARS_StaticAll                   100000             14099 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_StaticAll                 10000            257922 ns/op          120577 B/op       1413 allocs/op
BenchmarkMartini_StaticAll                  1000           1701669 ns/op          125442 B/op       1717 allocs/op
BenchmarkPat_StaticAll                      1000           1844571 ns/op          533904 B/op      11123 allocs/op
BenchmarkPossum_StaticAll                  10000            108620 ns/op           65312 B/op        471 allocs/op
BenchmarkR2router_StaticAll                20000             62036 ns/op           22608 B/op        628 allocs/op
BenchmarkRivet_StaticAll                   30000             45232 ns/op           17584 B/op        314 allocs/op
BenchmarkTango_StaticAll                   10000            150674 ns/op           39225 B/op       1256 allocs/op
BenchmarkTigerTonic_StaticAll              50000             36968 ns/op            7504 B/op        157 allocs/op
BenchmarkTraffic_StaticAll                   300           4987419 ns/op         3711480 B/op      27004 allocs/op
BenchmarkVulcan_StaticAll                  10000            105752 ns/op           15386 B/op        471 allocs/op
```

### Micro Benchmarks

The following benchmarks measure the cost of some very basic operations.

In the first benchmark, only a single route, containing a parameter, is loaded into the routers. Then a request for a URL matching this pattern is made and the router has to call the respective registered handler function. End.
```
BenchmarkAce_Param                      20000000               115 ns/op              32 B/op          1 allocs/op
BenchmarkBear_Param                      2000000               972 ns/op             456 B/op          5 allocs/op
BenchmarkBeego_Param                     2000000               832 ns/op             352 B/op          3 allocs/op
BenchmarkBone_Param                      1000000              1635 ns/op             816 B/op          6 allocs/op
BenchmarkChi_Param                       2000000               797 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_Param                    20000000               114 ns/op              32 B/op          1 allocs/op
BenchmarkEcho_Param                     10000000               176 ns/op              32 B/op          1 allocs/op
BenchmarkGin_Param                      30000000                55.0 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_Param                1000000              1198 ns/op             648 B/op          8 allocs/op
BenchmarkGoji_Param                      2000000               630 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_Param                    1000000              2411 ns/op            1328 B/op         11 allocs/op
BenchmarkGoJsonRest_Param                1000000              1279 ns/op             649 B/op         13 allocs/op
BenchmarkGoRestful_Param                  500000              4216 ns/op            2296 B/op         21 allocs/op
BenchmarkGorillaMux_Param                1000000              2411 ns/op            1264 B/op         10 allocs/op
BenchmarkHttpRouter_Param               20000000                85.2 ns/op            32 B/op          1 allocs/op
BenchmarkHttpTreeMux_Param               2000000               781 ns/op             352 B/op          3 allocs/op
BenchmarkKocha_Param                    10000000               207 ns/op              56 B/op          3 allocs/op
BenchmarkLARS_Param                     30000000                59.8 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_Param                   1000000              2114 ns/op            1056 B/op         10 allocs/op
BenchmarkMartini_Param                    500000              4161 ns/op            1072 B/op         10 allocs/op
BenchmarkPat_Param                       1000000              1433 ns/op             648 B/op         12 allocs/op
BenchmarkPossum_Param                    1000000              1066 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_Param                  2000000               969 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_Param                     2000000               955 ns/op             464 B/op          5 allocs/op
BenchmarkTango_Param                     2000000               832 ns/op             248 B/op          8 allocs/op
BenchmarkTigerTonic_Param                1000000              2392 ns/op             992 B/op         17 allocs/op
BenchmarkTraffic_Param                    500000              3689 ns/op            1960 B/op         21 allocs/op
BenchmarkVulcan_Param                    3000000               432 ns/op              98 B/op          3 allocs/op
```

Same as before, but now with multiple parameters, all in the same single route. The intention is to see how the routers scale with the number of parameters. The values of the parameters must be passed to the handler function somehow, which requires allocations. Let's see how clever the routers solve this task with a route containing 5 and 20 parameters:
```
BenchmarkAce_Param5                      5000000               294 ns/op             160 B/op          1 allocs/op
BenchmarkBear_Param5                     1000000              1038 ns/op             501 B/op          5 allocs/op
BenchmarkBeego_Param5                    1000000              1137 ns/op             352 B/op          3 allocs/op
BenchmarkBone_Param5                     1000000              1608 ns/op             864 B/op          6 allocs/op
BenchmarkChi_Param5                      2000000               865 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_Param5                    5000000               324 ns/op             160 B/op          1 allocs/op
BenchmarkEcho_Param5                     5000000               283 ns/op              32 B/op          1 allocs/op
BenchmarkGin_Param5                     20000000                93.0 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_Param5               1000000              1761 ns/op             920 B/op         11 allocs/op
BenchmarkGoji_Param5                     2000000               808 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_Param5                   1000000              2724 ns/op            1392 B/op         11 allocs/op
BenchmarkGoJsonRest_Param5               1000000              2280 ns/op            1097 B/op         16 allocs/op
BenchmarkGoRestful_Param5                 300000              4477 ns/op            2392 B/op         21 allocs/op
BenchmarkGorillaMux_Param5                500000              3530 ns/op            1328 B/op         10 allocs/op
BenchmarkHttpRouter_Param5               5000000               320 ns/op             160 B/op          1 allocs/op
BenchmarkHttpTreeMux_Param5              1000000              1340 ns/op             576 B/op          6 allocs/op
BenchmarkKocha_Param5                    2000000               954 ns/op             440 B/op         10 allocs/op
BenchmarkLARS_Param5                    20000000                94.7 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_Param5                  1000000              2224 ns/op            1056 B/op         10 allocs/op
BenchmarkMartini_Param5                   300000              4829 ns/op            1232 B/op         11 allocs/op
BenchmarkPat_Param5                       500000              3385 ns/op             964 B/op         32 allocs/op
BenchmarkPossum_Param5                   1000000              1060 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_Param5                 2000000               849 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_Param5                    1000000              1118 ns/op             528 B/op          9 allocs/op
BenchmarkTango_Param5                    1000000              1867 ns/op             936 B/op         16 allocs/op
BenchmarkTigerTonic_Param5                300000              6990 ns/op            2551 B/op         43 allocs/op
BenchmarkTraffic_Param5                   300000              5230 ns/op            2248 B/op         25 allocs/op
BenchmarkVulcan_Param5                   3000000               566 ns/op              98 B/op          3 allocs/op
```

```
BenchmarkAce_Param20                     1000000              1342 ns/op             640 B/op          1 allocs/op
BenchmarkBear_Param20                     500000              3372 ns/op            1665 B/op          5 allocs/op
BenchmarkBeego_Param20                   1000000              2324 ns/op             352 B/op          3 allocs/op
BenchmarkBone_Param20                     500000              3977 ns/op            2031 B/op          6 allocs/op
BenchmarkChi_Param20                     1000000              1513 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_Param20                   1000000              1295 ns/op             640 B/op          1 allocs/op
BenchmarkEcho_Param20                    3000000               583 ns/op              32 B/op          1 allocs/op
BenchmarkGin_Param20                    10000000               242 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_Param20               300000              7696 ns/op            3795 B/op         15 allocs/op
BenchmarkGoji_Param20                     500000              2623 ns/op            1247 B/op          2 allocs/op
BenchmarkGojiv2_Param20                   500000              3332 ns/op            1632 B/op         11 allocs/op
BenchmarkGoJsonRest_Param20               200000              8524 ns/op            4484 B/op         20 allocs/op
BenchmarkGoRestful_Param20                200000              8900 ns/op            4723 B/op         23 allocs/op
BenchmarkGorillaMux_Param20               200000              7595 ns/op            3436 B/op         12 allocs/op
BenchmarkHttpRouter_Param20              1000000              1409 ns/op             640 B/op          1 allocs/op
BenchmarkHttpTreeMux_Param20              200000              6450 ns/op            3195 B/op         10 allocs/op
BenchmarkKocha_Param20                    500000              3786 ns/op            1808 B/op         27 allocs/op
BenchmarkLARS_Param20                   10000000               222 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_Param20                  300000              5606 ns/op            2908 B/op         12 allocs/op
BenchmarkMartini_Param20                  200000              9478 ns/op            3596 B/op         13 allocs/op
BenchmarkPat_Param20                      100000             14462 ns/op            4687 B/op        111 allocs/op
BenchmarkPossum_Param20                  1000000              1054 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_Param20                 500000              4624 ns/op            2283 B/op          7 allocs/op
BenchmarkRivet_Param20                    300000              5020 ns/op            2620 B/op         26 allocs/op
BenchmarkTango_Param20                    200000             16340 ns/op            8216 B/op         46 allocs/op
BenchmarkTigerTonic_Param20                50000             27464 ns/op           10533 B/op        138 allocs/op
BenchmarkTraffic_Param20                  100000             17749 ns/op            7942 B/op         45 allocs/op
BenchmarkVulcan_Param20                  2000000               968 ns/op              98 B/op          3 allocs/op
```

Now let's see how expensive it is to access a parameter. The handler function reads the value (by the name of the parameter, e.g. with a map lookup; depends on the router) and writes it to our [web scale storage](https://www.youtube.com/watch?v=b2F-DItXtZs) (`/dev/null`).
```
BenchmarkAce_ParamWrite                 10000000               193 ns/op              40 B/op          2 allocs/op
BenchmarkBear_ParamWrite                 2000000               868 ns/op             456 B/op          5 allocs/op
BenchmarkBeego_ParamWrite                2000000               936 ns/op             360 B/op          4 allocs/op
BenchmarkBone_ParamWrite                 1000000              1545 ns/op             816 B/op          6 allocs/op
BenchmarkChi_ParamWrite                  2000000               792 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_ParamWrite               10000000               152 ns/op              32 B/op          1 allocs/op
BenchmarkEcho_ParamWrite                 5000000               254 ns/op              40 B/op          2 allocs/op
BenchmarkGin_ParamWrite                 20000000               117 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_ParamWrite           1000000              1230 ns/op             656 B/op          9 allocs/op
BenchmarkGoji_ParamWrite                 2000000               655 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_ParamWrite               1000000              2689 ns/op            1360 B/op         13 allocs/op
BenchmarkGoJsonRest_ParamWrite            500000              2347 ns/op            1128 B/op         18 allocs/op
BenchmarkGoRestful_ParamWrite             300000              4327 ns/op            2304 B/op         22 allocs/op
BenchmarkGorillaMux_ParamWrite           1000000              2398 ns/op            1264 B/op         10 allocs/op
BenchmarkHttpRouter_ParamWrite          20000000               117 ns/op              32 B/op          1 allocs/op
BenchmarkHttpTreeMux_ParamWrite          2000000               792 ns/op             352 B/op          3 allocs/op
BenchmarkKocha_ParamWrite                5000000               241 ns/op              56 B/op          3 allocs/op
BenchmarkLARS_ParamWrite                20000000               109 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_ParamWrite              1000000              2389 ns/op            1160 B/op         14 allocs/op
BenchmarkMartini_ParamWrite               300000              4455 ns/op            1176 B/op         14 allocs/op
BenchmarkPat_ParamWrite                  1000000              2189 ns/op            1072 B/op         17 allocs/op
BenchmarkPossum_ParamWrite               1000000              1067 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_ParamWrite             2000000               970 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_ParamWrite                1000000              1001 ns/op             472 B/op          6 allocs/op
BenchmarkTango_ParamWrite                3000000               471 ns/op             136 B/op          4 allocs/op
BenchmarkTigerTonic_ParamWrite            500000              3192 ns/op            1424 B/op         23 allocs/op
BenchmarkTraffic_ParamWrite               500000              4714 ns/op            2384 B/op         25 allocs/op
BenchmarkVulcan_ParamWrite               3000000               430 ns/op              98 B/op          3 allocs/op
```

### [Parse.com](https://parse.com/docs/rest#summary)

Enough of the micro benchmark stuff. Let's play a bit with real APIs. In the first set of benchmarks, we use a clone of the structure of [Parse](https://parse.com)'s decent medium-sized REST API, consisting of 26 routes.

The tasks are 1.) routing a static URL (no parameters), 2.) routing a URL containing 1 parameter, 3.) same with 2 parameters, 4.) route all of the routes once (like the StaticAll benchmark, but the routes now contain parameters).

Worth noting is, that the requested route might be a good case for some routing algorithms, while it is a bad case for another algorithm. The values might vary slightly depending on the selected route.

```
BenchmarkAce_ParseStatic                20000000                62.9 ns/op             0 B/op          0 allocs/op
BenchmarkBear_ParseStatic                3000000               335 ns/op             120 B/op          3 allocs/op
BenchmarkBeego_ParseStatic               2000000               831 ns/op             352 B/op          3 allocs/op
BenchmarkBone_ParseStatic                3000000               567 ns/op             144 B/op          3 allocs/op
BenchmarkChi_ParseStatic                 2000000               763 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_ParseStatic              50000000                28.6 ns/op             0 B/op          0 allocs/op
BenchmarkEcho_ParseStatic               10000000               172 ns/op              32 B/op          1 allocs/op
BenchmarkGin_ParseStatic                30000000                51.1 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_ParseStatic          2000000               577 ns/op             296 B/op          5 allocs/op
BenchmarkGoji_ParseStatic               10000000               152 ns/op               0 B/op          0 allocs/op
BenchmarkGojiv2_ParseStatic              1000000              2003 ns/op            1312 B/op         10 allocs/op
BenchmarkGoJsonRest_ParseStatic          2000000               872 ns/op             329 B/op         11 allocs/op
BenchmarkGoRestful_ParseStatic            300000              6525 ns/op            3224 B/op         22 allocs/op
BenchmarkGorillaMux_ParseStatic          1000000              2313 ns/op             960 B/op          9 allocs/op
BenchmarkHttpRouter_ParseStatic         50000000                23.8 ns/op             0 B/op          0 allocs/op
BenchmarkHttpTreeMux_ParseStatic        30000000                49.0 ns/op             0 B/op          0 allocs/op
BenchmarkKocha_ParseStatic              30000000                41.5 ns/op             0 B/op          0 allocs/op
BenchmarkLARS_ParseStatic               30000000                51.8 ns/op             0 B/op          0 allocs/op
BenchmarkMartini_ParseStatic              500000              3708 ns/op             768 B/op          9 allocs/op
BenchmarkPat_ParseStatic                 2000000               853 ns/op             240 B/op          5 allocs/op
BenchmarkPossum_ParseStatic              2000000               789 ns/op             416 B/op          3 allocs/op
BenchmarkR2router_ParseStatic            5000000               371 ns/op             144 B/op          4 allocs/op
BenchmarkRivet_ParseStatic               5000000               274 ns/op             112 B/op          2 allocs/op
BenchmarkTango_ParseStatic               2000000               746 ns/op             248 B/op          8 allocs/op
BenchmarkTigerTonic_ParseStatic         10000000               195 ns/op              48 B/op          1 allocs/op
BenchmarkTraffic_ParseStatic              500000              4693 ns/op            2328 B/op         24 allocs/op
BenchmarkVulcan_ParseStatic              3000000               471 ns/op              98 B/op          3 allocs/op

BenchmarkAce_ParseParam                 10000000               173 ns/op              64 B/op          1 allocs/op
BenchmarkBear_ParseParam                 2000000               959 ns/op             467 B/op          5 allocs/op
BenchmarkBeego_ParseParam                2000000               874 ns/op             352 B/op          3 allocs/op
BenchmarkBone_ParseParam                 1000000              1701 ns/op             896 B/op          7 allocs/op
BenchmarkChi_ParseParam                  2000000               826 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_ParseParam               10000000               179 ns/op              64 B/op          1 allocs/op
BenchmarkEcho_ParseParam                10000000               198 ns/op              32 B/op          1 allocs/op
BenchmarkGin_ParseParam                 30000000                60.9 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_ParseParam           1000000              1222 ns/op             664 B/op          8 allocs/op
BenchmarkGoji_ParseParam                 2000000               685 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_ParseParam               1000000              2673 ns/op            1360 B/op         12 allocs/op
BenchmarkGoJsonRest_ParseParam           1000000              1317 ns/op             649 B/op         13 allocs/op
BenchmarkGoRestful_ParseParam             300000              7084 ns/op            3544 B/op         23 allocs/op
BenchmarkGorillaMux_ParseParam           1000000              2480 ns/op            1264 B/op         10 allocs/op
BenchmarkHttpRouter_ParseParam          10000000               126 ns/op              64 B/op          1 allocs/op
BenchmarkHttpTreeMux_ParseParam          2000000               707 ns/op             352 B/op          3 allocs/op
BenchmarkKocha_ParseParam               10000000               210 ns/op              56 B/op          3 allocs/op
BenchmarkLARS_ParseParam                20000000                63.9 ns/op             0 B/op          0 allocs/op
BenchmarkMartini_ParseParam               500000              4177 ns/op            1072 B/op         10 allocs/op
BenchmarkPat_ParseParam                  1000000              2233 ns/op            1120 B/op         17 allocs/op
BenchmarkPossum_ParseParam               1000000              1006 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_ParseParam             2000000               877 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_ParseParam                2000000               989 ns/op             464 B/op          5 allocs/op
BenchmarkTango_ParseParam                2000000               785 ns/op             280 B/op          8 allocs/op
BenchmarkTigerTonic_ParseParam           1000000              2175 ns/op            1008 B/op         17 allocs/op
BenchmarkTraffic_ParseParam               500000              4490 ns/op            2248 B/op         23 allocs/op
BenchmarkVulcan_ParseParam               3000000               505 ns/op              98 B/op          3 allocs/op

BenchmarkAce_Parse2Params               10000000               176 ns/op              64 B/op          1 allocs/op
BenchmarkBear_Parse2Params               2000000              1036 ns/op             496 B/op          5 allocs/op
BenchmarkBeego_Parse2Params              2000000               925 ns/op             352 B/op          3 allocs/op
BenchmarkBone_Parse2Params               1000000              1618 ns/op             848 B/op          6 allocs/op
BenchmarkChi_Parse2Params                2000000               820 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_Parse2Params             10000000               183 ns/op              64 B/op          1 allocs/op
BenchmarkEcho_Parse2Params              10000000               215 ns/op              32 B/op          1 allocs/op
BenchmarkGin_Parse2Params               20000000                68.1 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_Parse2Params         1000000              1353 ns/op             712 B/op          9 allocs/op
BenchmarkGoji_Parse2Params               2000000               640 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_Parse2Params             1000000              2643 ns/op            1344 B/op         11 allocs/op
BenchmarkGoJsonRest_Parse2Params         1000000              1422 ns/op             713 B/op         14 allocs/op
BenchmarkGoRestful_Parse2Params           200000             11946 ns/op            6008 B/op         25 allocs/op
BenchmarkGorillaMux_Parse2Params          500000              2634 ns/op            1280 B/op         10 allocs/op
BenchmarkHttpRouter_Parse2Params        10000000               133 ns/op              64 B/op          1 allocs/op
BenchmarkHttpTreeMux_Parse2Params        2000000               784 ns/op             384 B/op          4 allocs/op
BenchmarkKocha_Parse2Params              3000000               383 ns/op             128 B/op          5 allocs/op
BenchmarkLARS_Parse2Params              20000000                76.7 ns/op             0 B/op          0 allocs/op
BenchmarkMartini_Parse2Params             500000              4323 ns/op            1152 B/op         11 allocs/op
BenchmarkPat_Parse2Params                1000000              2041 ns/op             832 B/op         17 allocs/op
BenchmarkPossum_Parse2Params             2000000              1047 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_Parse2Params           2000000               906 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_Parse2Params              2000000               939 ns/op             480 B/op          6 allocs/op
BenchmarkTango_Parse2Params              1000000              1007 ns/op             408 B/op         10 allocs/op
BenchmarkTigerTonic_Parse2Params          500000              3321 ns/op            1408 B/op         24 allocs/op
BenchmarkTraffic_Parse2Params             500000              3880 ns/op            2040 B/op         22 allocs/op
BenchmarkVulcan_Parse2Params             3000000               543 ns/op              98 B/op          3 allocs/op

BenchmarkAce_ParseAll                     500000              3361 ns/op             640 B/op         16 allocs/op
BenchmarkBear_ParseAll                    100000             17105 ns/op            8928 B/op        110 allocs/op
BenchmarkBeego_ParseAll                   100000             21468 ns/op            9152 B/op         78 allocs/op
BenchmarkBone_ParseAll                     50000             31492 ns/op           16208 B/op        147 allocs/op
BenchmarkChi_ParseAll                     100000             21321 ns/op           11232 B/op         78 allocs/op
BenchmarkDenco_ParseAll                   500000              3428 ns/op             928 B/op         16 allocs/op
BenchmarkEcho_ParseAll                    300000              5838 ns/op             832 B/op         26 allocs/op
BenchmarkGin_ParseAll                    1000000              1659 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_ParseAll               50000             26776 ns/op           13728 B/op        181 allocs/op
BenchmarkGoji_ParseAll                    100000             12411 ns/op            5376 B/op         32 allocs/op
BenchmarkGojiv2_ParseAll                   30000             64157 ns/op           34448 B/op        277 allocs/op
BenchmarkGoJsonRest_ParseAll               50000             30250 ns/op           13866 B/op        321 allocs/op
BenchmarkGoRestful_ParseAll                10000            227524 ns/op          110928 B/op        600 allocs/op
BenchmarkGorillaMux_ParseAll               20000             86824 ns/op           29872 B/op        250 allocs/op
BenchmarkHttpRouter_ParseAll             1000000              2201 ns/op             640 B/op         16 allocs/op
BenchmarkHttpTreeMux_ParseAll             200000             11992 ns/op            5728 B/op         51 allocs/op
BenchmarkKocha_ParseAll                   300000              5078 ns/op            1112 B/op         54 allocs/op
BenchmarkLARS_ParseAll                   1000000              1705 ns/op               0 B/op          0 allocs/op
BenchmarkMartini_ParseAll                  10000            108746 ns/op           25072 B/op        253 allocs/op
BenchmarkPat_ParseAll                      30000             46421 ns/op           17264 B/op        343 allocs/op
BenchmarkPossum_ParseAll                  100000             19939 ns/op           10816 B/op         78 allocs/op
BenchmarkR2router_ParseAll                100000             15694 ns/op            8352 B/op        120 allocs/op
BenchmarkRivet_ParseAll                   100000             15853 ns/op            8592 B/op        103 allocs/op
BenchmarkTango_ParseAll                   100000             22460 ns/op            7456 B/op        214 allocs/op
BenchmarkTigerTonic_ParseAll               30000             46752 ns/op           19728 B/op        379 allocs/op
BenchmarkTraffic_ParseAll                  10000            121399 ns/op           68416 B/op        705 allocs/op
BenchmarkVulcan_ParseAll                  100000             14564 ns/op            2548 B/op         78 allocs/op```
```


### [GitHub](http://developer.github.com/v3/)

The GitHub API is rather large, consisting of 203 routes. The tasks are basically the same as in the benchmarks before.

```
BenchmarkAce_GithubStatic               20000000                65.8 ns/op             0 B/op          0 allocs/op
BenchmarkBear_GithubStatic               5000000               366 ns/op             120 B/op          3 allocs/op
BenchmarkBeego_GithubStatic              2000000               911 ns/op             352 B/op          3 allocs/op
BenchmarkBone_GithubStatic                200000             10770 ns/op            2880 B/op         60 allocs/op
BenchmarkChi_GithubStatic                2000000               834 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_GithubStatic             50000000                29.3 ns/op             0 B/op          0 allocs/op
BenchmarkEcho_GithubStatic              10000000               188 ns/op              32 B/op          1 allocs/op
BenchmarkGin_GithubStatic               20000000                75.3 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_GithubStatic         2000000               657 ns/op             296 B/op          5 allocs/op
BenchmarkGoji_GithubStatic              10000000               172 ns/op               0 B/op          0 allocs/op
BenchmarkGojiv2_GithubStatic             1000000              2658 ns/op            1312 B/op         10 allocs/op
BenchmarkGoRestful_GithubStatic           100000             12462 ns/op            3224 B/op         22 allocs/op
BenchmarkGoJsonRest_GithubStatic         2000000               922 ns/op             329 B/op         11 allocs/op
BenchmarkGorillaMux_GithubStatic          200000             12084 ns/op             960 B/op          9 allocs/op
BenchmarkHttpRouter_GithubStatic        50000000                39.1 ns/op             0 B/op          0 allocs/op
BenchmarkHttpTreeMux_GithubStatic       30000000                48.9 ns/op             0 B/op          0 allocs/op
BenchmarkKocha_GithubStatic             30000000                53.0 ns/op             0 B/op          0 allocs/op
BenchmarkLARS_GithubStatic              20000000                71.0 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_GithubStatic            1000000              1761 ns/op             768 B/op          9 allocs/op
BenchmarkMartini_GithubStatic             200000             11725 ns/op             768 B/op          9 allocs/op
BenchmarkPat_GithubStatic                 200000             11638 ns/op            3648 B/op         76 allocs/op
BenchmarkPossum_GithubStatic             2000000               748 ns/op             416 B/op          3 allocs/op
BenchmarkR2router_GithubStatic           5000000               372 ns/op             144 B/op          4 allocs/op
BenchmarkRivet_GithubStatic              5000000               276 ns/op             112 B/op          2 allocs/op
BenchmarkTango_GithubStatic              2000000               882 ns/op             248 B/op          8 allocs/op
BenchmarkTigerTonic_GithubStatic        10000000               205 ns/op              48 B/op          1 allocs/op
BenchmarkTraffic_GithubStatic              50000             37326 ns/op           22776 B/op        166 allocs/op
BenchmarkVulcan_GithubStatic             2000000               659 ns/op              98 B/op          3 allocs/op

BenchmarkAce_GithubParam                10000000               243 ns/op              96 B/op          1 allocs/op
BenchmarkBear_GithubParam                2000000               916 ns/op             496 B/op          5 allocs/op
BenchmarkBeego_GithubParam               2000000               968 ns/op             352 B/op          3 allocs/op
BenchmarkBone_GithubParam                 200000              5832 ns/op            1888 B/op         19 allocs/op
BenchmarkChi_GithubParam                 2000000               789 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_GithubParam               5000000               278 ns/op             128 B/op          1 allocs/op
BenchmarkEcho_GithubParam                5000000               265 ns/op              32 B/op          1 allocs/op
BenchmarkGin_GithubParam                20000000               117 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_GithubParam          1000000              1236 ns/op             712 B/op          9 allocs/op
BenchmarkGoji_GithubParam                2000000               822 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_GithubParam              1000000              2141 ns/op            1408 B/op         13 allocs/op
BenchmarkGoJsonRest_GithubParam          1000000              1618 ns/op             713 B/op         14 allocs/op
BenchmarkGoRestful_GithubParam            100000             13575 ns/op            2360 B/op         21 allocs/op
BenchmarkGorillaMux_GithubParam           200000              7470 ns/op            1280 B/op         10 allocs/op
BenchmarkHttpRouter_GithubParam         10000000               204 ns/op              96 B/op          1 allocs/op
BenchmarkHttpTreeMux_GithubParam         2000000               789 ns/op             384 B/op          4 allocs/op
BenchmarkKocha_GithubParam               3000000               433 ns/op             128 B/op          5 allocs/op
BenchmarkLARS_GithubParam               20000000               107 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_GithubParam             1000000              2007 ns/op            1056 B/op         10 allocs/op
BenchmarkMartini_GithubParam              200000              8886 ns/op            1152 B/op         11 allocs/op
BenchmarkPat_GithubParam                  200000              8439 ns/op            2464 B/op         48 allocs/op
BenchmarkPossum_GithubParam              1000000              1013 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_GithubParam            2000000               719 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_GithubParam               2000000               905 ns/op             480 B/op          6 allocs/op
BenchmarkTango_GithubParam               1000000              1190 ns/op             472 B/op         11 allocs/op
BenchmarkTigerTonic_GithubParam           500000              3343 ns/op            1440 B/op         24 allocs/op
BenchmarkTraffic_GithubParam              100000             13535 ns/op            6952 B/op         56 allocs/op
BenchmarkVulcan_GithubParam              1000000              1031 ns/op              98 B/op          3 allocs/op

BenchmarkAce_GithubAll                     30000             42324 ns/op           13792 B/op        167 allocs/op
BenchmarkBear_GithubAll                    10000            178683 ns/op           86448 B/op        943 allocs/op
BenchmarkBeego_GithubAll                   10000            204920 ns/op           71456 B/op        609 allocs/op
BenchmarkBone_GithubAll                     1000           2268964 ns/op          720160 B/op       8620 allocs/op
BenchmarkChi_GithubAll                     10000            163773 ns/op           87696 B/op        609 allocs/op
BenchmarkDenco_GithubAll                   30000             49109 ns/op           20224 B/op        167 allocs/op
BenchmarkEcho_GithubAll                    30000             54878 ns/op            6496 B/op        203 allocs/op
BenchmarkGin_GithubAll                    100000             23876 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_GithubAll              10000            252232 ns/op          131656 B/op       1686 allocs/op
BenchmarkGoji_GithubAll                     5000            356992 ns/op           56112 B/op        334 allocs/op
BenchmarkGojiv2_GithubAll                   2000            653979 ns/op          352720 B/op       4321 allocs/op
BenchmarkGoJsonRest_GithubAll              10000            311841 ns/op          134371 B/op       2737 allocs/op
BenchmarkGoRestful_GithubAll                 500           2683671 ns/op          689672 B/op       4519 allocs/op
BenchmarkGorillaMux_GithubAll                300           4406573 ns/op          248400 B/op       1994 allocs/op
BenchmarkHttpRouter_GithubAll              50000             35709 ns/op           13792 B/op        167 allocs/op
BenchmarkHttpTreeMux_GithubAll             10000            140273 ns/op           65856 B/op        671 allocs/op
BenchmarkKocha_GithubAll                   20000             86897 ns/op           23304 B/op        843 allocs/op
BenchmarkLARS_GithubAll                   100000             22248 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_GithubAll                  5000            404849 ns/op          204193 B/op       2000 allocs/op
BenchmarkMartini_GithubAll                   500           3716036 ns/op          226547 B/op       2325 allocs/op
BenchmarkPat_GithubAll                       500           3622153 ns/op         1499568 B/op      27435 allocs/op
BenchmarkPossum_GithubAll                  10000            142150 ns/op           84448 B/op        609 allocs/op
BenchmarkR2router_GithubAll                10000            131614 ns/op           77328 B/op        979 allocs/op
BenchmarkRivet_GithubAll                   10000            152883 ns/op           84272 B/op       1079 allocs/op
BenchmarkTango_GithubAll                   10000            223634 ns/op           85451 B/op       2064 allocs/op
BenchmarkTigerTonic_GithubAll               2000            579981 ns/op          239104 B/op       5374 allocs/op
BenchmarkTraffic_GithubAll                   300           5392693 ns/op         3097761 B/op      23742 allocs/op
BenchmarkVulcan_GithubAll                  10000            165795 ns/op           19894 B/op        609 allocs/op
```

### [Google+](https://developers.google.com/+/api/latest/)

Last but not least the Google+ API, consisting of 13 routes. In reality this is just a subset of a much larger API.

```
BenchmarkAce_GPlusStatic                30000000                54.2 ns/op             0 B/op          0 allocs/op
BenchmarkBear_GPlusStatic                5000000               282 ns/op             104 B/op          3 allocs/op
BenchmarkBeego_GPlusStatic               2000000               809 ns/op             352 B/op          3 allocs/op
BenchmarkBone_GPlusStatic               10000000               147 ns/op              32 B/op          1 allocs/op
BenchmarkChi_GPlusStatic                 2000000               821 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_GPlusStatic              50000000                23.5 ns/op             0 B/op          0 allocs/op
BenchmarkEcho_GPlusStatic               10000000               165 ns/op              32 B/op          1 allocs/op
BenchmarkGin_GPlusStatic                30000000                53.1 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_GPlusStatic          3000000               583 ns/op             280 B/op          5 allocs/op
BenchmarkGoji_GPlusStatic               10000000               134 ns/op               0 B/op          0 allocs/op
BenchmarkGojiv2_GPlusStatic              1000000              2324 ns/op            1312 B/op         10 allocs/op
BenchmarkGoJsonRest_GPlusStatic          2000000               809 ns/op             329 B/op         11 allocs/op
BenchmarkGoRestful_GPlusStatic            500000              3905 ns/op            1976 B/op         20 allocs/op
BenchmarkGorillaMux_GPlusStatic          1000000              1768 ns/op             960 B/op          9 allocs/op
BenchmarkHttpRouter_GPlusStatic         50000000                24.2 ns/op             0 B/op          0 allocs/op
BenchmarkHttpTreeMux_GPlusStatic        50000000                31.7 ns/op             0 B/op          0 allocs/op
BenchmarkKocha_GPlusStatic              50000000                35.2 ns/op             0 B/op          0 allocs/op
BenchmarkLARS_GPlusStatic               30000000                52.4 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_GPlusStatic             1000000              1682 ns/op             768 B/op          9 allocs/op
BenchmarkMartini_GPlusStatic              500000              3560 ns/op             768 B/op          9 allocs/op
BenchmarkPat_GPlusStatic                 5000000               299 ns/op              96 B/op          2 allocs/op
BenchmarkPossum_GPlusStatic              2000000               823 ns/op             416 B/op          3 allocs/op
BenchmarkR2router_GPlusStatic            5000000               371 ns/op             144 B/op          4 allocs/op
BenchmarkRivet_GPlusStatic               5000000               265 ns/op             112 B/op          2 allocs/op
BenchmarkTango_GPlusStatic               2000000               701 ns/op             200 B/op          8 allocs/op
BenchmarkTigerTonic_GPlusStatic         10000000               136 ns/op              32 B/op          1 allocs/op
BenchmarkTraffic_GPlusStatic              500000              2761 ns/op            1464 B/op         18 allocs/op
BenchmarkVulcan_GPlusStatic              3000000               414 ns/op              98 B/op          3 allocs/op

BenchmarkAce_GPlusParam                 10000000               174 ns/op              64 B/op          1 allocs/op
BenchmarkBear_GPlusParam                 1000000              1003 ns/op             480 B/op          5 allocs/op
BenchmarkBeego_GPlusParam                2000000               904 ns/op             352 B/op          3 allocs/op
BenchmarkBone_GPlusParam                 1000000              1613 ns/op             816 B/op          6 allocs/op
BenchmarkChi_GPlusParam                  2000000               824 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_GPlusParam               10000000               167 ns/op              64 B/op          1 allocs/op
BenchmarkEcho_GPlusParam                10000000               195 ns/op              32 B/op          1 allocs/op
BenchmarkGin_GPlusParam                 20000000                70.3 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_GPlusParam           1000000              1281 ns/op             648 B/op          8 allocs/op
BenchmarkGoji_GPlusParam                 2000000               654 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_GPlusParam               1000000              2409 ns/op            1328 B/op         11 allocs/op
BenchmarkGoJsonRest_GPlusParam           1000000              1446 ns/op             649 B/op         13 allocs/op
BenchmarkGoRestful_GPlusParam             300000              4904 ns/op            2296 B/op         21 allocs/op
BenchmarkGorillaMux_GPlusParam            500000              3278 ns/op            1264 B/op         10 allocs/op
BenchmarkHttpRouter_GPlusParam          10000000               150 ns/op              64 B/op          1 allocs/op
BenchmarkHttpTreeMux_GPlusParam          2000000               741 ns/op             352 B/op          3 allocs/op
BenchmarkKocha_GPlusParam                5000000               239 ns/op              56 B/op          3 allocs/op
BenchmarkLARS_GPlusParam                20000000                73.2 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_GPlusParam              1000000              2152 ns/op            1056 B/op         10 allocs/op
BenchmarkMartini_GPlusParam               500000              4548 ns/op            1072 B/op         10 allocs/op
BenchmarkPat_GPlusParam                  1000000              1653 ns/op             688 B/op         12 allocs/op
BenchmarkPossum_GPlusParam               1000000              1065 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_GPlusParam             2000000               897 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_GPlusParam                2000000               977 ns/op             464 B/op          5 allocs/op
BenchmarkTango_GPlusParam                2000000               894 ns/op             264 B/op          8 allocs/op
BenchmarkTigerTonic_GPlusParam           1000000              2615 ns/op            1056 B/op         17 allocs/op
BenchmarkTraffic_GPlusParam               500000              4080 ns/op            1976 B/op         21 allocs/op
BenchmarkVulcan_GPlusParam               3000000               635 ns/op              98 B/op          3 allocs/op

BenchmarkAce_GPlus2Params               10000000               207 ns/op              64 B/op          1 allocs/op
BenchmarkBear_GPlus2Params               2000000               971 ns/op             496 B/op          5 allocs/op
BenchmarkBeego_GPlus2Params              1000000              1124 ns/op             352 B/op          3 allocs/op
BenchmarkBone_GPlus2Params               1000000              3265 ns/op            1168 B/op         10 allocs/op
BenchmarkChi_GPlus2Params                2000000               784 ns/op             432 B/op          3 allocs/op
BenchmarkDenco_GPlus2Params             10000000               216 ns/op              64 B/op          1 allocs/op
BenchmarkEcho_GPlus2Params               5000000               241 ns/op              32 B/op          1 allocs/op
BenchmarkGin_GPlus2Params               20000000                89.9 ns/op             0 B/op          0 allocs/op
BenchmarkGocraftWeb_GPlus2Params         1000000              1357 ns/op             712 B/op          9 allocs/op
BenchmarkGoji_GPlus2Params               2000000               830 ns/op             336 B/op          2 allocs/op
BenchmarkGojiv2_GPlus2Params              500000              2727 ns/op            1408 B/op         14 allocs/op
BenchmarkGoJsonRest_GPlus2Params         1000000              1687 ns/op             713 B/op         14 allocs/op
BenchmarkGoRestful_GPlus2Params           500000              4563 ns/op            2360 B/op         21 allocs/op
BenchmarkGorillaMux_GPlus2Params          200000              5971 ns/op            1280 B/op         10 allocs/op
BenchmarkHttpRouter_GPlus2Params        10000000               157 ns/op              64 B/op          1 allocs/op
BenchmarkHttpTreeMux_GPlus2Params        2000000               854 ns/op             384 B/op          4 allocs/op
BenchmarkKocha_GPlus2Params              3000000               446 ns/op             128 B/op          5 allocs/op
BenchmarkLARS_GPlus2Params              20000000                88.4 ns/op             0 B/op          0 allocs/op
BenchmarkMacaron_GPlus2Params            1000000              2118 ns/op            1056 B/op         10 allocs/op
BenchmarkMartini_GPlus2Params             200000              8678 ns/op            1200 B/op         13 allocs/op
BenchmarkPat_GPlus2Params                 300000              5863 ns/op            2256 B/op         34 allocs/op
BenchmarkPossum_GPlus2Params             1000000              1029 ns/op             544 B/op          5 allocs/op
BenchmarkR2router_GPlus2Params           2000000               908 ns/op             432 B/op          5 allocs/op
BenchmarkRivet_GPlus2Params              2000000              1001 ns/op             480 B/op          6 allocs/op
BenchmarkTango_GPlus2Params              1000000              1198 ns/op             440 B/op         10 allocs/op
BenchmarkTigerTonic_GPlus2Params          500000              3881 ns/op            1488 B/op         24 allocs/op
BenchmarkTraffic_GPlus2Params             200000              9056 ns/op            3512 B/op         32 allocs/op
BenchmarkVulcan_GPlus2Params             2000000               886 ns/op              98 B/op          3 allocs/op

BenchmarkAce_GPlusAll                    1000000              2257 ns/op             640 B/op         11 allocs/op
BenchmarkBear_GPlusAll                    200000             10945 ns/op            5488 B/op         61 allocs/op
BenchmarkBeego_GPlusAll                   100000             12854 ns/op            4576 B/op         39 allocs/op
BenchmarkBone_GPlusAll                     50000             25934 ns/op           11744 B/op        109 allocs/op
BenchmarkChi_GPlusAll                     200000             10325 ns/op            5616 B/op         39 allocs/op
BenchmarkDenco_GPlusAll                  1000000              2491 ns/op             672 B/op         11 allocs/op
BenchmarkEcho_GPlusAll                    500000              3169 ns/op             416 B/op         13 allocs/op
BenchmarkGin_GPlusAll                    2000000               958 ns/op               0 B/op          0 allocs/op
BenchmarkGocraftWeb_GPlusAll              100000             16196 ns/op            8040 B/op        103 allocs/op
BenchmarkGoji_GPlusAll                    200000              8572 ns/op            3696 B/op         22 allocs/op
BenchmarkGojiv2_GPlusAll                   50000             34081 ns/op           17616 B/op        154 allocs/op
BenchmarkGoJsonRest_GPlusAll              100000             19869 ns/op            8117 B/op        170 allocs/op
BenchmarkGoRestful_GPlusAll                20000             61555 ns/op           32024 B/op        275 allocs/op
BenchmarkGorillaMux_GPlusAll               30000             51982 ns/op           15904 B/op        128 allocs/op
BenchmarkHttpRouter_GPlusAll             1000000              1635 ns/op             640 B/op         11 allocs/op
BenchmarkHttpTreeMux_GPlusAll             200000              8034 ns/op            4032 B/op         38 allocs/op
BenchmarkKocha_GPlusAll                   300000              3733 ns/op             976 B/op         43 allocs/op
BenchmarkLARS_GPlusAll                   2000000               963 ns/op               0 B/op          0 allocs/op
BenchmarkMacaron_GPlusAll                  50000             28993 ns/op           13152 B/op        128 allocs/op
BenchmarkMartini_GPlusAll                  20000             72007 ns/op           14016 B/op        145 allocs/op
BenchmarkPat_GPlusAll                      30000             42265 ns/op           16576 B/op        298 allocs/op
BenchmarkPossum_GPlusAll                  200000             10632 ns/op            5408 B/op         39 allocs/op
BenchmarkR2router_GPlusAll                200000             10344 ns/op            5040 B/op         63 allocs/op
BenchmarkRivet_GPlusAll                   200000             10582 ns/op            5408 B/op         64 allocs/op
BenchmarkTango_GPlusAll                   100000             14624 ns/op            4200 B/op        116 allocs/op
BenchmarkTigerTonic_GPlusAll               50000             39714 ns/op           14512 B/op        288 allocs/op
BenchmarkTraffic_GPlusAll                  10000            104479 ns/op           40784 B/op        410 allocs/op
BenchmarkVulcan_GPlusAll                  200000              8839 ns/op            1274 B/op         39 allocs/op```
```


## Conclusions
First of all, there is no reason to use net/http's default [ServeMux](http://golang.org/pkg/net/http/#ServeMux), which is very limited and does not have especially good performance. There are enough alternatives coming in every flavor, choose the one you like best.

Secondly, the broad range of functions of some of the frameworks comes at a high price in terms of performance. For example Martini has great flexibility, but very bad performance. Martini has the worst performance of all tested routers in a lot of the benchmarks. Beego seems to have some scalability problems and easily defeats Martini with even worse performance, when the number of parameters or routes is high. I really hope, that the routing of these packages can be optimized. I think the Go-ecosystem needs great feature-rich frameworks like these.

Last but not least, we have to determine the performance champion.

Denco and its predecessor Kocha-urlrouter seem to have great performance, but are not convenient to use as a router for the net/http package. A lot of extra work is necessary to use it as a http.Handler. [The README of Denco claims](https://github.com/naoina/denco/blob/b03dbb499269a597afd0db715d408ebba1329d04/README.md), that the package is not intended as a replacement for [http.ServeMux](http://golang.org/pkg/net/http/#ServeMux).

[Goji](https://github.com/zenazn/goji/) looks very decent. It has great performance while also having a great range of features, more than any other router / framework in the top group.

Currently no router can beat the performance of the [HttpRouter](https://github.com/julienschmidt/httprouter) package, which currently dominates nearly all benchmarks.

In the end, performance can not be the (only) criterion for choosing a router. Play around a bit with some of the routers, and choose the one you like best.

## Usage

If you'd like to run these benchmarks locally, you'll need to install the package first:

```bash
go get github.com/julienschmidt/go-http-routing-benchmark
```
This may take a while due to the large number of dependencies that need to be downloaded. Once that command has finished you can run the full set of benchmarks like this:

```bash
cd $GOPATH/src/github.com/julienschmidt/go-http-routing-benchmark
go test -bench=.
```

> **Note:** If you run the tests and it SIGQUIT's make the go test timeout longer (#44)
>
>```
go test -timeout=2h -bench=.
```


You can bench specific frameworks only by using a regular expression as the value of the `bench` parameter:
```bash
go test -bench="Martini|Gin|HttpMux"
```
