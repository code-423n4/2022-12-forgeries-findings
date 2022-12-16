[G-01]EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

INSTANCES:

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L-59




[G-02] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

INSTANCES:

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L-75