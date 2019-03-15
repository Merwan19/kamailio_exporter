# Kamailio Exporter for Prometheus

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/pascomnet/kamailio_exporter/blob/master/LICENSE)
[![CircleCI](https://circleci.com/gh/pascomnet/kamailio_exporter.svg?style=shield)](https://circleci.com/gh/pascomnet/kamailio_exporter)
[![Go Report Card](https://goreportcard.com/badge/github.com/pascomnet/kamailio_exporter)](https://goreportcard.com/report/github.com/pascomnet/kamailio_exporter)

A [Kamailio](https://www.kamailio.org/) exporter for Prometheus.
It exports a range of core and often used module statistics as well as scripted metrics. 

## Kamailio configuration

The Exporter needs a [BINRPC](http://kamailio.org/docs/modules/stable/modules/ctl.html) connection to Kamailio.
So you have to load and configure the [CTL](http://kamailio.org/docs/modules/stable/modules/ctl.html) module. 

If you run the Exporter and Kamailio on the same Machine, it's recommended to use a unix socket for the connection.
The path for the socket defaults to "unix:/var/run/kamailio/kamailio_ctl" and can be used out of the box.

Depending on your deployment, you might want to open a tcp socket on a *private or firewalled* interface.
This allows you in example to run the Exporter as a Sidecar to your Kamailio Container in a Dockerized environment.

```
modparam("ctl", "binrpc", "tcp:192.168.1.10:2046")
```

## Running the Exporter

Download or build the kamailio_exporter binary and start it. If you do so, it'll try to reach Kamailio on the default
unix domain socket /var/run/kamailio/kamailio_ctl. Metrics are exposed on all available interfaces on port 9494 and 
http path "/metrics". 

### Options

You can configure some things by using command line options or docker/systemd-friendly environment variables:

#### Connecting to Kamailio

  * --socketPath=/some/path :  Path to Kamailio unix domain socket (default: "/var/run/kamailio/kamailio_ctl") (env variable: SOCKET_PATH)
  * --host=1.2.3.4 :     Kamailio ip or hostname. Domain socket is used if no host is defined. (env variable: HOST)
  * --port=3012 :          Kamailio port (default: 3012) (env variable: PORT)
   
#### Expose metrics via http

  * --bindIp=127.0.0.1 :  Listen on this ip for scrape requests (default: "0.0.0.0") (env variable: BIND_IP)
  * --bindPort=9494 :     Listen on this port for scrape requests (default: 9494) (env variable: BIND_PORT)
  * --metricsPath=/metrics :   The http scrape path (default: "/metrics") (env variable: METRICS_PATH)

#### Misc

  * --debug :              Enable debug logging (env variable: DEBUG)

## Exported core and module metrics

Metrics are generated by running "stats.fetch all" RPC call.
Then a series of defined stats from core, shm, sl, tcp and tmx are turned into metrics. 

```
# HELP kamailio_bad_msg_hdr Messages with bad message header
# TYPE kamailio_bad_msg_hdr counter
kamailio_bad_msg_hdr 0
# HELP kamailio_bad_uri_total Messages with bad uri
# TYPE kamailio_bad_uri_total counter
kamailio_bad_uri_total 0
# HELP kamailio_core_rcv_reply_total Received replies by code
# TYPE kamailio_core_rcv_reply_total counter
kamailio_core_rcv_reply_total{code="18x"} 0
kamailio_core_rcv_reply_total{code="1xx"} 0
kamailio_core_rcv_reply_total{code="2xx"} 0
kamailio_core_rcv_reply_total{code="3xx"} 0
kamailio_core_rcv_reply_total{code="401"} 0
kamailio_core_rcv_reply_total{code="404"} 0
kamailio_core_rcv_reply_total{code="407"} 0
kamailio_core_rcv_reply_total{code="480"} 0
kamailio_core_rcv_reply_total{code="486"} 0
kamailio_core_rcv_reply_total{code="4xx"} 0
kamailio_core_rcv_reply_total{code="5xx"} 0
kamailio_core_rcv_reply_total{code="6xx"} 0
# HELP kamailio_core_rcv_request_total Received requests by method
# TYPE kamailio_core_rcv_request_total counter
kamailio_core_rcv_request_total{method="ack"} 0
kamailio_core_rcv_request_total{method="bye"} 0
kamailio_core_rcv_request_total{method="cancel"} 0
kamailio_core_rcv_request_total{method="info"} 0
kamailio_core_rcv_request_total{method="invite"} 0
kamailio_core_rcv_request_total{method="message"} 0
kamailio_core_rcv_request_total{method="notify"} 0
kamailio_core_rcv_request_total{method="options"} 0
kamailio_core_rcv_request_total{method="prack"} 0
kamailio_core_rcv_request_total{method="publish"} 0
kamailio_core_rcv_request_total{method="refer"} 0
kamailio_core_rcv_request_total{method="register"} 0
kamailio_core_rcv_request_total{method="subscribe"} 0
kamailio_core_rcv_request_total{method="unsupported"} 0
kamailio_core_rcv_request_total{method="update"} 0
# HELP kamailio_core_reply_total Reply counters
# TYPE kamailio_core_reply_total counter
kamailio_core_reply_total{type="drop"} 0
kamailio_core_reply_total{type="err"} 0
kamailio_core_reply_total{type="fwd"} 0
kamailio_core_reply_total{type="rcv"} 0
# HELP kamailio_core_request_total Request counters
# TYPE kamailio_core_request_total counter
kamailio_core_request_total{method="drop"} 0
kamailio_core_request_total{method="err"} 0
kamailio_core_request_total{method="fwd"} 0
kamailio_core_request_total{method="rcv"} 0
# HELP kamailio_dns_failed_request_total Failed dns requests
# TYPE kamailio_dns_failed_request_total counter
kamailio_dns_failed_request_total 0
# HELP kamailio_shm_bytes Shared memory sizes
# TYPE kamailio_shm_bytes gauge
kamailio_shm_bytes{type="free"} 6.3184376e+07
kamailio_shm_bytes{type="max_used"} 3.924488e+06
kamailio_shm_bytes{type="real_used"} 3.924488e+06
kamailio_shm_bytes{type="total"} 6.7108864e+07
kamailio_shm_bytes{type="used"} 3.030808e+06
# HELP kamailio_shm_fragments Shared memory fragment count
# TYPE kamailio_shm_fragments gauge
kamailio_shm_fragments 1
# HELP kamailio_sl_reply_total Stateless replies by code
# TYPE kamailio_sl_reply_total counter
kamailio_sl_reply_total{code="1xx"} 0
kamailio_sl_reply_total{code="200"} 0
kamailio_sl_reply_total{code="202"} 0
kamailio_sl_reply_total{code="2xx"} 0
kamailio_sl_reply_total{code="300"} 0
kamailio_sl_reply_total{code="301"} 0
kamailio_sl_reply_total{code="302"} 0
kamailio_sl_reply_total{code="3xx"} 0
kamailio_sl_reply_total{code="400"} 0
kamailio_sl_reply_total{code="401"} 0
kamailio_sl_reply_total{code="403"} 0
kamailio_sl_reply_total{code="404"} 0
kamailio_sl_reply_total{code="407"} 0
kamailio_sl_reply_total{code="408"} 0
kamailio_sl_reply_total{code="483"} 0
kamailio_sl_reply_total{code="4xx"} 0
kamailio_sl_reply_total{code="500"} 0
kamailio_sl_reply_total{code="5xx"} 0
kamailio_sl_reply_total{code="6xx"} 0
# HELP kamailio_sl_type_total Stateless replies by type
# TYPE kamailio_sl_type_total counter
kamailio_sl_type_total{type="failure"} 0
kamailio_sl_type_total{type="received_ack"} 0
kamailio_sl_type_total{type="sent_err_reply"} 0
kamailio_sl_type_total{type="sent_reply"} 0
kamailio_sl_type_total{type="xxx_reply"} 0
# HELP kamailio_tcp_connections Opened TCP connections
# TYPE kamailio_tcp_connections gauge
kamailio_tcp_connections 0
# HELP kamailio_tcp_total TCP connection counters
# TYPE kamailio_tcp_total counter
kamailio_tcp_total{type="con_reset"} 0
kamailio_tcp_total{type="con_timeout"} 0
kamailio_tcp_total{type="connect_failed"} 0
kamailio_tcp_total{type="connect_success"} 0
kamailio_tcp_total{type="established"} 0
kamailio_tcp_total{type="local_reject"} 0
kamailio_tcp_total{type="passive_open"} 0
kamailio_tcp_total{type="send_timeout"} 0
kamailio_tcp_total{type="sendq_full"} 0
# HELP kamailio_tcp_writequeue TCP write queue size
# TYPE kamailio_tcp_writequeue gauge
kamailio_tcp_writequeue 0
# HELP kamailio_tmx Ongoing Transactions
# TYPE kamailio_tmx gauge
kamailio_tmx{type="active"} 0
kamailio_tmx{type="inuse"} 0
# HELP kamailio_tmx_code_total Completed Transaction counters by code
# TYPE kamailio_tmx_code_total counter
kamailio_tmx_code_total{code="2xx"} 0
kamailio_tmx_code_total{code="3xx"} 0
kamailio_tmx_code_total{code="4xx"} 0
kamailio_tmx_code_total{code="5xx"} 0
kamailio_tmx_code_total{code="6xx"} 0
# HELP kamailio_tmx_rpl_total Tmx reply counters
# TYPE kamailio_tmx_rpl_total counter
kamailio_tmx_rpl_total{type="absorbed"} 0
kamailio_tmx_rpl_total{type="generated"} 0
kamailio_tmx_rpl_total{type="received"} 0
kamailio_tmx_rpl_total{type="relayed"} 0
kamailio_tmx_rpl_total{type="sent"} 0
# HELP kamailio_tmx_type_total Completed Transaction counters by type
# TYPE kamailio_tmx_type_total counter
kamailio_tmx_type_total{type="uac"} 0
kamailio_tmx_type_total{type="uas"} 0
```


## Scripted metrics

Often you might want to record some values from your own business logic. As usual in the Kamailio ecosystem,
there is already a module for this purpose: [statistics](http://kamailio.org/docs/modules/stable/modules/statistics.html) 

Statistics Module can be used both from Kamailio native scripts and all KEMI Languages, e.g. Lua or Python.

Configuration and usage is quite simple. Of course you need to load it:

```
loadmodule "statistics.so"
```

Next, all to-be-exported metrics have to be declared as statistic variable:

```
modparam("statistics", "variable", "my_custom_value_total")
```

Finally, in some route block, you have to populate the statistic variable with a value:

```
update_stat("my_custom_value_total", "+1");
```

A scraped metric will look like this:

```
# HELP kamailio_my_custom_value_totalScripted metric my_custom_value_total
# TYPE kamailio_my_custom_value_total counter
kamailio_my_custom_value_total 1
```

### Scripted metric details

* the statistic variable name is prefixed by "kamailio_" and changed to lower-case
* a suffix of "_total", "_seconds" or "_bytes" will export a Prometheus Counter, omitting the suffix produces a Prometheus Gauge, see [metric types](https://prometheus.io/docs/concepts/metric_types/).

## Building

Kamailio Exporter uses the [go module system](https://github.com/golang/go/wiki/Modules) and thus the minimum go version is 1.11.
Building the binary is straight forward:

1. clone or download the source code
1. change directory
1. go build -o kamailio_exporter


## Acknowledgements

Kudos to Florent Chauveau for his golang BINRPC implementation:  https://github.com/florentchauveau/go-kamailio-binrpc.
Also noteworthy is, that he provides an alternative implementation for scraping Kamailio statistics: https://github.com/florentchauveau/kamailio_exporter
