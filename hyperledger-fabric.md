# Inbox

- [http://miiingo.tistory.com/49?category=644184](http://miiingo.tistory.com/49?category=644184) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html#mapping-msps-to-organizations](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html#mapping-msps-to-organizations) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/developapps/scenario.html](https://hyperledger-fabric.readthedocs.io/en/latest/developapps/scenario.html) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html](https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/ops_guide.html](https://hyperledger-fabric.readthedocs.io/en/latest/ops_guide.html) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/architecture.html](https://hyperledger-fabric.readthedocs.io/en/latest/architecture.html) 
- [https://hyperledger-fabric.readthedocs.io/en/latest/Fabric-FAQ.html](https://hyperledger-fabric.readthedocs.io/en/latest/Fabric-FAQ.html) 
- Instantiate 이후 Container 및 이미지로 배포됨?
- 만약 유일하게 Chaincode가 install된 Peer가 죽으면 그 채널의 Chaincode는 어떻게 되는가?
- 네트워크에 Consortium이 전혀 존재하지 않아도 채널을 만드는데 문제가 없어 보인다. 
- Chaincode를 모든 Peer에 Install하지 않아도 Channel에 Instantiate되어 있으면 Channel에 query가 들어왔을 때 다른 Peer에 자동으로 Install되는가?
- Peer에 Chaincode를 설치했지 Ledger를 초기화한 적은 없다. 자동으로 생성되는 것인가? 그리고 Chaincode 별로 따로 생성되는 것인지
- 로컬에서 Kafka 기반의 Orderer 띄워보기
- 로컬에서 CA 서버도 띄워서 연동해보기

**Blockchain**

- 작동 원리

**Bitcoin**

- UTXO (Unspent Transaction Output) Model
- PoW

**Ethereum**

- Account/Balance Model
- PoS (Casper)
- ERC20

**Hyperledger Fabric**

- 인증과 전자서명에 대한 지식
- 컨테이너 기반 가상화
- 보안통신(TLS SSL 등)
- Chaincode(Smart Contract) 작성을 위한 기본적인 Token 설계 로직에 대한 이해 → Ethereum Token 소스 참고
- https://etherscan.io/tokens : Ethereum Top20 Token들이며, 이더 네트워크에 올라간 Smart Contract 확인 가능
- 공식 문서 참조
- 직접 직접 체인코드 개발 및 네트워크 구성하여 배포해보기 : balance-transfer가 참고하기 좋음
- Optimizing : AWS에서 TPS 약 2000 ~ 3000 이상
    - Orderer : 4 Cores, Ram 32g
    - Peer : 16 Cores, Ram 32g

**Go**

- Hyperledger Fabric SDK를 Go 버전으로 사용중

**Git**

- 여기서 어떤 모델로 운영중인지 확인

**Ansible**

- 개념 및 사용법 실습

**K8s**

- 개념 및 사용법 실습

**AWS**

- EC2 종류와 사용법

**Helm Chart?**

- 

**NodeJS?**

- 테스트용 API 서버가 NodeJS로 구성되어 있음

**React Native?**

- Client가 React Native로 구성되어 있으나 바뀔 가능성이 크다고 함


# Archive


### Channel

Network configuration과 별개로 Channel configuration에 의해 정의된다. 

Network/Channel configuration 모두 동일한 configuration transaction으로 설정된다. admin이 configuration transaction을 submit하여 endorse 받으려면 해당 org의 cert로 sign되어야 하고 mod_policy 권한이 있어야 한다. 


### Orderer

The ordering service is the component which gathers endorsed transactions from applications and orders them into transaction blocks, which are subsequently distributed to every peer node in the channel. At each of these committing peers, transactions are recorded, whether valid or invalid, and their local copy of the ledger updated appropriately.

보통 여러개의 노드를 클러스터링하여 사용한다. 

한 Org 안에 여러개의 Orderer를 동작시킬 수 있다. 

Orderer node는 자신이 속한 Network configuration의 사본을 가진다. 

Orderer는 별도로 System Channel을 사용하며 하나의 작은 블록체인처럼 동작한다. 


### Peer

물리적으로 Ledger 사본을 가진다. 논리적으로는 해당 Peer가 연결된 Channel에서 Ledger 사본을 가지는 것으로 볼 수 있다. (Channel을 통해 Peer와 통신하므로)

Peer가 최초 Orderer에 Channel join 요청을 보내면, Channel configuration을 사용하여 해당 Channel에서 Peer의 사용 권한을 결정한다. Channel configuration은 Peer가 Ledger에 정보를 읽거나 쓸 수 있는지 여부를 결정한다.

Peer node는 자신이 속한 Channel configuration의 사본을 가진다. 

Peer의 종류

1. **Committing peer** : Every peer node in a channel is a committing peer. It receives blocks of generated transactions, which are subsequently validated before they are committed to the peer node’s copy of the ledger as an append operation.
2. **Endorsing peer** : Every peer with a smart contract can be an endorsing peer if it has a smart contract installed. However, to actually be an endorsing peer, the smart contract on the peer must be used by a client application to generate a digitally signed transaction response. The term endorsing peer is an explicit reference to this fact. An endorsement policy for a smart contract identifies the organizations whose peer should digitally sign a generated transaction before it can be accepted onto a committing peer’s copy of the ledger.
3. **Leader peer** : When an organization has multiple peers in a channel, a leader peer is a node which takes responsibility for distributing transactions from the orderer to the other committing peers in the organization. A peer can choose to participate in static or dynamic leadership selection. \
It is helpful, therefore to think of two sets of peers from leadership perspective – those that have static leader selection, and those with dynamic leader selection. For the static set, zero or more peers can be configured as leaders. For the dynamic set, one peer will be elected leader by the set. Moreover, in the dynamic set, if a leader peer fails, then the remaining peers will re-elect a leader. \
It means that an organization’s peers can have one or more leaders connected to the ordering service. This can help to improve resilience and scalability in large networks which process high volumes of transactions.
4. **Anchor peer** : If a peer needs to communicate with a peer in another organization, then it can use one of the anchor peers defined in the channel configuration for that organization. An organization can have zero or more anchor peers defined for it, and an anchor peer can help with many different cross-organization communication scenarios.

Note that a peer can be a committing peer, endorsing peer, leader peer and anchor peer all at the same time! Only the anchor peer is optional – for all practical purposes there will always be a leader peer and at least one endorsing peer and at least one committing peer.

## CA (Certificate Authority)

CA에서 Certificates를 배분한다. 

MSP가 Certificates와 Member Organization을 매핑한다. 

Network Configuration이 MSP를 사용한다. 

Network Configuration이 Policies 설정에 MSP’s name를 사용한다. 


### MSP vs CA

MSP is a Membership Service Provider - pluggable interface to support variety of credentials architectures, basically offering abstraction layer for membership orchestration architecture. MSP abstraction provides:



1. Concrete identity format
2. User credential validation
3. User credential revocation
4. Signature generation and verification

While Fabric-CA used to generate certificates and keys to actually initialize MSP facilities. Fabric-CA is a default implementation of MSP interface to cover identity management.

X.509 certificates are used in client application transaction proposals and smart contract transaction responses to digitally sign transactions. 

CA can be used to identify users from organizations. 


### Register

**Request parameters**

- adminCert = B64(admin.getEnrollment().getCert())
- signature = adminPrivateKey("${method}.${file}.${body}.${adminCert}")
    - method = POST
    - file = B64(url.getFile())
    - body = B64(RegistrationRequest.toJson())

**Handling request in CA**

- adminInfo, adminPublicKey = caPublicKey(adminCert)
- method, file, body, adminCert = adminPublicKey(signature)

**Response body**

- enrollSecret


### Enroll

**Before request**

- userPrivateKey : generated locally, not will be sent
- userPublicKey : generated locally, will be sent

**Request parameters**

- enrollId
- enrollSecret : optional
- userPublicKey : in CSR (Certificate Signing Request)

**Response body**

- userCert = caPrivateKey(userInfo, userPublicKey)


## Consortium

Consortium definition은 Network Configuration에 저장된다.

갯수와 상관없이 Organizational members를 가진다. 


## (Client) Application

Application → Ledger 접근시 [Application → Channel → Peer → Chaincode → Ledger]의 흐름을 따른다. 


## Chaincode

Chaincode 개발 완료 → Org의 Admin이 해당 Chaincode를 Peer에 Install → 해당 Peer를 사용하여 Channel에 대해 Instantiate → Application에서 Chaincode 호출 가능

네트워크 상의 모든 Peer가 Chaincode를 가져야 하는 것은 아니다. Channel에 Transaction proposal이 오면 Channel은 Chaincode 수행 가능한 Peer를 자동으로 찾아낸다. 

installing a smart contract shows how we think of it being physically hosted on a peer, whereas instantiating a smart contract shows how we consider it logically hosted by the channel.

Instantiate 시 Endorsement Policy를 설정한다. 이는 어떤 Org이 다른 Org에서 Ledger 사본으로 승인되기 전에 거래를 승인해야 하는지를 정의한다.

Flow of invoking a smart contract

1. Client application에서 Transaction proposal을 Peer에게 보낸다. (Transaction proposal은 Smart contract의 input으로 보면 된다)
2. Peer에서 Transaction response를 Client application으로 반환한다. 

Read/Write Set

Read Set은 transaction proposal이 반영되기 전 값들

Write Set은 transaction proposal이 반영된 후의 값들


## Consensus


## Ledger
