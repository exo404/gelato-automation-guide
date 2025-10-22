# Solidity Functions automation

## All Checks On-chain?

If all the conditions necessary for your automation task can be directly verified on the blockchain, you have the option to select between Typescript Functions, Solidity Functions & Automated Transactions.

## Implementation path
* Start by deciding on the type of trigger you want to use:
  - Time Interval: use this trigger to execute tasks at regular intervals, e.g., every 10 minutes.
  - Cron Expressions: this offers a more refined control compared to the Time Interval. Check [cron expressions](https://en.wikipedia.org/wiki/Cron), 
  - On-Chain Event: Ideal for those wanting their tasks to respond dynamically to blockchain activities.
  - Every Block: executing your chosen function each time a new block is created.
* Create a Web3 Function task to allow the solidity execution 
* Once you've defined your function ensure you monitor its execution to confirm that it works as expected. Make any necessary adjustments.
   
## Pre-Requisite of Target Smart Contract

Smart contract functions in the target contract that can be automated should follow these properties:

* They need to be functions that are usually called by the development team or external keepers, not "user facing" functions called by users directly
* They need to be either public or external
* They do not have access restrictions like an onlyOwner modifier, unless the user's dedicated msg.sender address is whitelisted through the proxy module
* They do not require msg.sender to be tx.origin

## Write Solidity Functions

### 1. Understand the Role of a Checker

A Checker acts as a bridge between conditions and smart contract executions. Its purpose? To check conditions and determine whether a task should be executed by Gelato. Every Checker returns two main things:

* `canExec` (Boolean): Indicates if Gelato should execute the task.
* `execData` (Bytes): Contains the data that executors will use during execution.

<Note>
  Solidity functions must adhere to the block gas limit for checker calls; exceeding it will cause the call to fail.
</Note>

### 2. Solidity Function Example

```solidity  theme={null}
contract CounterChecker{
    ICounter public immutable counter;

    constructor(ICounter _counter) {
        counter = _counter;
    }

    function checker()
        external
        view
        returns (bool canExec, bytes memory execPayload)
    {
        uint256 lastExecuted = counter.lastExecuted();

        if(block.timestamp - lastExecuted < 180) return(false, bytes("Time not elapsed"));

        canExec = (block.timestamp - lastExecuted) > 180;

        execPayload = abi.encodeCall(ICounter.increaseCount, (1));
    }
}
```
In the above, the checker checks the state of a counter and prompts Gelato to execute if 3 minutes (180 seconds) have elapsed since its last execution.


### 3. Checking Multiple Functions

Suppose you're automating tasks across different pools. Instead of creating multiple tasks, iterate through your list of pools within a single checker:

```solidity  theme={null}
function checker()
    external
    view
    returns (bool canExec, bytes memory execPayload)
{
    uint256 delay = harvester.delay();

    for (uint256 i = 0; i < vaults.length(); i++) {
        IVault vault = IVault(getVault(i));

        canExec = block.timestamp >= vault.lastDistribution().add(delay);

        execPayload = abi.encodeWithSelector(
            IHarvester.harvestVault.selector,
            address(vault)
        );

        if (canExec) return(true, execPayload);
    }

    return(false, bytes("No vaults to harvest"));
}
```
### 4. Limit the Gas Price of your execution

On networks such as Ethereum, gas will get expensive at certain times. If what you are automating is not time-sensitive and don't mind having your transaction mined at a later point, you can limit the gas price used in your execution in your checker.

```solidity  theme={null}
function checker()
    external
    view
    returns (bool canExec, bytes memory execPayload)
{
    // condition here
    
    if(tx.gasprice > 80 gwei) return (false, bytes("Gas price too high"));
}
```
This way, Gelato will not execute your transaction if the gas price is higher than 80 GWEI.

## Deploy Solidity Functions

To deploy your Solidity functions, please proceed with deploying your contract to the network. Once deployed, ensure you verify your contract on Etherscan to enable automatic ABI fetching within our app.

### Creating Solidity Function Tasks

<img src="https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=c9996862fd394b5b4b09ef7ab9c7aafb" alt="Creating Solidity Function Tasks" data-og-width="845" width="845" data-og-height="630" height="630" data-path="images/creating_solidity_function_tasks.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=280&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=1c45e6c7fe20ead0edf2597282414253 280w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=560&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=dfa9058a7740050cdd86af43c1096043 560w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=840&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=5a373ada9b1fc91483a9360a951cf978 840w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=1100&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=019e1b91cf99c9c93d1fa6f50d9ed70f 1100w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=1650&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=a15ab8fa08eabd9deb97a649a0a5ae7c 1650w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/creating_solidity_function_tasks.png?w=2500&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=7d0d740a84f7d1f152f005a5f8c51009 2500w" />

1. **Selection of Function**
   * Navigate to the What to trigger section.
   * Choose the Solidity Function option

2. **Network Configuration**
   * Locate the Network dropdown.
   * Select your desired blockchain network where the contract is deployed, e.g., "Göerli."

3. **Function Details Input**
   * Under the Solidity Function section, find the input labeled Contract Address.
   * Enter the Ethereum address of your deployed Solidity contract. Ensure accuracy as this determines where your functions will interact.
   * Once the contract address is entered, the ABI (Application Binary Interface) should automatically populate. If using a custom ABI, select the Custom ABI option and input it accordingly.

4. **Task Configuration**
   * A checker function evaluates conditions before triggering the main function. From the Checker Function dropdown, choose the specific function you want as a condition checker.
   * Enter the Target Contract where the automated function call should be sent. From the subsequent dropdown, select the specific function you wish to automate.

<img src="https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=044f27813a89223940e071bf204a72fe" alt="Task Configuration" data-og-width="828" width="828" data-og-height="611" height="611" data-path="images/task_config.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=280&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=a83cdf998bb4ee12fa9ba279ca91b1bc 280w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=560&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=08b3d096c048d2397221c2ee55b59a10 560w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=840&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=0d1da2b8713c017e13c5732e512b44a4 840w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=1100&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=68b41af01172bcee2d09bcaab67cd692 1100w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=1650&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=e4c80630b9ed7460f35b02da597587a3 1650w, https://mintcdn.com/gelato-6540eeb1/8BeiyOhSgNF7mtMT/images/task_config.png?w=2500&fit=max&auto=format&n=8BeiyOhSgNF7mtMT&q=85&s=3aaa2c71df833fd6d0074a0353aeb72e 2500w" />


## Resources
cc: [Gelato Docs](https://docs.gelato.cloud/)
