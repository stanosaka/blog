---
title: tools
date: 2020-08-09 14:15:15
tags:
---
<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [-](#-)
- [Multiplexers](#multiplexers)
    - [byobu](#byobu)
    - [mosh](#mosh)
    - [ss](#ss)
    - [mtr](#mtr)
    - [iftop](#iftop)
    - [iperf](#iperf)
    - [Troubleshooting layer 1 problems](#troubleshooting-layer-1-problems)
    - [HTTP core protocol](#http-core-protocol)
        - [Request](#request)
        - [Response](#response)
    - [cURL](#curl)
    - [netcat](#netcat)
        - [Basic client/server](#basic-clientserver)

<!-- markdown-toc end -->

# Multiplexers

## byobu

F2: Create a new windows.
F3: Go to the previous windows.
F4: Go to the next windows.
F9: help and configuration

Quitting a byobu session: locking and disconnecting.
F12: Lock access

F6: disconnect
byobu -r: reconnect
F7: scrollback, spacebar for mark the start of buffer, spacebar again to mark the end of selection.
Paste: Byobu-Escape + [

## mosh
```
mosh jdoe@10.10.10.101
mosh username@remoteserver.org --ssh="ssh -i ~/.ssh/identity_file -p 1234"
```

## ss
```
user@opsschool ~$ ss -tuln
Netid  State      Recv-Q Send-Q         Local Address:Port         Peer Address:Port
tcp    LISTEN     0      128                        *:80                      *:*
tcp    LISTEN     0      50                         *:4242                    *:*
tcp    LISTEN     0      50                        :::4242                   :::*
tcp    LISTEN     0      50                         *:2003                    *:*
tcp    LISTEN     0      50                         *:2004                    *:*
tcp    LISTEN     0      128                       :::22                     :::*
tcp    LISTEN     0      128                        *:22                      *:*
tcp    LISTEN     0      100                        *:3000                    *:*
tcp    LISTEN     0      100                      ::1:25                     :::*
tcp    LISTEN     0      100                127.0.0.1:25                      *:*
tcp    LISTEN     0      50                         *:7002                    *:*
```
* means the deamon is listening on all IP addresses the server might have.

127.0.0.1 means the daemon is listening only to the loopback interface

::: is the same thing as *, but for IPv6

::1 is the same as 127.0.0.1, but for IPv6

## mtr
is a program that combines the functionality of ping and traceroute into one utility
```
 mtr -r uow.edu.au
Start: 2020-08-09T15:14:55+1000
HOST: stan-OptiPlex-380           Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- router.lan                 0.0%    10    0.4   0.4   0.3   0.4   0.0
  2.|-- 58.162.27.77               0.0%    10    8.6  12.7   8.0  17.5   3.4
  3.|-- 10.2.2.150                 0.0%    10    8.6  13.7   8.3  29.7   6.6
  4.|-- 10.2.1.149                 0.0%    10   15.2  13.6  10.2  16.1   2.0
  5.|-- Bundle-Ether77.chw-edge90  0.0%    10   12.2  14.0  10.1  23.7   3.9
  6.|-- bundle-ether2.ken-edge903  0.0%    10   12.1  13.3   9.2  26.3   4.8
  7.|-- aar3533567.lnk.telstra.ne  0.0%    10   14.3  14.1  10.4  31.7   6.5
  8.|-- xe-0-2-1.pe1.rsby.nsw.aar  0.0%    10   12.7  12.0   8.9  14.8   2.2
  9.|-- 138.44.5.111               0.0%    10   10.2  14.5  10.2  17.5   2.5
 10.|-- 203.10.91.20               0.0%    10   11.8  16.6   9.9  27.2   6.1
 11.|-- www.uow.edu.au             0.0%    10   11.8  13.7  11.8  16.4   1.5
```
## iftop
Displays bandwith usage on a specific interface, broken down by remote host
```
sudo iftop -i eth0
```
## iperf
a bandwidth testing utility. It consists of a daemon and client, running on separate machines.
```
sudo iperf3 -c remote-host
```
## Troubleshooting layer 1 problems
```
ip -s link show eth0
sudo ethtool eth0
## -S, which shows all the same metrics as the ip command, plus many, many more
sudo ethtool -S eth0
```
## HTTP core protocol
### Request
| Method    | Description                                                                            |
| --------- | :-----------------------------:                                                        |
| GET       | Tranfers a current representation of the target resource.                              |
| HEAD      | Same as GET, but only transfer the status line and header section.                     |
| POST      | Perform resource-specific processing on the request payload.                           |
| PUT       | Replace all current representations of the target resource with the requested payload. |
| DELETE    | Remove all current representations of the target resource.                             |
| CONNECT   | Establish a tunnel to the server identified by the target resource.                    |
| OPTIONS   | Describe the communication options for the target resource.                            |
| TRACE     | Perform a message loop-back test along the path to the target resource.                |

### Response
|      Code | Reason                          | Description                                                                                                                                                                |
| --------- | :-----------------------------: |                                                                                                                                                                            |
|       200 | OK                              | The request has succeeded.                                                                                                                                                 |
|       301 | Moved Permenantly               | The target has moved to a new location and future references should use a new location. The server should mention the new location in the Location header in the response. |
|       302 | Found                           | The target resources been temporarily moved. As the move is temporarily, future references should still use the same location.                                             |
|       400 | Bad Request                     | The server cannot process the request as it is perceived as invalid.                                                                                                       |
|       401 | Unauthorized                    | The request to the target resource is not allowed due to missing or incorrect authentication credentials.                                                                  |
|       403 | Forbidden                       | The request to the target resource is not allowed for reasons unrelated to authentication.                                                                                 |
|       404 | Not Found                       | The target resource was not found.                                                                                                                                         |
|       500 | Internal Server Error           | The server encountered an unexpected error.                                                                                                                                |
|       502 | Bad Gateway                     | The server is acting as a gateway/proxy and received an invalid response from a server it contacted to fulfill the request.                                                |
|       503 | Service Unavailable             | The server is currently unable to fulfill the request.                                                                                                                     |

## cURL
is a tool and library for transferring data with URL syntax, capable of handling various protocols.
```
curl http://foobar/index.html
With the --request or -X parameter, the method can be specified. To include the the headers in the output.
curl -I --request GET  http://www.opsschool.org/en/latest/http_101.html
The -O switch makes cURL write output to a local file named like the remote file.
$ curl -O http://localhost/bigfile
```
By default cURL does not follow HTTP redirects, and instead a 3xx redirection message is given.
The -L switch makes cURL automatically follow redirects and issue another request to the given localtion till it finds the targeted resource.
```
curl -IL foobar.org
```
## netcat
a networking tool capable of reading and writing data across a network connection.
### Basic client/server
```
$ nc -l 192.168.0.1 1234 > output.log
### then in another terminal, pipe some data to netcat that is connected to the destination ip and port
$ echo "I'm connected as client"| nc 192.168.0.1 1234
## Then again in the first (server) terminal, the data is displayed:
$ cat output.log
I'm connected as client
```
