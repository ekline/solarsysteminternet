# Solar System Internet DNS Concept of Operations

[Erik Kline](mailto:ek@aalyria.com)  
[Rick Taylor](mailto:rtaylor@aalyria.com)


# Motivation

As networked devices move futher out into the Solar System there will be an increase in the number of enclaves of synchronous connectivity using Internet technologies that rely upon highly constrained links to reach other such enclaves.  Within each enclave, users expect the full Internet experience with low-latency access to local services, real-time applications, and immediate resource discovery.  Reaching resources within other enclaves, however, will generally involve significant delays and requires different asynchronous communication operations to inform and assist users and applications.

As an example, consider a Mars-local Internet comprising several satellites, rovers, and one or more research stations.  A researcher at one station should be able to seemlessly initiate a audio call with a peer in another Martian base, view real-time video from a rover relayed via Martian satellites, and access local CDN-managed resources.  The same researcher, with minimal alternation in work flow, should also be able to request data from websites hosted on Earth, send email to a colleague within the same organization (and ideally the same domain name) who is currently working at a lunar base, and issue commands advancing a science mission to a satellite orbiting Europa.  The former operations are all feasible with Internet technologies today, the latter are largely possible with DTN network archtictures encapsulating traffic in BPv7 bundles (building a Bundle Protocol overlay), however the user experience shifting between these two modalities is not presently seemless.

This document outlines a hybrid architecture with some changes to DNS infrastructure and operational practices to seamlessly bridge synchronous and asynchronous network connectivity.  DNS operations today generally assume global reachability and relatively quick response times, but DTN-connected enclaves will require new Resource Record types, resolution strategies, and network signaling that can handle temporal unavailability of authoritative servers, maintain coherent namespace views across disconnected periods, and provide intelligent hints about resource availability windows.  Additionally, applications and services need to be designed with "enclave-aware" behaviors, automatically adapting their functionality based on current connectivity state while maintaining transparent user experiences that gracefully degrade when crossing enclave boundaries.

# Requirements
* The application should be considered unknown because DNS generally doesn't know if a given A/AAAA lookup is for a bespoke service, HTTPS, or SMTP post-MX-lookup-fail, or …  
* Browsers have shifted to preferring looking up HTTPS and SVCB Resource Resources (RR)s  
  * [RFC 9460](https://datatracker.ietf.org/doc/html/rfc9460)  
  * [RFC 9461](https://datatracker.ietf.org/doc/html/rfc9461)  
* These newer RRs provide a lot of context for applications:  
  * IPv4 and IPv6 addresses  
  * TLS ALPN  
  * Encrypted ClientHello parameters  
  * extensible set of parameters

# Caveats

* rough sketch  
* not battle-tested  
* the various **NOTE:** s sprinkled throughout represent future work or discussion items

# Assumptions

* scoped "Internets", network of networks using Internet technologies, with non-DTN connectivity within its subgraph  
* scoped Internet has a view of a DNS hierarchy, i.e. DNS server architecture like Earth's Internet with access to root zone and hierarchical zones below that  
* zero, one, or multiple DTN connectivity between any pair of scoped Internets  
* extensions to existing RIR schemes work for allocation IPv6 blocks, no different from Earth today

# REQs

* any DNS zone should be able to serve a reasonable tree in the local SSI network  
* they should be able to have CDNs locally presently  
* and they should be able to indicate the BPv7 and/or IP addresses of local ALG servers  
* the Control Plane info that programs the ALG servers is out of scope

# CONOPS for arbitrary applications

* Unknown modern application looks up either an HTTPS or SVCB RR  
* any domain owner can have their zone information synchronized out to any of the scoped Internets  
  * **NOTE:** there is likely a Control Plane issue deciding what how much of what information is synced from the zone owner to each of the scoped Internets  
* If a DNS lookup doesn't find any RRs, application DNS error paths are executed  
  * DNSSEC NSEC/NSEC3 can be used to validate the absence of RRs or zones  
  * **NOTE:** possible extension to say "it's not available within this SSI"  
* If the scoped Internet DNS has an HTTPS/SVCB RR:  
  * **NOTE:** add a new extension that tells you a DTN Bundle EID of the destination in addition to the IP addresses and so on  
  * For an HTTPS RR  
    * could get back **both** the Bundle EID of the destination or the IP addresses and HTTPS parameters of a local proxy service  
* If the IP addresses are reachable: great  
  * if not, the scoped Internet can help operationally by giving ICMP errors to traffic that will otherwise leave the scoped Internet  
    * **NOTE:** perhaps a new Destination Unreachable subcode  
      * [https://datatracker.ietf.org/doc/html/rfc4443\#section-3.1](https://datatracker.ietf.org/doc/html/rfc4443#section-3.1)  
      * Destination Unreachable // "beyond scope of local Internet"  
* If you speak BPv7:  
  * Application execute Bundle Protocol operations, whatever that means for itself (how to pack its application payload into Bundles)  
    * e.g. [RFC 9292](https://datatracker.ietf.org/doc/html/rfc9292) binary HTTP messages  
  * Look up how to reach the DNS-advertised Bundle EID  
    * **NOTE:** this can also be stored in DNS, with various records for a DNS name transformation of the Bundle EID (e.g. something in ipn.arpa)  
  * CLA\_connect() to BP Agent and away we go…

# DNS extensions for CLA information BP EIDs

In general we should consider SVCB extensions for identifying everything you need to connect to a CLA

* no different from the HTTPS lookup  
* "BPv7 QUIC-CLA a/aaaa=..." where "QUIC\_CLA" is possibly a new ALPN we request  
* (insert even more unstructured thoughts here)…
