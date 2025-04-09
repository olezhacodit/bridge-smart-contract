# bridge-smart-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function transfer(address recipient, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract TokenBridge {
    address public admin;
    mapping(bytes32 => bool) public processedTransactions;
    
    event TokenLocked(address indexed user, uint256 amount, address token, string targetChain, bytes32 transactionId);
    event TokenUnlocked(address indexed user, uint256 amount, address token, bytes32 transactionId);

    constructor() {
        admin = msg.sender;
    }
    
    modifier onlyAdmin() {
        require(msg.sender == admin, "Not authorized");
        _;
    }
    
    function lockTokens(address token, uint256 amount, string memory targetChain) external {
        require(amount > 0, "Amount must be greater than 0");
        require(IERC20(token).transferFrom(msg.sender, address(this), amount), "Transfer failed");
        
        bytes32 transactionId = keccak256(abi.encodePacked(msg.sender, amount, token, targetChain, block.timestamp));
        emit TokenLocked(msg.sender, amount, token, targetChain, transactionId);
    }
    
    function unlockTokens(address user, address token, uint256 amount, bytes32 transactionId) external onlyAdmin {
        require(!processedTransactions[transactionId], "Transaction already processed");
        require(IERC20(token).balanceOf(address(this)) >= amount, "Not enough liquidity");
        
        processedTransactions[transactionId] = true;
        require(IERC20(token).transfer(user, amount), "Transfer failed");
        emit TokenUnlocked(user, amount, token, transactionId);
    }
}
345
