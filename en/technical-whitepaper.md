# CITA Technical Whitepaper

Jan Xie

Version 1.0

## Introduction

Known for proving the concept and introducing the world to blockchain technology, Bitcoin creatively solves the distributed consensus problem in a permissionless setting, and reveals a whole new architecture based on distributed computing and cryptography. Ethereum brings in a more general purpose computation layer, that demonstrates the extraordinary potential of smart contract technology. In the context of permissioned blockchains, we ask an interesting question: what if we consider this new architecture in a permissioned environment, where nodes maintain their identities?

Permissioned blockchains satisfy the performance requirements for enterprise level applications. Identify management and permission control are the key components to making the system functional. Only nodes that satisfy permission requirements are allowed to join the network and communicate with other nodes, while in a permissionless blockchain nodes join and leave freely. Permission control allows the prevention of sybil attacks, and allows traditional consensus algorithms to be applicable, while largely boosting throughput and reducing latency of transaction processing.

Node configuration and network status vary notably on open networks. In order to lower the entry barrier and avoid centralization, public blockchain protocols have to give consideration to the weakest node, thus limiting the design space. Nodes in permissioned networks can use better hardware and are more closely aligned with each other. An appropriate architecture should take advantage of this dynamic in order to achieve further performance improvements.

In a partitioned system, it’s known that we cannot have both availability and consistency at the same time. Public blockchains usually favor availability since it is much harder to recover from emergencies in a highly decentralized governance model. In enterprise blockchain applications the existence of efficient off-chain collaboration and governance means that faster response times are possible, so that users can favor consistency within the system.

As users of blockchain applications increase over time, the system has to scale to provide more and more transaction processing and storage capacity. Scalability while maintaining  security is a must-have in the context of  blockchain technology and although we do not see this in public chain infrastructures yet, permissioned blockchains can presently produce a viable solution to this conundrum.  

Data stored on the blockchain is public to all nodes, and while enterprise blockchain applications ask for privacy protection, the solutions based on pseudonyms or temporary transaction keys are not enough to satisfy requirements. Moreover, privacy solutions using bleeding-edge cryptography are not yet mature and fast enough for real world use. We believe what enterprises need is an imperfect but practical solution, and a modularized blockchain architecture with which future privacy solutions can easily plug-in.

Business logic is extremely complicated in enterprise applications; a single design can hardly meet all the requirements of a certain deployment. To maximize efficiency gains instantiated by blockchain technology, blockchain node software must be customizable to adapt different deploy and integration environments.

We perceive that independent blockchain networks are constantly emerging nowadays while even more will appear in the future. Various chains will begin to communicate with each other forming a network of blockchain networks. All blockchains should foreseeably prepare for cross-chain communication, to amplify the value of applications running on the various chains.

Given our analysis of this environment in addition to thorough understanding of the technology, we created CITA, an enterprise oriented blockchain framework that supports  smart contract execution and design. CITA provides a stable, efficient, flexible and future-proof platform for enterprise-level blockchain applications.

## Microservices

When designing enterprise-level applications, scalability is a key concern , which is one of the marquee problems of blockchain technology today. No matter how many nodes there are in a blockchain network, the capacity is limited to a single node. To increase system capacity we have two options:

1. comprise global transaction validation on the premise of security, i.e. sharding or cross-chain;
2. enhance the capability of every single node, i.e. use powerful and expensive servers (scale up).

CITA adopts a microservices architecture to boost each (logical) node’s performance. As shown in Fig. 1, a logical node is composed by a group of loosely coupled microservices, and those services communicate with each other through a message bus. In CITA, a "node" is a logic concept which may be a single server (with a group of services running on it), or a cluster of servers.

With a microservice architecture, CITA can be easily scaled. Node administrators can increase system capacity simply by adding more PC servers on high load. The administrator can even use dedicated servers to provide services for hot-spot accounts. We call this as **Internal Sharding**.

There’s no special hardware requirement other than a common PC. Transactions can be routed to multiple servers, and each server only needs to process a fraction of the load.  Together they will meet any level of enterprise application activity in the system. In certain scenarios, different logic nodes run different groups of microservices to provide a variety of  services to the system if necessary, e.g. validator vs. storage nodes.

Customization and integration is easy with CITA’s microservice architecture. Microservices are loosely coupled and their communications are only via messages. Hence microservices/components in CITA can be replaced by users with any programming language, as long as the microservice implements standard internal API’s to parse, process and return messages. External systems can connect directly to the message bus too, in order to read internal messages at runtime, for easy and deep integration.

![Fig 1. CITA Microservice Architecture](architecture.png)

## Consensus

Blockchain nodes reach a consistent transaction history via consensus algorithms. The consensus service selects valid transactions to build a global total/partial order log, which is the input for later execution. Transactions are stored into an immutable history built on authenticated data structures, resulting in what we would acknowledge as a ‘view’ after transactions are processed by the executor. A ledger maintaining account balances is a view, for example.

Different blockchains differ on whether a view requires consensus. For instance, UTXO sets are not pinned into Bitcoin blocks, while the hash of the ‘world state’ consists of all accounts will be included in an Ethereum/Fabric block. Consensus on the view can help identify bugs in transaction execution. Including hashes of view in blocks also facilitates view data exchange between nodes, which benefits light clients and cross-chain protocols. Therefore we have designed CITA’s block data structure to support consensus on view.

As a service shared by multiple participants, transactions sent by users must be included into the blockchain within a certain time frame, in order to facilitate censorship resistance. Being unable to execute a transaction can lead to great loss in many financial scenarios, e.g. trader failed to deposit more cash or sell some assets in time on a margin call. Due to the fact that validator nodes are able to pick and order transactions, they must be rotated regularly in order to make sure that any specific transactions won’t be blocked by a certain node for too long. CITA applies a proactive round-robin rotation strategy by default to ensure censorship resistance, and random rotation is available as an extension.

On new blocks generated by leader on rotation, CITA runs Tendermint consensus. Tendermint is a high performance consensus algorithm designed for blockchains. Based on the partial-asynchrony assumption, Tendermint can tolerate up to 1/3 Byzantine nodes. Validator nodes in CITA use gossip protocols to broadcast their votes on every block, achieving  consensus rapidly. Votes will be stored in the block for verification and auditing. With CITA's low latency technology sub-second transaction confirmation time is achievable, qualifying the system for most critical applications.

Tendermint can be replaced by more appropriate consensus algorithms if necessary. The alternate consensus can be written in any language as long as it implements the consensus microservice interface. Note that consensus algorithms are hard to abstract perfectly since they are usually associated with special network/storage requirements, and their replacement may be coupled with modifications to other services.

## Execution

### Asynchronous Transaction Execution (ATE)

A blockchain node’s functionalities include peer-to-peer networking, consensus, transaction execution and authenticated data storage. Nodes reach consensus on transaction order, and execute transactions one by one in the determined order. Under deterministic execution, all nodes reach a consistent state eventually, that is, they store the same data in their local database.

Consensus and transaction execution are highly coupled in today’s blockchain projects. As a result, the execution of transactions set a bottleneck to the performance of consensus. In CITA, consensus and transaction execution are decoupled as separate microservices. The  consensus service is only responsible for transaction ordering, which can finish independently before transaction execution, so the later can run asynchronously. ATE enables better consensus performance as well as elastic transaction processing: system loads can be distributed to many blocks in a time range (Fig. 2).

However, due to asynchronous executions, only limited validation (i.e. signature verification) can be applied to transactions in consensus; therefore consensus output may include invalid transactions which will be fed to the executor. This problem can be solved by quota control mechanisms and garbage cleaning tools provided by CITA.

![Fig 2. Asynchronous Transaction Execution](ate.png)

### Executor

Applications care more about views than transaction history. Transaction executors take ordered transactions as inputs, and update associated view during processing. Different executors will generate different views even with the same transaction history. CITA supports executors listed below by default:

1. NOOP: Simplest executor which does nothing.
2. Native: Feed the transaction as input data to contract written in native language.
3. EVM: A thin wrapper of the Ethereum Virtual Machine; support the creation and invocation of Ethereum smart contract.
4. Private: Private transaction execution.
5. Hybrid: a combinator for executors.

View state is read and written by an executor during transaction processing. View states may have their own unique data models, which are most commonly, UTXO and Account models. In the UTXO model, UTXOs constitute a view of the ledger where each transaction creates new UTXOs out of consumed UTXOs. In the Account model, accounts constitute a view of world state where a transaction may read/write multiple accounts in execution.

The UTXO model introduces a ledger invariant, in which the total number of accounting units must stay the same before and after transaction execution, by sacrificing generality. By splitting account balances into multiple UTXOs, it benefits parallelization to a certain extent, while compromising versatility for complex business and bringing in the complexity of splitting/combining UTXOs. Account model is simpler and more efficient for general tasks. Moreover, metadata such as those for authentication and authorization can be associated with accounts naturally in enterprise applications. CITA supports the Account model by default, while users can build their own view state models like UTXO.

### Quota

Transactions will be replicated, stored and executed on multiple nodes. The resources of nodes are limited and shared by all users, and nodes will be overloaded and lose  responsiveness if too many workloads are submitted into the system, therefore creating a situation in which blockchains with smart contract support must find a way to restrict resource use. In CITA we call the unit of resource a ‘quota’, and the issuance and consumption  mechanisms are called quota management. Quota issuance and consumption strategies are  configurable to users with quota management permissions.

How quota is consumed depends on the executor used in transaction processing. For example, the NOOP executor consumes quota by transaction size; the Native executor consumes quota as a realtime clock ticks; while the EVM has built-in fine grained GAS mechanisms which count opcode’s complexity. In CITA we set quota limits for blocks/views and the users in order to limit resource usage of individual blocks.

In contrast to a public blockchain, permissioned blockchains usually don’t issue tokens to provide on-chain consensus incentivisation, therefore this platform needs an alternative  solution to issue quota in order to compensate user’s quota consumption. Quote issuance is very flexible in CITA: a simple periodical-recovery strategy is supported by default, and customized strategies can also be configured if necessary.

### View

Blockchain is an Online Transaction Processing (OLTP) system: users broadcast their transactions; nodes execute the transactions upon receival. There are alternatives with different pros and cons to transaction processing in blockchain systems. Nevertheless, most blockchain systems pick one of them, and are thus inflexible and encounter difficulty in  meeting the needs of varied scenarios. By what we call transaction tunnels, CITA supports multiple-strategy transaction processing and basic parallelization. 

One can set multiple independent views when configuring CITA. Each view has its own executor and state data model, and registers its executor to the transaction router. After ordering by consensus service, transactions are dispatched via the router to corresponding executors. Transaction sets processed in different views may or may not have intersections.

By proper view configuration, CITA versatilely supports all kinds of application scenarios. A view with a NOOP executor is economical for notary businesses. A view with a Native executor and Account model fits best for applications with stable business logic. The EVM combined with the Account model is adequate for businesses with frequently changing requirements.

CITA runs independent transaction execution services for different views, due in part to the  state data independence. In a CITA network with multiple views, processing capacity is nearly proportional to the number of views.

![Fig 3. Transaction Router and Views](router-and-views.png)

### Privacy 

Replicated execution of smart contracts and privacy are contradictory in nature. Execution/verification requires that all validator nodes can read data within a transaction. However, to protect privacy, irrelevant validator nodes shall not be allowed to see the respective data.

Privacy protection based on pseudonyms conceal the sender and receiver of a transaction to some extent. Though with the help of data analysis, one can still acquire user information in a transaction. In a one-time transaction key based solution, transactions are still decrypted before execution, and data is only hidden to ordinary users not validator nodes. This is not quite practical since validators are also competitors in most enterprise applications.

Some advances in cryptography, like zero knowledge proofs or full homomorphic encryption, help us to proceed a transaction without revealing its data. However, technologies of such kind have their own bottlenecks against practicality and maturity.

CITA features partial-execution to protect privacy for its users. Before a private transaction is submitted, transaction data is encrypted. The encrypted transaction is  then sent to relevant nodes through a peer-to-peer private transport connection, while its hash value gets packed into the block. Private transactions are only stored and executed on relevant nodes, completely eliminating the risk of privacy leaks.

## Authentication and Authorization

There are generally two kinds of roles in a blockchain network: nodes and users. Nodes are service providers and users are consumers of the shared computing and storage service.

CITA provides a standard interface for node authentication, and imposes very strict restrictions on nodes joining. Connections from a node that fails to authenticate itself will be dropped even if it is in the same network as other nodes.

For widely adopted centralized user authentication services based on LDAP or PKI in enterprise applications, CITA provides standard interfaces to facilitate integration with such services too.

In a public-key cryptography based authentication solution, the result could be disastrous if one loses his/her private key. CITA has a sophisticated identity management solution. When users lose their keys, or when their keys need to be updated, administrators with key update permissions can replace the old key with a new one at a user’s request.

CITA also features role-based access control for enterprise-level applications. Resources operable by users are divided in fine-grained components to various levels of authority. Users can define roles to organise users access control and manage resource accessibility, allowing enterprises to configure CITA deployment to match their organizational structure. Updates to permissions and roles are all stored on a blockchain for future auditing.

## Governance

Blockchain is a tool used to reflect the **consensus of human**. In normal cases, blockchain facilitates coordination automation. In abnormal cases, erroneous view data may occur. Calibrations can be made based on immutable transaction history. As a distributed system of equivalent peers, blockchain has no central node in its networking topology, but its governance may be carried out in either decentralized or centralized processes. With regards to the principle that **transaction history must be immutable** while views are mutable, CITA supports various governance structures and view amendment capabilities.

There is a superadmin role in CITA. Benefiting from a flexible authentication service design, superadmins may have any authentication logic. In a centralized governance method, superadmins can be controlled by a core member. In a multi-center governance method, core members can constitute a committee to manage the superadmin together. 

A centralized governance role/committee is able to agree on a resolution via some off-chain channel, which allows the system to recover rapidly when emergencies happen. When problems like mistaken operations, software errors or hardware errors happen, systems will enter into an emergency status. We define two types of emergency statuses: Transaction Recoverable and Message Recoverable.

Systems with erroneous view data from mistaken transactions or bugged smart contracts are  in a transaction recoverable status, since nodes can still process transactions. A superadmin can create an amend transaction for a fast fix. Nodes will include such amend transactions into blocks too allowing that any view update operations are stored as evidence for auditing.

When a system is in a message recoverable status, nodes are unable to execute transactions and consensus services are at a standstill, though peer-to-peer networks are still functional. A superadmin can broadcast a special message with the administrator tool provided by CITA. Upon receival of such messages, nodes verify the sender's identity and execute them directly without consensus.

## Summary

For enterprise-level blockchain applications, we propose CITA, a blockchain framework with smart contract support. In CITA, functionalities of a blockchain node are decoupled into microservices, including consensus, transaction execution, peer-to-peer network, quota, authentication and authorization. Microservices coordinate via a message bus. CITA is designed to be a highly extensible and a future-proof general blockchain framework, where users can configure and customize services on demand.
