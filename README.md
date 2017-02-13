<h3 align="center">
  <a href="https://www.cossacklabs.com"><img src="https://github.com/cossacklabs/acra/wiki/Images/acra_web.jpg" alt="Acra: transparent database encryption server" width="500"></a>
  <br>
  Transparent database encryption layer with strong security guarantees.
  <br>
</h3>


[![CircleCI](https://circleci.com/gh/cossacklabs/acra/tree/master.svg?style=shield)][circleci]
[![Go Report Card](https://goreportcard.com/badge/github.com/cossacklabs/acra)](https://goreportcard.com/report/github.com/cossacklabs/acra)

## What is Acra

Acra helps you to easily secure your databases in distributed, microservice-rich environments. It gives you means to encrypt the data on application side into a special cryptographic container, store it in the database and then decrypt in secure compartmented area (separate virtual machine/container). Cryptographic design ensures that no secret (password, key, anything) leaked from the application or database is sufficient to decrypt protected data chunks which originate from it. 

Acra was built with specific user experiences in mind: 
- **quick and easy integration** of security instrumentation.
- cryptographically protect data in threat model, where **all other parts of infrastructure could be compromised**, and if AcraServer isn't, data is safe. 
- **proper abstraction** of all cryptographic processes: you don't risk mischoosing key length or algorithm padding. 
- **strong default settings** to get you going. 
- **high degree of configurability** to create perfect balance between extra security features and performance. 
- **automation-friendly**: most of Acra's features were built to be easily configured / automated from configuration automation environment.

Acra currently supports PostgreSQL as database backend, MongoDB and MariaDB (and other MySQL flavours) coming quite soon. Acra components should build on most modern Linux installations, but was built and test in debian-type Linuxes.

Acra has writer libraries for Ruby, Python, Go and PHP, but you can easily [generate AcraStruct containers](https://github.com/cossacklabs/acra/wiki/AcraStruct)  with [Themis](https://github.com/cossacklabs/themis) for any other platform you desire. 

Acra is available under Apache 2 license.

## Typical architecture

![](https://github.com/cossacklabs/acra/wiki/Images/generalarch.png)

* Your app talks to **AcraProxy**, local daemon, which emulates your normal PostgreSQL database, forwards all requests to **AcraServer** and reads back plaintext output. It is connected to **AcraServer** via [Secure Session](https://github.com/cossacklabs/themis/wiki/Secure-Session-cryptosystem), ensuring that all plaintext goes over trusted channel. It is highly desirable to run AcraProxy via separate user to compartment it from client-facing code. 
* **AcraServer** is a core entity, providing decryption services for all encrypted envelopes coming from database and then re-packing database answers for the application.
* To write protected data to database, you can use **AcraWriter library**, which generates AcraStructs and helps you  integrate it as a type into your ORM or database management code. You will need Acra's public key to do that. AcraStructs generated by AcraWriter are not readable by it - only server has sufficient keys to decrypt it. 
* You can connect to both **AcraProxy** and directly to database when you don't need encrypted reads/writes. However, increased performance might cost design elegance (which is sometimes perfectly fine, when it's conscious choice).

Typical flow looks like this: 
- App writes some records using AcraWriter, generating AcraStruct with AcraServer public key, updates database. 
- App sends SQL request through AcraProxy, which forwards it to AcraServer, AcraServer forwards it to database. 
- Upon receiving an answer, AcraServer tries to detect encrypted envelopes (AcraStructs), and, if succeeded, replacing them with plaintext answer, which then gets returned to AcraProxy over secure channel. 
- AcraProxy then provides answer to application, as if no complex security instrumentation is ever present within the system.

## 4 steps to start

* Read the wiki page on [building and installing](https://github.com/cossacklabs/acra/wiki/Installing,-building-and-running)  all components. Soon enough, they'll be available as pre-built binaries, but for now you'll need to fire a few commands to get the binaries going. 
* Deploy [AcraServer](https://github.com/cossacklabs/acra/wiki/How-AcraServer-works) binaries in separate virtual machine (docker container soon!). Generate keys, put AcraServer public key into both clients (AcraProxy and AcraWriter, see next).
* Deploy [AcraProxy](https://github.com/cossacklabs/acra/wiki/Client-side:-AcraProxy-and-AcraWriter#acraproxy) on each server you need to read sensitive data. Generate proxy keys, provide public one to AcraServer. Point your database access code to AcraProxy, access it as if it's your normal database installation!
* Integrate [AcraWriter](https://github.com/cossacklabs/acra/wiki/Client-side:-AcraProxy-and-AcraWriter#acrawriter) into your code where you need to store sensitive data, supply AcraWriter with proper server key.

## Zones

Acra provides means to cryptographically separate pieces of data belonging to different entities (users, clients, etc), by using separate Acra key for each of them. 
This happens at cost of: 
- performance of AcraServer during decryption
- necessity to perform minimal key management on app site to match key to zone during writes
- stipulating database design decisions: AcraServer, when reading through server response, will need to match detected AcraStructs to zone keys. To achieve that, each AcraStruct needs to be preceeded with ZoneID in query's output.

## Additionally

We fill [wiki](https://github.com/cossacklabs/acra/wiki) with useful reads on core Acra concepts, use cases, details on cryptographic and security design. You might want to:
- Read notes on [security design](https://github.com/cossacklabs/acra/wiki/Security-design) to understand better what you get by using Acra and what is the threat model Acra operates in. 
- Read [some notes on making Acra stronger / more performant](https://github.com/cossacklabs/acra/wiki/Tuning-Acra), adding security features or increasing throughput, depending on your goals and security model.

## Project status

Acra is early alpha. We've built it in cooperation with one of our early partners for their specific security goals, liked the design and then tried to generalize experience received for all other kinds of users. We're giving it out to the security/engineering community in hope that these use-cases are not overly unique and will benefit many other infrastructures. Please let us know in [Issues](https://www.github.com/cossacklabs/acra/issues) whenever you stumble upon a bug, see a possible enhancement or would just generally like to help.
