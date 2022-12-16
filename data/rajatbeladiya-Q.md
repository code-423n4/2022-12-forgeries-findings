### 1). Different solidity version is used

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L2

all the other solidity files used solidity version 0.8.16. `Version.sol` uses ^0.8.0 and it uses an unlocked pragma version. Suggesting change it to 0.8.16

 

