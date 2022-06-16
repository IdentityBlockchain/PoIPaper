# Identity Blockchain

*IdentityBlockchain* uses state-certified electronic identities (eIDs) to establish blockchain identities and a consensus protocol called Proof of Identity (PoI). 

PoI is ”green” and enables many applications that are impossible to implement without verified identity on the blockchain, e.g., direct democracy and universal basic income, encrypted messaging, etc.

*IdentityBlockchain*, as presented here, aims to preserve identity anonymity by using zk-SNARKs and an anonymizing protocol for identity onboarding.

*IdentityBlockchain* will be a fork of Avalanche blockchain based on PoI.

The material that follows is under construction, bit it can offer a glimpse into the overlying ideas and concepts.

## Description

### Notation

| Symbol      | Meaning     |
| :---        |    :----  |
| $K$      | State CA certified public key of e.g. eID       |
| $K(value)$   | value encrypted with the public key $K$        |
| $K^{-1}$ | Private key corresponding to the public key $K$    |
| $P$ | Person key, used to access IdentityBlockchain as an unknown Person    |
| $H$ | SHA256    |

### Registering P


1. Use $K$ to sign *IdentityBlockchain*: $$K_1 = K^{-1}(registerK).$$ Entities who know $K$ (e.g. CA that issued it) can know that the Person is using *IdentityBlockchain*, and what identification document the Person is using.
  
2. Register $K$ on *IdentityBlockchain*, by calling $$registerK(Pin, K_1, proofPinOwnsK).$$

3. Register $P$ on *IdentityBlockchain*, by calling $$registerP(PinK, PinKP, proof),$$ where:
    - $K_2=K^{-1}(registerP)$
    - $PinK=H(Pin,K_2)$
    - $P_1 = P^{-1}(registerP)$
    - $PinKP=H(Pin, K_2, P_1)$
