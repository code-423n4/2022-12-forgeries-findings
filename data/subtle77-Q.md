ADD GAP TO RESERVE SPACE ON UPGRADEBLE ABSTRACT CONTRACT TO ADD NEW VARIABLES

On OwnableUpgradeable.sol add this lines;

/**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[50] private __gap;

In case you need to add variables for an upgrade you will have reserved space.
