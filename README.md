# Unitychain-core-contracts
// SPDX-License-Identifier: MIT pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol"; import "@openzeppelin/contracts/access/Ownable.sol";

contract UCNToken is ERC20, Ownable { constructor() ERC20("UnityChain Network", "UCN") { _mint(msg.sender, 1_000_000_000 * 10 ** decimals()); }

function mint(address to, uint256 amount) external onlyOwner {
    _mint(to, amount);
}

}

contract HumanityFund is Ownable { UCNToken public ucnToken; uint256 public cooldown = 1 days; uint256 public maxRequest = 1000 * 10 ** 18;

struct Claim {
    uint256 lastClaim;
    uint256 totalClaimed;
}

mapping(address => Claim) public claims;
mapping(address => bool) public flagged;

event Requested(address indexed user, uint256 amount);
event Flagged(address indexed user);

constructor(address _ucnToken) {
    ucnToken = UCNToken(_ucnToken);
}

function requestFund(uint256 amount) external {
    require(!flagged[msg.sender], "User flagged for abuse");
    require(amount <= maxRequest, "Request too large");
    Claim storage user = claims[msg.sender];
    require(block.timestamp - user.lastClaim >= cooldown, "Cooldown not met");

    user.lastClaim = block.timestamp;
    user.totalClaimed += amount;

    ucnToken.transfer(msg.sender, amount);
    emit Requested(msg.sender, amount);
}

function flagUser(address user) external onlyOwner {
    flagged[user] = true;
    emit Flagged(user);
}

function unflagUser(address user) external onlyOwner {
    flagged[user] = false;
}

function fundContract(uint256 amount) external {
    ucnToken.transferFrom(msg.sender, address(this), amount);
}

function updateCooldown(uint256 time) external onlyOwner {
    cooldown = time;
}

function updateMaxRequest(uint256 amount) external onlyOwner {
    maxRequest = amount;
}

}

