# Lecture 01 Introduction

# Intro

*   What is a distributed system?

    *   multiple networked cooperating computers
    *   Examples:
    *   Email
    *   Gmail
    *   NFS
    *   HTTP
    *   DNS
    *   ARP
    *   Databases?
    *   MapReduce
*   Why distribute?

    *   Performance
    *   Parallel CPUs/mem/disk/net
    *   Capacity
    *   Fault-tolerance
    *   Connect physically separate entities
    *   Security via physical isolation?
*   But:

    *   complex, hard to debug
    *   new classes of problems, e.g. partial failure (did server accept my e-mail?)
    *   advice: don't distribute if a central system will work
*   Why take this course?

    *   interesting -- hard problems, non-obvious solutions
    *   active research area -- lots of progress + big unsolved problems
    *   used by real systems -- driven by the rise of big Web sites
    *   hands-on -- you'll build a real system in the labs

# Assessment Exercise

# Orgranization

Do (organization stuff)[01-org].

# Main Topics

*   Calendar will flow through these topics (we'll visit all of these ideas several times in many orders, but this is the intended order of the focus of the papers):
    *   Messaging, remote interaction (RPC)
    *   Fault-tolerance, replication, and consensus (Raft)
    *   Primary-backup replication (GFS)
    *   Fault-tolerant large-scale compute (MapReduce, Spark)
    *   Consistency/consistency models (Bayou, Dynamo)
    *   Real-world consistency and scaling (Scaling Memcached at Facebook)
    *   Transactions (Thor, Spanner, Argus)
    *   Byzantine fault-tolerance, P2P (PBFT, Bitcoin)
    *   Other possibly topics: verifying distributed systems (Verdi)

# Discussion

*   Example:

    *   a shared file system, so users can cooperate, like NFS
    *   lots of client computers
    *   [diagram: clients, network, vague set of servers]
*   _So many possibilities; let go of how you expect it to work, and just consider all of the possibilities._

*   Topic: architecture

    *   What interface?
        *   Clients talk to servers -- what do they say?
        *   File system (files, file names, directories, etc.)?
        *   Disk blocks, with FS in client?
        *   Separate naming + file servers?
        *   Separate FS + block servers?
    *   Single machine room or unified wide area system?
        *   Wide-area more difficult.
    *   Transparent?
        *   i.e. should it act exactly like a local disk file system?
        *   or is it OK if apps/users have to cope with distribution, e.g. know what server files are on, or deal with failures.
    *   Client/server or peer-to-peer?
    *   All these interact w/ performance, usefulness, fault behavior.
*   Topic: implementation

    *   How to simplify network communication?
        *   Can be messy (msg formatting, re-transmission, host names, etc.)
        *   Frameworks can help: RPC, MapReduce, etc.
    *   How to cope with inherent concurrency?
        *   Threads, locks, etc.
*   Topic: performance

    *   Distribution can hurt: network b/w and latency bottlenecks
        *   Lots of tricks, e.g. caching, concurrency, pre-fetch
    *   Distribution can help: parallelism, pick server near client
    *   Idea: scalable design
        *   Nx servers -> Nx total performance
    *   Need a way to divide the load by N
        *   Divide data over many servers ("sharding" or "partitioning")
        *   By hash of file name?
        *   By user?
        *   Move files around dynamically to even out load?
        *   "Stripe" each file's blocks over the servers?
    *   Performance scaling is rarely perfect
        *   Some operations are global and hit all servers (e.g. search)
            *   Nx servers -> 1x performance
        *   Load imbalance
            *   Everyone wants to get at a single popular file
            *   one server 100%, added servers mostly idle
            *   Nx servers -> 1x performance
*   Topic: fault tolerance

    *   Big system (1000s of server, complex net) -> always something broken
    *   We might want:
        *   Availability -- I can keep using my files despite failures
        *   Durability -- my files will come back to life someday
    *   Availability idea: replicate
        *   Servers form pairs, each file on both servers in the pair
        *   Client sends every operation to both
        *   If one server down, client can proceed using the other
    *   Opportunity: operate from both "replicas" independently if partitioned?
    *   Opportunity: can 2 servers yield 2x availability AND 2x performance?
*   Topic: consistency

    *   Assume a contract w/ apps/users about meaning of operations
    *   e.g. "read yields most recently written value"
    *   Consistency is about fulfiling the contract despite failure, replication/caching, concurrency, etc.
    *   Problem: keep replicas identical
    *   If one is down, it will miss operations
        *   Must be brought up to date after reboot
    *   If net is broken, _both_ replicas maybe live, and see different ops
        *   Delete file, still visible via other replica
        *   "split brain" -- usually bad
    *   Problem: clients may see updates in different orders
        *   Due to caching or replication
        *   I make a class directory private, then TA creates grades file
        *   What if the operations run in different order on different replicas?
    *   Consistency often hurts performance (communication, blocking)
        *   Many systems cut corners -- "relaxed consistency"
        *   Shifts burden to applications
