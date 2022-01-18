<img src="Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/main.png" width="100%"></img>

# Solidity - Fund Me

In this contract we are going to add a functionality to receive and withdraw payment

Get free tokens at

[LINK Token Contracts | Chainlink Documentation](https://docs.chain.link/docs/link-token-contracts/)

## Payable Function

If we declare our function as payable it means that we can accept payment on that function

```solidity
// SPDX_License_Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// Here we will make a contract that will accept payment
contract FundMe{
    function fund() public payable {

    }
}
```

After deployment

![Screenshot 2022-01-17 at 11.08.47 PM.png](Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/Screenshot_2022-01-17_at_11.08.47_PM.png)

This Red button means it is used for transaction

### Keeping track of the payments done by a particular address →

```solidity
// SPDX_License_Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// Here we will make a contract that will accept payment
contract FundMe{

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
    }
}
```

![Screenshot 2022-01-17 at 11.17.01 PM.png](Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/Screenshot_2022-01-17_at_11.17.01_PM.png)

![Screenshot 2022-01-17 at 11.20.52 PM.png](Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/Screenshot_2022-01-17_at_11.20.52_PM.png)

### In order to make a minimum limit for a transaction we need the latest price for Etherium so we will get data from (we can’t use any single owned api to get price because of security issue or ruining the concept of decentralisation)

### We can check the prices at →

[Decentralized Price Reference Data | Chainlink](https://data.chain.link/)

### We can get live price Chainlink API at →

[Data Feeds Contract Addresses | Chainlink Documentation](https://docs.chain.link/docs/reference-contracts/)

API Code is available in the documentation only and a link is given there to open it directly in Remix IDE

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract PriceConsumerV3 {

    AggregatorV3Interface internal priceFeed;

    /**
     * Network: Kovan
     * Aggregator: ETH/USD
     * Address: 0x9326BFA02ADD2366b30bacB125260Af641031331
     */
    constructor() {
        priceFeed = AggregatorV3Interface(0x9326BFA02ADD2366b30bacB125260Af641031331);
    }

    /**
     * Returns the latest price
     */
    function getLatestPrice() public view returns (int) {
        (
            uint80 roundID,
            int price,
            uint startedAt,
            uint timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        return price;
    }
}
```

After Deploying →

![Screenshot 2022-01-17 at 11.51.43 PM.png](Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/Screenshot_2022-01-17_at_11.51.43_PM.png)

\*\*This must be deployed on Web3 only because nodes are available in web3 not in a virtual environment

### ABI (Application Binary Interface) →

ABI tells solidity and other languages how it can interact with other contracts

If we want to interact with already deployed contract then we surely need ABI

- Interfaces compile down to ABI

### Data feed Keys are available at →

[Ethereum Data Feeds | Chainlink Documentation](https://docs.chain.link/docs/ethereum-addresses/)

```solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// importing the chainlink code for getting the price feed
// we can also copy the original code but the import statement will also work fine
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";

// Here we will make a contract that will accept payment
contract FundMe{

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
    }

    function getVersion() public view returns (uint256){
        // defining a AggregatorV3Interface instance here
        // The key provided is the chainlink price feed key that is avialable in chainlink documentation
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        return priceFeed.version();
    }

}
```

After Deployment →

![Screenshot 2022-01-18 at 1.54.22 PM.png](Solidity%20-%20Fund%20Me%200e1e08a29d2747ce99475e4679790ff2/Screenshot_2022-01-18_at_1.54.22_PM.png)

We can see the [getVersion] Function which is equal to the [version] function in [AggregatorV3Interface.sol]

### Add minimum amount & Revert transaction if condition not met

### Adding a withdraw method to withdraw all the funds that we have deposited

```solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// importing the chainlink code for getting the price feed
// we can also copy the original code but the import statement will also work fine
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
// using safe math to check for any integer overflows or underflows
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";

// Here we will make a contract that will accept payment
contract FundMe{

    using SafeMathChainlink for uint256;

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // $50
        uint256 minimumUSD = 50 * (10**18); // WEI
        require(getConversionRate(msg.value) >= minimumUSD, "You need to spend more eth"); // This will check for the condition here and revert if its not true
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
    }

    function getVersion() public view returns (uint256){
        // defining a AggregatorV3Interface instance here
        // The key provided is the chainlink price feed key that is avialable in chainlink documentation
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        return priceFeed.version();
    }

    // This will return the latest price of etherium
    function getPrice() public view returns(uint256){
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        (uint80 roundId,int256 answer,uint256 startedAt,uint256 updatedAt,uint80 answeredInRound) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }

    // 1 Eth = 1000000000 GWEI = 10^9 GWEI
    function getConversionRate(uint256 ethAmount) public view returns (uint256){
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
        // This will give us the value in USD = ethAmountInUsd * 10^-10 * 1 GWEI
    }

    // Withdrawing the amount sent to the contract
    function withdraw() payable public{
        // this keyword is same as python
        msg.sender.transfer(address(this).balance);
    }
}
```

### If we want a functionality if only owner can withdraw the funds collected

(Constructors)

```solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// importing the chainlink code for getting the price feed
// we can also copy the original code but the import statement will also work fine
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
// using safe math to check for any integer overflows or underflows
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";

// Here we will make a contract that will accept payment
contract FundMe{

    using SafeMathChainlink for uint256;

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;
    address public owner;

    // Constructor
    constructor() public {
        owner = msg.sender;
    }

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // $50
        uint256 minimumUSD = 50 * (10**18); // WEI
        require(getConversionRate(msg.value) >= minimumUSD, "You need to spend more eth"); // This will check for the condition here and revert if its not true
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
    }

    function getVersion() public view returns (uint256){
        // defining a AggregatorV3Interface instance here
        // The key provided is the chainlink price feed key that is avialable in chainlink documentation
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        return priceFeed.version();
    }

    // This will return the latest price of etherium
    function getPrice() public view returns(uint256){
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        // values between the commas will be ignored
        (,int256 answer,,,) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }

    // 1 Eth = 1000000000 GWEI = 10^9 GWEI
    function getConversionRate(uint256 ethAmount) public view returns (uint256){
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
        // This will give us the value in USD = ethAmountInUsd * 10^-10 * 1 GWEI
    }

    // Withdrawing the amount sent to the contract
    function withdraw() payable public{
        require(msg.sender == owner, "You are not allowed to withdraw"); //chacking if the owner is withdrawing or someone else
        // this keyword is same as python
        msg.sender.transfer(address(this).balance);
    }
}
```

## Modifiers

Modifiers allow us to change the behaviour of a function in a declarative way. we can customise it according to our interest

\*\* We will add only owner property using modifiers

```solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// importing the chainlink code for getting the price feed
// we can also copy the original code but the import statement will also work fine
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
// using safe math to check for any integer overflows or underflows
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";

// Here we will make a contract that will accept payment
contract FundMe{

    using SafeMathChainlink for uint256;

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;
    address public owner;

    // Constructor
    constructor() public {
        owner = msg.sender;
    }

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // $50
        uint256 minimumUSD = 50 * (10**18); // WEI
        require(getConversionRate(msg.value) >= minimumUSD, "You need to spend more eth"); // This will check for the condition here and revert if its not true
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
    }

    function getVersion() public view returns (uint256){
        // defining a AggregatorV3Interface instance here
        // The key provided is the chainlink price feed key that is avialable in chainlink documentation
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        return priceFeed.version();
    }

    // This will return the latest price of etherium
    function getPrice() public view returns(uint256){
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        // values between the commas will be ignored
        (,int256 answer,,,) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }

    // 1 Eth = 1000000000 GWEI = 10^9 GWEI
    function getConversionRate(uint256 ethAmount) public view returns (uint256){
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
        // This will give us the value in USD = ethAmountInUsd * 10^-10 * 1 GWEI
    }

    // Adding modifier to check if a owner is accessing a particular function or not
    modifier onlyOwner{
        require(msg.sender == owner, "You are not allowed to withdraw");
        _;
    }

    // Withdrawing the amount sent to the contract
    function withdraw() payable onlyOwner public{
        msg.sender.transfer(address(this).balance);
    }
}
```

## Resetting →

```solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.6.6 <0.9.0;

// importing the chainlink code for getting the price feed
// we can also copy the original code but the import statement will also work fine
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
// using safe math to check for any integer overflows or underflows
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";

// Here we will make a contract that will accept payment
contract FundMe{

    using SafeMathChainlink for uint256;

    // we made a mapping to get the amount sent to us by a particular address
    mapping(address => uint256) public addressToAmountFunded;
    address[] public funders; // array to keep track of all addresses
    address public owner;

    // Constructor
    constructor() public {
        owner = msg.sender;
    }

    // if we declare our function as payable it means that we can accept it as payment
    function fund() public payable {
        // $50
        uint256 minimumUSD = 50 * (10**18); // WEI
        require(getConversionRate(msg.value) >= minimumUSD, "You need to spend more eth"); // This will check for the condition here and revert if its not true
        // This code adds amount sent by address in the mapping
        addressToAmountFunded[msg.sender] += msg.value;
        // pushing address to funders array
        funders.push(msg.sender);
    }

    function getVersion() public view returns (uint256){
        // defining a AggregatorV3Interface instance here
        // The key provided is the chainlink price feed key that is avialable in chainlink documentation
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        return priceFeed.version();
    }

    // This will return the latest price of etherium
    function getPrice() public view returns(uint256){
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);
        // values between the commas will be ignored
        (,int256 answer,,,) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
        // returns usd value in 18 decimals
    }

    // 1 Eth = 1000000000 GWEI = 10^9 GWEI
    function getConversionRate(uint256 ethAmount) public view returns (uint256){
        uint256 ethPrice = getPrice();
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1000000000000000000;
        return ethAmountInUsd;
        // This will give us the value in USD = ethAmountInUsd * 10^-10 * 1 GWEI
    }

    // Adding modifier to check if a owner is accessing a particular function or not
    modifier onlyOwner{
        require(msg.sender == owner, "You are not allowed to withdraw");
        _;
    }

    // Withdrawing the amount sent to the contract
    function withdraw() payable onlyOwner public{
        msg.sender.transfer(address(this).balance);
        // for loop to reset the amount when all tokens withdrawn
        for (uint256 fundersIndex = 0; funderIndex < funders.length; fundersIndex++){
            address funder = funders[fundersIndex];
            addressToAmountFunded[funder] = 0;
        }
        funders = new address[]; // resetting the array
    }
}
```

### Our Full Fund Me Application is now complete
