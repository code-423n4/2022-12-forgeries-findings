# Report

---

## Gas Optimizations

---

|       | Issue                                                      | Instances |
| ----- | ---------------------------------------------------------- | --------- |
| GAS-1 | USE NAMED RETURNS FOR LOCAL VARIABLES WHERE IT IS POSSIBLE | 1         |

---

Total: 1 instances over 1 issue with estimation that good amount of gas is saved on deployment.

#### [GAS-1] USE NAMED RETURNS FOR LOCAL VARIABLES WHERE IT IS POSSIBLE

We should be using named returns for local variables whenever it is possible. It saves gas on deployment and also in every call.

```
File: /CIP-001/src/Turnstile.sol
38:   function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
39:       external
40:        returns (address)
41:    {
42:       address admin = msg.sender;
43:     // Clone the contract
44:      address newDrawing = ClonesUpgradeable.clone(implementation);
45:        // Setup the new drawing
46:        IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
47:        // Emit event for indexing
48:        emit SetupNewDrawing(admin, newDrawing);
49:        // Return address for integration or testing
50:        return newDrawing;
51:    }

```

- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L38

#### Mitigation

USE NAMED RETURNS FOR LOCAL VARIABLES

```
File: /CIP-001/src/Turnstile.sol
38:   function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
39:       external
40:        returns (address newDrawing)
41:    {
42:       address admin = msg.sender;
43:     // Clone the contract
44:      address newDrawing = ClonesUpgradeable.clone(implementation);
45:        // Setup the new drawing
46:        IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
47:        // Emit event for indexing
48:        emit SetupNewDrawing(admin, newDrawing);
49:
51:    }


```
