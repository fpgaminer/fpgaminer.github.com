---
layout: default
title: Everything You Need To Know To Code a Miner
---

This article will cover everything that you'll need to know to code mining software. We're going to deal strictly with what's required to write mining software that talks to a Mining Pool, because it's a little bit easier to understand, it's the most common usage of mining software, and mining solo isn't much different. In fact, any software written to work with a Mining Pool will also work as a solo miner (but perhaps not as efficiently).


**Overview**
Mining Bitcoins is accomplished by performing these steps:

	* Getting **work** from a Mining Pool using JSON-RPC.
	* Running calculations on that **work** using SHA-256.
	* Returning results to the Mining Pool using JSON-RPC.

The faster you can process **work** and return results, the more bitcoins you'll ultimately make.


**Getting work (HTTP and JSON-RPC)**
Getting work from a Mining Pool is *really* easy. All you need to do is make an HTTP request to the pool that looks like this:

POST / HTTP/1.1
Host: mining.eligius.st:8337
Accept-Encoding: identity
Content-Length: 44
Content-type: application/json
Authorization: Basic MUZaTVc3QkN6RXhzTG1FclQybzhvQ01MY01ZS3dkN3NIUTo=
User-Agent: Modular Python Bitcoin Miner v0.0.4alpha (bcjsonrpc.JSONRPCPool v0.0.1)

{"params": [], "method": "getwork", "id": 0}



This is just a normal HTTP request. There is Basic Authorization to give the pool the miner username/password. The only thing special about it is that we're POSTing some data; a JSON-RPC request. Getting work always uses the same request:

{"params": [], "method": "getwork", "id": 0}

So you don't need to understand too much about JSON or JSON-RPC. You can read about [http://en.wikipedia.org/wiki/Json JSON] and [http://en.wikipedia.org/wiki/JSON-RPC JSON-RPC] if you would like, though.


Assuming the username/password is correct, the Mining Pool will respond with something like the following:

HTTP/1.1 200 OK
Content-Length: 622
X-Roll-NTime: expire=120
X-Long-Polling: /LP
Server: Eloipool
Date: Thu, 29 Mar 2012 06:44:57 GMT
Content-Type: application/json

{"result": {"data": "00000001b105f79066f1130282bfa084569a15fcdc9155799fd2a3f0000000da00000000e5d46e948fdf71333f6f4341755db5c66481f3eacb2fa82eebc0630bfed72e7f4f7404e91a0a507e456c6f69000000800000000000000000000000000000000000000000000000000000000000000000000000000000000080020000", "hash1": "00000000000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000010000", "target": "ffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000", "midstate": "73a2e372c30f339cb41357488d0cdc86615abb8a2ac3f23fee959db08338acc5"}, "id": 0, "error": null}


Again, this is a standard HTTP response, with a few mining specific headers, and some JSON data. You should be able to quickly see that the pool has given us four important pieces of data:

data: 00000001b105f79066f1130282bfa084569a15fcdc9155799fd2a3f0000000da00000000e5d46e948fdf71333f6f4341755db5c66481f3eacb2fa82eebc0630bfed72e7f4f7404e91a0a507e456c6f69000000800000000000000000000000000000000000000000000000000000000000000000000000000000000080020000

hash1: 00000000000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000010000

target: ffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000

midstate: 73a2e372c30f339cb41357488d0cdc86615abb8a2ac3f23fee959db08338acc5


Together, all of this data represents a unit of **work** which, as mining software, we will use as input to some pretty intesive calculations.


#Work#
As seen in the previous section, we get four pieces of data from our mining pool.

**data**: This is actually the Bitcoin Block Header, and is the most important pieces of data.
