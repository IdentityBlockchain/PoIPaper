# Identity Blockchain - Proof of Identity

## Introduction

_Identity Blockchain_ uses state-certified electronic identities (eIDs) to create blockchain identities and a consensus protocol called Proof of Identity (PoI). PoI is "green" and enables many applications that are impossible to implement without verified identity on the blockchain, e.g., direct democracy and universal basic income, encrypted messaging, etc. _Identity Blockchain_ preserves identity anonymity by using zk-SNARKs and an anonymization protocol for identity onboarding.

## Notation

| Symbol          | Description                                                        |
| :-------------- | :----------------------------------------------------------------- |
| $(K, K^{-1})$   | Pair of public key $K$ and the corresponding private key $K^{-1}$  |
| $K(value)$      | String $value$ encrypted with a public key $K$                     |
| $K^{-1}(value)$ | String $value$ encrypted with a private key $K^{-1}$               |
| $H$             | zk-SNARK-friendly hash function                                    |
| $Nin$           | Unique state-issued personal _National Identification Number_      |
| $CA$            | Certificate Authority                                              |
| $eID$           | Electronic identification document                                 |
| $K_{ID}$        | CA-certified public key of eID                                     |
| $K_{P}$         | Person key, unique public key used to access _Identity Blockchain_ |

Sometimes, instead of $(K, K^{-1})$, we simply write $K$.

## Registering $K_{ID}$

Register $K_{ID}$ on _Identity Blockchain_ by calling
$$registerK_{ID}(H(Nin), K_{ID}^{-1}(\text{"}registerK_{ID}\text{"}), proofNinOwnsKid).$$

### Pseudo-solidity code

```solidity
  mapping(uint256 => uint256) public ninKidUsed;
  mapping(uint256 => bool) public kidInvalid;

  function registerKid(
    uint256 hNin, // hNin = H(Nin)
    uint256 vKid, // vKid = KidPrivate("registerKid")
    bytes proofNinOwnsKid
  ) public {
    uint256 previousKey = ninKidUsed[hNin];
    if (previousKey == vKid) return; // already registered
    bool isOwner = zksnarkverify(VK, [hNin, vKid], proofNinOwnsKid); // VK = verification key
    if (isOwner && !kidInvalid[vKid]) {
      if (previousKey != 0) kidInvalid[previousKey] = true;
      ninKidUsed[hNin] = vKid;
    }
  }
```

### Features

1. **Sybil resistance**: A Person is prevented from registering more than one $K_P$, e.g. by using different identification documents issued for the same $Nin$.
2. **Fix for the loss of eID**: A Person who has lost a previously registered $K_{ID}$, e.g. due to loss of eID, can overwrite it by registering a new $K_{ID}$.
3. **Identity theft damage control**: If a Person overwrites the previous $K_{ID}$ with a new one, the identity Thief loses access to services that verify the entire identity chain.

### Remarks

An entity that knows $K_{ID}$, e.g. the CA who certified the corresponding eID, can know that the Person has registered $K_{ID}$ on _Identity Blockchain_.

## Registering $K_P$

Register $K_P$ on _Identity Blockchain_ by calling
$$registerK_P(NinK_{ID}, NinK_{ID}K_P, proof),$$
where:

$NinK_{ID}=H(Nin,K_{ID}^{-1}(\text{"}registerK_P\text{"}))$

$NinK_{ID}K_P=H(Nin, K_{ID}^{-1}(\text{"}registerK_P\text{"}), K_{ID}^{-1}(\text{"}registerK_P\text{"}))$.

### Pseudo-solidity code

```solidity
mapping(uint256 => uint256) public ninKidKpUsed;
  mapping(uint256 => bool) public kpInvalid;
  mapping(uint256 => uint256) public ninKidKpLastConfirmed;

  function registerKp(
    uint256 ninKid,
    uint256 ninKidKp,
    bytes proof
  ) public {
    uint256 previousKey = ninKidKpUsed[ninKid];
    if (previousKey == ninKid) return; // already registered
    bool ok = zksnarkverify(VK, [ninKid, ninKidKp], proof); // VK = verification key
    if (ok && !kpInvalid[ninKidKp]) {
      if (previousKey != 0) kpInvalid[previousKey] = true;
      ninKidKpUsed[ninKid] = ninKidKp;
      ninKidKpLastConfirmed[ninKid] = block.number;
    }
  }
```

### Features

1. **Sybil resistance**: $K_P$ is a unique public key bound to the Person on the _Identity Blockchain_. By design, the Person cannot have two valid $K_P$ keys at the same time.
2. **Anonimity**: No one who has a database with all public keys $K_{ID}$ or/and public keys $K_P$ can tell, from the function call, which $Nin$, $K_{ID}$ or $K_P$ was used. This is true under the assumption that CA, which certified the eID, does not have access to $K_{ID}^{-1}$, i.e. that $K_{ID}$ was securely generated on the eID.

## Problems

1. The rate of addition of identities, the problems of frequent changes of Merkle Tree Roots, and how long it takes to generate zkp proof, and block generation time.
2. The Thief stole $K_{ID}^{-1}$ before we registered on _Identity Blockchain_ and then registered $K_{ID}$ and $K_P$.
3. The Thief stole $K_{ID}^{-1}$ after we registered on _Identity Blockchain_.
4. Trying to use unregistered Key.
5. Trying to register the same $Nin$ with multiple Keys (e.g. two physical ids).
6. Using $K_P$ without checking the whole $CA\rightarrow K_{ID} \rightarrow K_P$ chain.

   - Using $K_P$ to register as a mining key.
   - Invalidating $K_P$.
   - Using new $K_P$ as a mining key.

   Solution:
   Look at the solution to Problem 7. The Mining Registry can accept new $K_P$ after an expiration period (which can be e.g. one day).

7. A combination of Problem 2 and Problem 6. The Thief steals $K_{ID}$, registers $K_P$ and registers $K_P$ for mining. The problem is bigger, because we can't invalidate $K_P$ if we don't know which $K_P$ it is.

   Solution: We make $K_P$ renewable. We write the last block number it was renewed on to the _Identity Blockchain_.

   Mining Registry can choose to accept $K_P$ that is not older than some time period (for the Registry a good period would seem to be one day).
   When paying the miners, the Registry also requires proof of $K_P$.

8. Only $K_P$ is compromised.
