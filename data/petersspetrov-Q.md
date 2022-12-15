1.All contracts should have a latest solidity pragma ,is currently 0.8.17
	Also use latest Solidity version to get all compiler features, bugfixes and optimizations.
 

2.Functions should be written under the constructor for better code readability

	https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L9


3.ADD PARAMETER TO EVENT-EMIT
	Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app ,
	they can has that something has happened on the blockchain.
 
	https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L18
    https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L33
   

4.Empty function in VRFNFTRandomDrawFactory.sol
 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L55-L59
   

5.The Emit to be after the action  

  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L120
   function acceptOwnership() public onlyPendingOwner {
        emit OwnerUpdated(_owner, msg.sender);

        _owner = _pendingOwner;

        delete _pendingOwner;
    }
	
	must be:
	 function acceptOwnership() public onlyPendingOwner {
        _owner = _pendingOwner;
		
		emit OwnerUpdated(_owner, msg.sender);
		
        delete _pendingOwner;		
		
    }
	
	
6.Don't use magic numbers use  constants
  0 is used in many places, it is better to be a constant
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L101
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L106
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L144
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L152
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L255
  
  
7.Better code readability
  https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L215-L219
  
		 if (
            IERC721EnumerableUpgradeable(settings.token).ownerOf(
                settings.tokenId
            ) != address(this)
        )
       must be:
     if (
            IERC721EnumerableUpgradeable(settings.token)
			.ownerOf(settings.tokenId) != address(this)
        ) {
		
		
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L269-L273	
	      return
            user ==
            IERC721EnumerableUpgradeable(settings.drawingToken).ownerOf(
                request.currentChosenTokenId
            );
			mus be :
			return
            user == IERC721EnumerableUpgradeable(settings.drawingToken)
			.ownerOf(request.currentChosenTokenId);
		
	
8.Index event fields make the field more quickly accessible to off-chain tools that parse events.
  However, note that each index field costs extra gas during emission, so it’s not necessarily best to 
  index the maximum allowed per event (three fields). Each event should use three indexed fields if there 
  are three or more fields, and gas usage is not particularly of concern for the events in question. 
  If there are fewer than three fields, all of the fields should be indexed.
 
 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L11
 
 
 9.Change state variables of immutable  to constant.
 
 Both immutable and constant are keywords that can be used on state variables to restrict modifications to their state.
 The difference is that constant variables can never be changed after compilation, while immutable variables can be set
 within the constructor.
 
 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L22-L33