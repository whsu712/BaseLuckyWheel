# BaseLuckyWheel
BaseLuckyWheel
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseLuckyWheel is Ownable {

    uint256 public constant DAILY_SPINS = 8;
    uint256 public constant POINTS_PER_SPIN = 50;
    uint256 public constant STARTING_POINTS = 800;

    uint256 public totalSpins;

    struct Player {
        uint256 points;
        uint256 spinsToday;
        uint256 lastSpinDay;
        uint256 totalSpins;
        uint256 rareRewards;
    }

    mapping(address => Player) public players;

    enum Reward {
        Small,
        Medium,
        Big,
        Jackpot,
        RareCard
    }

    event SpinResult(address indexed player, uint256 spinId, Reward rewardType, uint256 pointsWon);

    constructor() Ownable(msg.sender) {}

    function _getToday() internal view returns (uint256) {
        return block.timestamp / 1 days;
    }

    function _initPlayer() internal {
        if (players[msg.sender].points == 0) {
            players[msg.sender].points = STARTING_POINTS;
        }
    }

    function spin() external {
        _initPlayer();
        uint256 today = _getToday();

        if (players[msg.sender].lastSpinDay != today) {
            players[msg.sender].spinsToday = 0;
            players[msg.sender].lastSpinDay = today;
        }

        require(players[msg.sender].spinsToday < DAILY_SPINS, "Daily spin limit reached");
        require(players[msg.sender].points >= POINTS_PER_SPIN, "Not enough points");

        players[msg.sender].points -= POINTS_PER_SPIN;
        players[msg.sender].spinsToday++;
        players[msg.sender].totalSpins++;
        totalSpins++;

        Reward rewardType = _getRandomReward();
        uint256 pointsWon = 0;

        if (rewardType == Reward.Small) {
            pointsWon = 30 + (totalSpins % 31);
        } else if (rewardType == Reward.Medium) {
            pointsWon = 90 + (totalSpins % 61);
        } else if (rewardType == Reward.Big) {
            pointsWon = 220 + (totalSpins % 181);
        } else if (rewardType == Reward.Jackpot) {
            pointsWon = 880;
            players[msg.sender].rareRewards++;
        } else {
            pointsWon = 150;
            players[msg.sender].rareRewards++;
        }

        players[msg.sender].points += pointsWon;

        emit SpinResult(msg.sender, totalSpins, rewardType, pointsWon);
    }

    function _getRandomReward() internal view returns (Reward) {
        uint256 rand = uint256(keccak256(
            abi.encodePacked(
                block.prevrandao,
                block.timestamp,
                msg.sender,
                totalSpins
            )
        )) % 100;

        if (rand < 45) return Reward.Small;
        if (rand < 75) return Reward.Medium;
        if (rand < 92) return Reward.Big;
        if (rand < 97) return Reward.Jackpot;
        return Reward.RareCard;
    }

    function claimDailyBonus() external {
        _initPlayer();
        uint256 today = _getToday();
        require(players[msg.sender].lastSpinDay != today, "Already claimed today");
        
        players[msg.sender].points += 120;
        players[msg.sender].lastSpinDay = today;
    }

    function getMyInfo() external view returns (
        uint256 points,
        uint256 spinsLeftToday,
        uint256 totalSpinsDone,
        uint256 rareRewards
    ) {
        Player memory p = players[msg.sender];
        uint256 today = _getToday();
        uint256 left = (p.lastSpinDay == today) ? 
                       (DAILY_SPINS > p.spinsToday ? DAILY_SPINS - p.spinsToday : 0) : DAILY_SPINS;

        return (p.points, left, p.totalSpins, p.rareRewards);
    }

    function givePoints(address player, uint256 amount) external onlyOwner {
        players[player].points += amount;
    }
}
transaction hash	0x675b83f34ce09e92872973bf237a996638b624735851bda7d3349a37410a2572
block hash	0xc83cfd00977080f79a79d70353e17034dda971a9ef14899318744144533b67f8
block number	47796325
contract address	0x63Dbd5572BaE9B821211F7829337d0434Cd2A98d
from	0x28FDAc6DdB21C7F4Bf5227E32E002D5A937De5db
to	BaseLuckyWheel.(constructor)
transaction cost	1169196 gas 
