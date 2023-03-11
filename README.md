# How to Build a Requests App Using Tailwind and Celo

## Table of Content
- [How to Build a Requests App Using Tailwind and Celo](#how-to-build-a-requests-app-using-tailwind-and-celo)
  - [Table of Content](#table-of-content)
  - [Introduction](#introduction)
  - [Prerequisite](#prerequisite)
  - [Requirement](#requirement)
  - [The Smart Contract](#the-smart-contract)
  - [Deploying Smart Contract to Alfajores](#deploying-smart-contract-to-alfajores)
  - [React App Development](#react-app-development)
  - [Conclusion](#conclusion)


## Introduction
Hi guys, let's build a Requests dapp that runs on the blockchain. You can use this dapp to request the Celo stablecoin cUSD (Celo Dollar) tokens from other users.

One good thing about the dapp is that anybody you are sending a *request* to doesn't have to be registered in the dapp. Once they open the application, they will immediately see your request, and with the click of a button, voila! Your request will be granted.

Follow me through this amazing journey as we learn how to write smart contracts with Solidity, deploy our smart contract to the Celo blockchain using the Remix IDE, build a front end for our decentralized application using React and Tailwind CSS, and a lot more, all from building this one dapp.

## Prerequisite
To grasp the concepts and technologies used in the tutorial, you need the following:
- Knowledge of writing smart contracts with [Solidity](https://soliditylang.org/)
- Knowledge of writing programs with [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- Knowledge of working with JavaScript frameworks such as [React](https://reactjs.org/)
- Knowledge of using code editors like VS Code
- Be open to gaining new knowledge

## Requirement
To follow this tutorial seamlessly, you need to have the following installed:
- [VSCode](https://code.visualstudio.com/)
- [Chrome](https://www.google.com/chrome/) or [Brave](https://brave.com/) browser
- [Celo Extension Wallet](https://docs.celo.org/wallet#celoextensionwallet)
- [Node.js](https://nodejs.org/en/)


## The Smart Contract
As stated earlier, we will use Solidity to write our smart contract. Let me show you the finished code before breaking it down into smaller snippets:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

interface IERC20Token {
    function transfer(address, uint256) external returns (bool);
    function approve(address, uint256) external returns (bool);
    function transferFrom(address, address, uint256) external returns (bool);
    function totalSupply() external view returns (uint256);
    function balanceOf(address) external view returns (uint256);
    function allowance(address, address) external view returns (uint256);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

contract Requests {
    
    struct Request {
        uint256 requestId;
        address payable from;
        address payable to;
        uint256 amount;
        bool completed;
    }

    mapping (uint256 => Request) internal requests;
    uint256 public requestsTracker;
    address cUsdTokenAddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    event MakeRequestEvent(address indexed from, address indexed to, uint256 amount);
    event CompleteRequestEvent(uint256 requestId);

    /**
        * @dev Validation checks are carried out to ensure valid input data
        * @notice Allow users to make requests for cUSD tokens from other users
        * @param _to The address to send the request to
        * @param _amount The cUSD amount for the request
    */
    function makeRequest(address _to, uint256 _amount) public {
        require(_to != address(0) && _to != msg.sender, "Invalid receiver address.");
        require(_amount >= 1 ether, "Amount needs to be at least one cUSD");
        Request storage request = requests[requestsTracker];
        request.requestId = requestsTracker;
        request.from = payable(msg.sender);
        request.to = payable(_to);
        request.amount = _amount;
        request.completed = false;

        requestsTracker++;
        emit MakeRequestEvent(msg.sender, _to, _amount);
    }

    /**
        * @dev Checks if the sender is the receiver of the request and the request isn't yet fulfilled
        * @notice Allow users to complete requests they received
        * @param _requestId The ID of a request
    */
    function completeRequest(uint256 _requestId) public {
        Request storage request = requests[_requestId];
        require(_requestId < requestsTracker, "Invalid request ID.");
        require(!request.completed, "Request has already been fulfilled.");
        require(request.to == msg.sender, "Unauthorized sender.");
        require(
            IERC20Token(cUsdTokenAddress).transferFrom(
                msg.sender,
                request.from,
                request.amount
            ),
            "Transfer Unsuccessful"
        );
        request.completed = true;
        emit CompleteRequestEvent(_requestId);
    }

    /**
        * @notice Allow users to fetch data of a request
        * @param _requestId The ID of a request
        * @return Array an array of properties of a Request struct
    */
    function getRequest(uint _requestId) public view returns (Request memory){
        require(_requestId < requestsTracker, "Invalid request ID.");
        return requests[_requestId];
    }
}
```

Let's break down the code into smaller snippets before digesting it.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

interface IERC20Token {
    function transfer(address, uint256) external returns (bool);
    function approve(address, uint256) external returns (bool);
    function transferFrom(address, address, uint256) external returns (bool);
    function totalSupply() external view returns (uint256);
    function balanceOf(address) external view returns (uint256);
    function allowance(address, address) external view returns (uint256);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

First of all, we declared the [SPDX](https://spdx.dev/) license to use for our Solidity code which is the [MIT](https://spdx.org/licenses/MIT.html) license. We then declared the Solidity compiler version to be used for our code which is version **0.8.7**.

In the next step, we create an [interface](https://docs.soliditylang.org/en/v0.8.19/contracts.html#interfaces) to be used for accessing our cUSD token on the Celo blockchain. We will be making use of the `transferFrom()` method of this interface to transfer cUSD tokens between users of our application.

After the interface, we create the body of our contract which is as below:

```solidity
contract Requests {}
```
This is the point where we write the code for our contract body.

```solidity
    struct Request {
        uint256 requestId;
        address payable from;
        address payable to;
        uint256 amount;
        bool completed;
    }

    mapping (uint256 => Request) internal requests;
    uint256 public requestsTracker;
    address cUsdTokenAddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    event MakeRequestEvent(address indexed from, address indexed to, uint256 amount);
    event CompleteRequestEvent(uint256 requestId);
```

Inside the contract body, we created a struct and we call it `Request`. This struct will store the following properties:
1. `requestID` - The ID of a request.
2. `from` - The user address that initiated the request.
3. `to` - The recipient address of the request.
4. `amount` - The cUSD amount requested.
5. `completed` - A boolean keeping track of whether a request has been fulfilled.

We then created a mapping and called it `requests`. This mapping will store all the requests ever created. The next variable is `requestsTracker` which tracks the number of requests that have been made. The last variable is `cUsdTokenAddress` which stores the address of the cUSD contract's address on the Celo Alfajores testnet. If you plan to build an app for the Celo mainnet, check the [Celo docs](https://docs.celo.org) for the mainnet address. The events we created are emitted when a request is made and a request is completed. Finally, we created two events `MakeRequestEvent` and `CompleteRequestEvent` which we will emit when calling the `makeRequest()` and `completeRequest` functions respectively.

We will start defining the functions in our contract below:

```solidity
    /**
        * @dev Validation checks are carried out to ensure valid input data
        * @notice Allow users to make requests for cUSD tokens from other users
        * @param _to The address to send the request to
        * @param _amount The cUSD amount for the request
    */
    function makeRequest(address _to, uint256 _amount) public {
        require(_to != address(0) && _to != msg.sender, "Invalid receiver address.");
        require(_amount >= 1 ether, "Amount needs to be at least one cUSD");
        Request storage request = requests[requestsTracker];
        request.requestId = requestsTracker;
        request.from = payable(msg.sender);
        request.to = payable(_to);
        request.amount = _amount;
        request.completed = false;

        requestsTracker++;
        emit MakeRequestEvent(msg.sender, _to, _amount);
    }
```
The `makeRequest()` function above will be used to create new requests. The function takes the following parameters:
1. `_to` - The recipient address of a request
2. `_amount` - The cUSD amount of a request

Next, the function performs a few validation checks on the input arguments sent to ensure that the `_to` address is not an invalid address(i.e. the zero address or the address of the sender of the request) and that the `_amount` is at least one cUSD. If any of the checks fail, the transaction is reverted with a custom error message, otherwise, the input arguments are used to create a new `Request` struct and the functions then stores it in the `requests` mapping. Finally, the `requestsTracker` is incremented and the `MakeRequestEvent` is emitted.

We will now create the `completeRequest()` function:

```solidity
    /**
        * @dev Checks if the sender is the receiver of the request and the request isn't yet fulfilled
        * @notice Allow users to complete requests they received
        * @param _requestId The ID of a request
    */
    function completeRequest(uint256 _requestId) public {
        Request storage request = requests[_requestId];
        require(_requestId < requestsTracker, "Invalid request ID.");
        require(!request.completed, "Request has already been fulfilled.");
        require(request.to == msg.sender, "Unauthorized sender.");
        require(
            IERC20Token(cUsdTokenAddress).transferFrom(
                msg.sender,
                request.from,
                request.amount
            ),
            "Transfer Unsuccessful"
        );
        request.completed = true;
        emit CompleteRequestEvent(_requestId);
    }
```
The `completeRequest()` function will be used to complete pending requests. The function has a parameter `_requestId` which will be used to update the state of the `Request` struct at the index `_requestId` inside the `requests` mapping. The function also carries out the following checks:
1. The `_requestId` is checked to be valid by comparing it to the current value of `requestsTracker` as valid IDs will always be less than the current value stored.
2. The value of the `completed` property is checked to be **false** as only pending and incomplete requests should be accessible using the `completeRequest()` function.
3. The next `require` statement ensures that the `msg.sender` is the recipient of the address by comparing it with the `to` property of the `Request` struct.

The function then carries out the cUSD transfer by calling the `transferFrom()` method of the cUSD token. The previous operation is inside a `require` statement to ensure that in case of failure, the execution stops and the function reverts the transaction. Finally, if nothing went wrong, the function updates the `completed` property to **true** and emits the `CompleteRequestEvent` event.

Finally the last function we will define in our smart contract is the `getRequest()` function:

```solidity
    /**
        * @notice Allow users to fetch data of a request
        * @param _requestId The ID of a request
        * @return Array an array of properties of a Request struct
    */
    function getRequest(uint _requestId) public view returns (Request memory){
        require(_requestId < requestsTracker, "Invalid request ID.");
        return requests[_requestId];
    }
```

The `getRequest()` function takes the parameter `_requestId` which is used to return a `Request` struct at an index inside the `requests` mapping.

## Deploying Smart Contract to Alfajores
After completing the smart contract, what is left is to deploy the smart contract to the Celo Alfajores network. We will achieve this using the Remix IDE and Celo Extension Wallet. Follow the steps below:

1. Open Remix in a browser using this [link](https://remix.ethereum.org)
2. Create a new file and paste the smart contract code into the file
3. Save and compile the file
4. Download the Celo Extension Wallet in your browser
5. Activate the Celo plugin in Remix
6. Click on the **Deploy** button to deploy your contract to the Celo Alfajores network
7. Copy and save the address as we will later use it inside the `App.js` file

## React App Development

We will now build the front-end user interface of the dapp. We will be using React and Tailwind CSS. 

Create a new directory and give it a name of your choice.
Open the newly created directory inside a code editor.
Open the directory in a terminal and run the command

```bash
npx create-react-app .
```
The command above will create a React application boilerplate inside the current directory. A React boilerplate contains the following files:

[[Plug an image of React folder structure from vscode here]()]

The next thing to do is to add Tailwind to your React application. There are two ways of doing this:
1. Add Tailwind to the installed packages by installing it from the command line
2. Use the Tailwind CDN by adding the CDNlink at the top of your HTML file. CDN stands for Content Delivery Network

We will go for the second option (using the CDN) because it's faster and easier to use.

To use the Tailwind CDN, follow the steps below:
1. Navigate to your `/public` folder in the boilerplate we created
2. Open the `index.html` file and locate the meta tags in the heading.
3. Add the following line anywhere inside the `head` tag:

```html
    <script src="https://cdn.tailwindcss.com"></script>
```
Optionally, you can update the content of your `title` tag to anything of your choice. In our case, we changed it from React App to Requests App.

And that's it. You have successfully added Tailwind to your React Application.

We will now continue with building the front end.

Chances are high that if you installed the React app using the command line, you have most likely installed the latest version of react (version 18). Unfortunately, some packages we will be using for this tutorial won't be compatible with version 18 so we will be downgrading to a lower version. Open the terminal again and run the command:

```bash
npm uninstall react react-dom react-scripts
```
The command will remove the versions of React we installed earlier and give room for us to install an older version of React.

```bash
npm install react@17.0.2 react-dom@17.0.2 react-scripts@4.0.1
```
The command we wrote above will install the versions of React that is compatible with the packages that we will install next.

The next package to install is the Celo **contractkit** and **web3** package. Run the following command to install it in our React project:

```bash
npm install web3 @celo/contractkit
```
Also, install `bignumber.js` as it will help us to handle large numbers on the front-end.
```bash
npm install bignumber.js
```

After everything is done installing, run this command to start up your local server:
```bash
npm start
```
It will open a new window in your browser with your web page ready to be updated.

Open your `App.js` file and replace the boilerplate code inside with this:

```javascript
import React, { useState, useEffect } from 'react'
import { newKitFromWeb3 } from '@celo/contractkit'
import BigNumber from "bignumber.js";
import Web3 from 'web3';
import erc20ABI from "./contracts/erc20.abi.json"
import requestsABI from "./contracts/requests.abi.json"
// replace with the saved contract address here
const requestsContractAddress = "";
const cUsdContractAddress = "0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1";
const ERC20_DECIMALS = 18;

const App = () => {
  const [kit, setKit] = useState("")
  const [address, setAddress] = useState("")
  const [requestsContract, setRequestsContract] = useState("")
  const [requestReceiver, setRequestReceiver] = useState("")
  const [requestAmount, setRequestAmount] = useState(0);
  const [outgoingRequests, setOutgoingRequests] = useState([])
  const [incomingRequests, setIncomingRequests] = useState([])

  async function connectWallet() {
    if (window.celo) {
      try {
        await window.celo.enable();
        const web3 = new Web3(window.celo);
        let kit = newKitFromWeb3(web3)
        const account = await kit.web3.eth.getAccounts();
        const defaultAccount = account[0];
        kit.defaultAccount = defaultAccount;
        setKit(kit)
        setAddress(defaultAccount)
      } catch (e) {
        console.log(e)
      }
    } else {
      alert("Please install CeloExtensionWallet to continue with this app")
    }
  }

  async function connectContract() {
    try {
      const requests = new kit.web3.eth.Contract(requestsABI, requestsContractAddress);
      setRequestsContract(requests)
    } catch (e) {
      console.log(e)
    }
  }

  async function sendRequest() {
    const oneCusd = new BigNumber("1").shiftedBy(ERC20_DECIMALS).toString();
    try {
      if(requestReceiver == kit.defaultAccount || requestReceiver == "0x0000000000000000000000000000000000000000"){
        alert("Invalid receiver address");
        return;
      }
      if(!new BigNumber(requestAmount).isGreaterThanOrEqualTo(oneCusd)){
        alert("Request amount needs to be at least One cUSD");
        return;
      }
      await requestsContract.methods.makeRequest(requestReceiver, requestAmount).send({ from: kit.defaultAccount })
      fetchRequests()
    } catch (error) {
      console.log(error)
    }
  }

  async function fetchRequests(){
    try{
      const _requestsTracker = await requestsContract.methods.requestsTracker().call();
      const _incomingRequests = [];
      const _outgoingRequests = [];
      let _requests = [];
      for(let i = 0; i < Number(_requestsTracker); i++){
        const _request = new Promise(async (resolve, reject) => {
          const r = await requestsContract.methods.getRequest(i).call();
          resolve({
            id: r[0],
            from: r[1],
            to: r[2],
            amount: new BigNumber(r[3]),
            completed: r[4]
          })
        })
        _requests.push(_request);
      }
      _requests = await Promise.all(_requests);
      _requests.forEach((r) => {
        if(r.to == kit.defaultAccount){
          _incomingRequests.push(r)
        }else if(r.from == kit.defaultAccount){
          _outgoingRequests.push(r)
        }
      })
      setOutgoingRequests(_outgoingRequests);
      setIncomingRequests(_incomingRequests);
    }catch(e) {
      console.log(e)
    }
  }

  async function approveRequestAmount(amount) {  
    const cusdContract = new kit.web3.eth.Contract(
      erc20ABI,
      cUsdContractAddress
    );
    await cusdContract.methods
      .approve(requestsContractAddress, amount)
      .send({ from: kit.defaultAccount });
  }

  async function grantRequest(requestId, _requestAmount) {
    try {
      await approveRequestAmount(_requestAmount);
      await requestsContract.methods.completeRequest(requestId).send({ from: kit.defaultAccount })
      fetchRequests()
    } catch (e) {
      console.log(e)
    }
  }

  useEffect(() => {
    connectWallet()
  }, [])

  useEffect(() => {
    if (kit && address) {
      connectContract()
    }
  }, [kit, address])

  useEffect(() => {
    if (requestsContract) {
      fetchRequests();
    }
  }, [requestsContract])

  return (
    <div className='w-5/6 bg-red-100 m-auto min-h-screen'>
      <div className='bg-green-200 p-2 m-[auto] w-fit rounded-md font-mono'>{address}</div>
      <div className='flex gap-[10px] w-full justify-around mt-[30px]'>
        <div className='bg-gray-300 w-[400px] min-h-[400px] rounded-md flex gap-[10px] flex-col'>
          <div className='text-lg text-center underline'>Incoming Requests</div>
          {
            incomingRequests.map(r => (
              <div className='bg-gray-200 mx-[10px] rounded-sm p-[5px]' key={r.id}>
                <span className='text-xs font-mono'>{r.from}</span> is requesting for <span className='underline'>{r.amount.shiftedBy(-ERC20_DECIMALS).toFixed(2)}</span> cUSD
                {r.completed ? <div className='font-mono text-xs underline mt-[10px]'>Granted</div> : 
                <input type={"button"} className="block bg-green-600 py-[5px] px-[15px] rounded-md mt-[15px] ml-[auto] mr-[5px] mb-[5px]" value="Grant" onClick={() => grantRequest(r.id, r.amount)} />}
                </div>
            ))
          }
        </div>
        <div className='bg-green-200 w-[400px] rounded-md flex flex-col gap-[10px]'>
          <div className='text-lg text-center underline'>Make a request for cUSD</div>
          <input type="number" onChange={e => setRequestAmount(new BigNumber(e.target.value).shiftedBy(ERC20_DECIMALS).toString())} placeholder='How many cUSD are you requesting for?' className='text-sm p-[10px] m-[5px] m-[10px]' />
          <input type="text" onChange={e => setRequestReceiver(e.target.value)} placeholder='Who are you requesting cUSD from? (Enter wallet address)' className='text-sm p-[10px] m-[5px] m-[10px]' />
          <input type="button" value={"Send"} onClick={() => sendRequest()} className="rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm m-[10px]" />
        </div>
        <div className='bg-yellow-200 w-[400px] rounded-md flex gap-[10px] flex-col'>
          <div className='text-lg text-center underline'>Outgoing Requests</div>
          {
          outgoingRequests.map(r => (
            <div className='bg-gray-200 mx-[10px] rounded-sm p-[5px] '  key={r.id}>
              You are requesting <span className='underline'>{r.amount.shiftedBy(-ERC20_DECIMALS).toFixed(2)}</span> cUSD from <span className='text-xs font-mono'>{r.to}</span>
              <div className='font-mono text-xs underline mt-[10px]'>{r.completed? "Completed": "Pending..."}</div>
            </div>
          ))
        }
        </div>
      </div>
    </div>
  )
}

export default App
```

Follow along as we break down the code into simple and understandable snippets.

We first import some of the packages we installed earlier to be used inside this file. Some of them include `Web3`, `BigNumber`, `newKitFromWeb3` etc.

We also created two variables that are very important to our application which are `cUsdContractAddress` and `requestsContractAddress`.

We then created a React app component called `App`. You can give your component any name of your choice, but we will just call ours because it tallies with the file name (by convention, components names are supposed to be the same as the file name).

Inside the app component, we created some state objects using `useState` provided by React. `useState` is React Hook which allows you to add state to a functional component. It returns an array with two values: the current state and a function to update it. The hook takes an initial state value as an argument and returns an updated state value whenever the setter function is called.

The `useState` objects created in our app will be used to store some important states and variables in our application.

```javascript
  async function connectWallet() {
    if (window.celo) {
      try {
        await window.celo.enable();
        const web3 = new Web3(window.celo);
        let kit = newKitFromWeb3(web3)
        const account = await kit.web3.eth.getAccounts();
        const defaultAccount = account[0];
        kit.defaultAccount = defaultAccount;
        setKit(kit)
        setAddress(defaultAccount)
      } catch (e) {
        console.log(e)
      }
    } else {
      alert("Please install CeloExtensionWallet to continue with this app")
    }
  }
```
We then created our first function and name it `connectWallet`. This function will be responsible for connecting our React application to the Celo blockchain using the Celo Extension Wallet installed in our browser. It also uses the `Web3` library to make this connection possible. After everything is done, it saves them into the `kit` and `address` variables using their respective setter function which are `setKit` and `setAddress`.

```javascript
  async function connectContract() {
    try {
      const requests = new kit.web3.eth.Contract(requestsABI, requestsContractAddress);
      setRequestsContract(requests)
    } catch (e) {
      console.log(e)
    }
  }
```

The next function we created is the `connectContract()` function. This function will be responsible for connecting our contract using the contract deployed address and the ABI generated from the contract (we will discuss the requestsContractAddress, ABI, etc later). The function then saves the contract in our state object called `requestsContract` using the setter function called `setRequestContract`.

The next series of functions we will be adding will be functions that add functionalities to our dapp. The first one is the `sendRequest()` function. Below is the code fur the function:

```javascript
  async function sendRequest() {
    const oneCusd = new BigNumber("1").shiftedBy(ERC20_DECIMALS).toString();
    try {
      if(requestReceiver == kit.defaultAccount || requestReceiver == "0x0000000000000000000000000000000000000000"){
        alert("Invalid receiver address");
        return;
      }
      if(!new BigNumber(requestAmount).isGreaterThanOrEqualTo(oneCusd)){
        alert("Request amount needs to be at least One cUSD");
        return;
      }
      await requestsContract.methods.makeRequest(requestReceiver, requestAmount).send({ from: kit.defaultAccount })
      fetchRequests()
    } catch (error) {
      console.log(error)
    }
  }
```
We made the function asynchronous by using the word `async` in front of the function. `async` means that we can use `await` inside the function and when you use await, it means that you want the function to wait for a pending `Promise` before continuing the function execution. The function uses the contract object we created above to call the `makeRequest` method from the contract and pass the necessary variables.

>**_Note:_** We carry out the same validation checks we are using on the smart contract which we have already covered in the [The Smart Contract](#the-smart-contract) section.

The next function is the `fetchRequests()` function:

```javascript
  async function fetchRequests(){
    try{
      const _requestsTracker = await requestsContract.methods.requestsTracker().call();
      const _incomingRequests = [];
      const _outgoingRequests = [];
      let _requests = [];
      for(let i = 0; i < Number(_requestsTracker); i++){
        const _request = new Promise(async (resolve, reject) => {
          const r = await requestsContract.methods.getRequest(i).call();
          resolve({
            id: r[0],
            from: r[1],
            to: r[2],
            amount: new BigNumber(r[3]),
            completed: r[4]
          })
        })
        _requests.push(_request);
      }
      _requests = await Promise.all(_requests);
      _requests.forEach((r) => {
        if(r.to == kit.defaultAccount){
          _incomingRequests.push(r)
        }else if(r.from == kit.defaultAccount){
          _outgoingRequests.push(r)
        }
      })
      setOutgoingRequests(_outgoingRequests);
      setIncomingRequests(_incomingRequests);
    }catch(e) {
      console.log(e)
    }
  }

```

The `fetchRequests()` function fetches all the requests created from our smart contract using a `for` loop and [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). After all requests have been fetched, we loop through them and only save incoming and outgoing requests made to the connected wallet address to the front-end state.

Next, we will define the `approveRequestAmount()` and the `grantRequest()` functions:

```javascript
  async function approveRequestAmount(amount) {  
    const cusdContract = new kit.web3.eth.Contract(
      erc20ABI,
      cUsdContractAddress
    );
    await cusdContract.methods
      .approve(requestsContractAddress, amount)
      .send({ from: kit.defaultAccount });
  }

  async function grantRequest(requestId, _requestAmount) {
    try {
      await approveRequestAmount(_requestAmount);
      await requestsContract.methods.completeRequest(requestId).send({ from: kit.defaultAccount })
      fetchRequests()
    } catch (e) {
      console.log(e)
    }
  }
```

The `approveRequestAmount()` function approves our smart contract to spend the request's amount from the connected account using the ERC20 's `approve()` method. It first creates an instance of the cUSD contract before calling the `approve()` method from it.

The `grantRequest()` function completes a request that was sent to the connected user. It, first of all, calls the `approveRequest()` function to approve the amount to be granted, before calling our request contract to transfer the amount from the connected account's balance to the user making the request.

Additionally, we added some `useEffect` hooks to update the variables in the use state objects we created in the `App` component. 

In the last part of our app, we returned the user interface code that will be displayed when the user visits our app. We used Tailwind to make it look better and add functionalities to it by integrating the functions we just created above. There is nothing much on the page, it's just basic HTML with class names that contained tailwind styles. 

Visit the [Tailwind documentation site](https://tailwindcss.com/docs/installation) to learn how to use tailwind if you are new to using it.



## Conclusion
Now that you have a deployed contract instance, go to your terminal and run the command 

```bash
npm start
```
It will open a window for you in your browser where you can interact with your new amazing application.

[This repository](https://github.com/buchifadev/requests-app) contains all the source codes used in this tutorial.

Thank you for reading!
