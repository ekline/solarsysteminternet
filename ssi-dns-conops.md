# Solar System Internet DNS Concept of Operations

[Erik Kline](mailto:ek@aalyria.com)  
[Rick Taylor](mailto:rtaylor@aalyria.com)


# Motivation

As networked devices move further out into the Solar System there will be an
increase in the number of enclaves of synchronous connectivity using Internet
technologies that rely upon highly constrained links to reach other such
enclaves.  Within each enclave, users expect the full Internet experience
with low-latency access to local services, real-time applications, and
immediate resource discovery.  Reaching resources within other enclaves,
however, will generally involve significant delays and require different
asynchronous communication operations to inform and assist users and
applications.

As an example, consider a Mars-local Internet comprising several satellites,
rovers, and one or more research stations.  A researcher at one station
should be able to seamlessly initiate a audio call with a peer in another
Martian base, view real-time video from a rover relayed via Martian
satellites, and access local CDN-managed resources (perhaps video
entertainment pushed semi-regularly from Earth).  The same researcher, with
minimal alternation in work flow, should also be able to request data from
websites hosted on Earth, send email to a colleague within the same
organization (and ideally the same domain name) who is currently working at
a lunar base, and issue commands advancing a science mission to a satellite
orbiting Europa.  The former operations are all feasible with Internet
technologies today, the latter are largely possible with DTN network
architectures encapsulating traffic in BPv7 bundles (e.g. building a Bundle
Protocol overlay), however the user experience shifting between these two
modalities is not presently seamless.

This document outlines a hybrid architecture with some changes to DNS
infrastructure and operational practices to seamlessly bridge uses of
synchronous and asynchronous network connectivity.  DNS operations today
generally assume global reachability and relatively quick response times,
but DTN-connected enclaves will require new support from DNS (see below),
resolution strategies, and network signaling that can handle temporal
unavailability of authoritative servers, maintain coherent namespace views
across disconnected periods, and provide intelligent hints about resource
availability windows.  Additionally, applications and services need to be
designed with "enclave-aware" behaviors, automatically adapting their
functionality based on current connectivity state while maintaining
transparent user experiences that gracefully degrade when crossing enclave
boundaries.

# Requirements
* There should be minimal to no assumptions made about specific applications.
   * In general DNS has little to no knowledge about a given expectation,
     e.g. an A/AAAA lookup may be for a bespoke service, HTTPS, or even
     SMTP after an MX lookup.
* Browsers have shifted to preferring looking up HTTPS and SVCB Resource Resources (RR)s  
  * [RFC 9460](https://datatracker.ietf.org/doc/html/rfc9460)  
  * [RFC 9461](https://datatracker.ietf.org/doc/html/rfc9461)  
* These newer RRs provide a lot of context for applications:  
  * IPv4 and IPv6 addresses  
  * TLS ALPN  
  * Encrypted ClientHello parameters  
  * extensible set of parameters

# Caveats

This document is:
* a very rough sketch
* not battle-tested
* contains numerous **NOTE:** s sprinkled throughout that represent future
  work or discussion items

# Assumptions

* This document assumes the existence of Internet "enclaves", which are each
  a network of networks using Internet technologies, with synchronous
  Internet connectivity within its subgraph and DTN high-delay-adapted
  connectivity to one or more other such enclaves.
* an Internet enclave has a view of a DNS hierarchy, i.e. a DNS server
  architecture like Earth's Internet with access to root zone and
  hierarchical zone structure below that
* one or more DTN or high-delay-adapted connectivity paths to other enclaves
* extensions to existing RIR schemes work for allocation of IPv6 blocks,
  no different from Earth today

# REQs

* there remains a single DNS logically rooted at the current infrastructure
   * [RFC 2826](https://datatracker.ietf.org/doc/html/rfc2826) is presumed
   * this does not preclude the presence of root servers in non-Earth
     enclaves
* any DNS zone should be able to serve a reasonable tree in the local SSI
  network
* how DNS zone data is synchronized among enclaves is out of scope for this
  document, but presumed to be a matter solvable given experience with DNS
  operations in a multi-enclave Solar System Internet
* it should be possible to support CDNs locally present in an enclave,
  exactly as happens in Earth's enclave today
* any zone entry should be able to indicate the BPv7 and/or IP addresses of
  local servers, even if such servers act as Application Layer Gateways
  (ALGs) that attempt to handle high-delay communications
* the Control Plane info that programs any ALG servers is out of scope

# CONOPS for an arbitrary application

* Unspecified modern application looks up either an HTTPS or SVCB RR
* any domain owner can have their zone information synchronized out to any
  of the scoped Internets
  * **NOTE:** there is likely a Control Plane issue deciding how much zone
    data is synced from the zone owner to each of the enclaves
* If a DNS lookup doesn't find any RRs, application DNS error paths are executed  
  * DNSSEC NSEC/NSEC3 can be used to validate the absence of RRs or zones  
  * **NOTE:** possible extension to say "it's not available within this SSI"?
* If the scoped Internet DNS has an HTTPS/SVCB RR:  
  * **NOTE:** add a new HTTPS/SVCB extension that indicates the DTN Bundle
    EID hint in addition to the IP addresses and so on
  * get back **both** the Bundle EID hint and/or the IP addresses, and HTTPS
    parameters of a local proxy service (if HTTPS RR)
* If the IP addresses are reachable: great  
  * if not, the scoped Internet can maybe help operationally by giving ICMP
    errors to traffic that will otherwise leave the enclave
    * **NOTE:** perhaps a new Destination Unreachable subcode  
      * [RFC 4443 S3.1](https://datatracker.ietf.org/doc/html/rfc4443#section-3.1)
      * Destination Unreachable // "beyond scope of local Internet"  
* If the application speaks BPv7:
  * Application executes Bundle Protocol operations, whatever that means for
    itself (how to pack its application payload into Bundles)
    * e.g. [RFC 9292](https://datatracker.ietf.org/doc/html/rfc9292) binary HTTP messages  
  * Look up how to reach the DNS-advertised Bundle EID  
    * **NOTE:** this can also be stored in DNS, with various records for a
      DNS name transformation of the Bundle EID (e.g. something in ipn.arpa)
  * Establish a CLA connection to the BP Agent from the hint and transmit

# DNS extensions for CLA information BP EIDs

In general we should consider SVCB extensions for identifying everything you need to connect to a CLA

* no different from the HTTPS lookup  
* "BPv7 QUIC-CLA a/aaaa=..." where "QUIC_CLA" is possibly a new ALPN we request
* (insert even more unstructured thoughts here)…
