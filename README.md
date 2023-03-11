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
// paste the address you saved from the previous section after deploying
// the smart contract into the requestsContractAddress variable
const requestsContractAddress = "";
const cUsdContractAddress = "0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1";

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
    try {
      await requestsContract.methods.makeRequest(requestReceiver, requestAmount).send({ from: kit.defaultAccount })
    } catch (error) {
      console.log(error)
    }
  }

  async function getIncomingRequest() {
    try {
      const requests = await requestsContract.methods.loadIncomingRequests().call({ from: kit.defaultAccount })
      const incomingRequests = await Promise.all(
        requests.map(request => {
          return {
            id: request.requestId,
            from: request.from,
            to: request.to,
            amount: request.amount,
            completed: request.completed
          }
        })
      );
      setIncomingRequests(incomingRequests)
    } catch (error) {
      console.log(error)
    }
  }

  async function getOutgoingRequests() {
    try {
      const requests = await requestsContract.methods.loadOutgoingRequests().call({ from: kit.defaultAccount })
      const outgoingRequests = await Promise.all(
        requests.map(request => {
          return {
            id: request.requestId,
            from: request.from,
            to: request.to,
            amount: request.amount,
            completed: request.completed
          }
        })
      );
      setOutgoingRequests(outgoingRequests)
    } catch (error) {
      console.log(error)
    }
  }

  async function approveRequestAmount(amount) {    
    const requestAmount = new BigNumber(amount).shiftedBy(18)
    const cusdContract = new kit.web3.eth.Contract(
      erc20ABI,
      cUsdContractAddress
    );
    await cusdContract.methods
      .approve(requestsContractAddress, requestAmount)
      .send({ from: kit.defaultAccount });
  }

  async function grantRequest(requestId, requestAmount) {
    try {
      await approveRequestAmount(requestAmount);
      await requestsContract.methods.completeRequest(requestId).send({ from: kit.defaultAccount })
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
      getOutgoingRequests();
      getIncomingRequest();
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
              <div className='bg-gray-200 mx-[10px] rounded-sm p-[5px]'>
                <span className='text-xs font-mono'>{r.from}</span> is requesting for <span className='underline'>{r.amount}</span> cUSD
                {r.completed ? <div className='font-mono text-xs underline mt-[10px]'>Granted</div> : 
                <input type={"button"} className="block bg-green-600 py-[5px] px-[15px] rounded-md mt-[15px] ml-[auto] mr-[5px] mb-[5px]" value="Grant" onClick={() => grantRequest(r.id, r.amount)} />}
                </div>
            ))
          }
        </div>
        <div className='bg-green-200 w-[400px] rounded-md flex flex-col gap-[10px]'>
          <div className='text-lg text-center underline'>Make a request for cUSD</div>
          <input type="number" onChange={e => setRequestAmount(e.target.value)} placeholder='How many cUSD are you requesting for?' className='text-sm p-[10px] m-[5px] m-[10px]' />
          <input type="text" onChange={e => setRequestReceiver(e.target.value)} placeholder='Who are you requesting cUSD from? (Enter wallet address)' className='text-sm p-[10px] m-[5px] m-[10px]' />
          <input type="button" value={"Send"} onClick={() => sendRequest()} className="rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm m-[10px]" />
        </div>
        <div className='bg-yellow-200 w-[400px] rounded-md flex gap-[10px] flex-col'>
          <div className='text-lg text-center underline'>Outgoing Requests</div>
          {
          outgoingRequests.map(r => (
            <div className='bg-gray-200 mx-[10px] rounded-sm p-[5px] '>
              You are requesting <span className='underline'>{r.amount}</span> cUSD from <span className='text-xs font-mono'>{r.from}</span>
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

We started by importing some of the packages we installed earlier to b eused inside this file. Some of them includes `Web3`, `BigNumber`, `newKitFromWeb3` etc.

We also created two variables that are very important to our application which are `cusdContractAddress` and `requestsContractAddress`.

We then created a react app component called `App`. You can give your component any name of your choice, but we will just call ours because it tallies with the file name (by convention, components name are supposed to be the same as the file name).

Inside the app component, we created some some state objects using `useState` provided by React. `useState` is React Hook that allows you to add state to a functional component. It returns an array with two values: the current state and a function to update it. The Hook takes an initial state value as an argument and returns an updated state value whenever the setter function is called.

The usestate objects created in our app will be used to store some important states and variables in our application.

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
We then created our first function and name it `connectWallet`. This function will be responsible for connecting our react application to the celo blockchain using the Celo Extension wallet installed in our browser. It also uses the Web3 library to make this connection possible. After everything is done, it saves them into the `kit` and `address` variable using their respective setter function which are `serKit` and `setAddress`.

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

The next function we created is connctContract function. This function will be responsible for connecting our contract using the contract deployed address and teh ABI generated from the contract (we will discuss about coutractadddress, abi, etc leter). The function then saves the contract in our state object called `requestsContrac`t using the setter funciton called `setRequestContract`.

The next series of functions we will be adding will be functions that adds functionalities to our dapp. The first one is `sendRequest` function. Below is the code fur the function:

```javascript
  async function sendRequest() {
    try {
      await requestsContract.methods.makeRequest(requestReceiver, requestAmount).send({ from: kit.defaultAccount })
    } catch (error) {
      console.log(error)
    }
  }
```
We made the function asyncronous by using the work `async` in front of the function. `async` means that we can use `await` inside the function.and when you use await, it means that you want to want for that particular expression fo complete execution before moving to the next expression. The function use the contract object we crated above to call the `makeRequest` method from the contracr and pass the necessary variables. 

The next functio is get incoming request.
```javascript
  async function getIncomingRequest() {
    try {
      const requests = await requestsContract.methods.loadIncomingRequests().call({ from: kit.defaultAccount })
      const incomingRequests = await Promise.all(
        requests.map(request => {
          return {
            id: request.requestId,
            from: request.from,
            to: request.to,
            amount: request.amount,
            completed: request.completed
          }
        })
      );
      setIncomingRequests(incomingRequests)
    } catch (error) {
      console.log(error)
    }
  }
```
get incoming request is responsible for fetching all the incoming requests from the contract. These are requests that are sent to this user. The function calls the loadIncomingREquest from the funcition and uses a promise to collect the data object gotten from contract. At the end of the day, it then return the data and store it inside `incomingRequests` variable.

The next function is get outgoing request:

```javascript
  async function getOutgoingRequests() {
    try {
      const requests = await requestsContract.methods.loadOutgoingRequests().call({ from: kit.defaultAccount })
      const outgoingRequests = await Promise.all(
        requests.map(request => {
          return {
            id: request.requestId,
            from: request.from,
            to: request.to,
            amount: request.amount,
            completed: request.completed
          }
        })
      );
      setOutgoingRequests(outgoingRequests)
    } catch (error) {
      console.log(error)
    }
  }
```
get outgoing request is responsible for fetching all the requests sent to this user an sstoring it to the `outGoingRequests` variable. 

the last two functions we create are 

```javascript
  async function approveRequestAmount(amount) {    
    const requestAmount = new BigNumber(amount).shiftedBy(18)
    const cusdContract = new kit.web3.eth.Contract(
      erc20ABI,
      cUsdContractAddress
    );
    await cusdContract.methods
      .approve(requestsContractAddress, requestAmount)
      .send({ from: kit.defaultAccount });
  }

  async function grantRequest(requestId, requestAmount) {
    try {
      await approveRequestAmount(requestAmount);
      await requestsContract.methods.completeRequest(requestId).send({ from: kit.defaultAccount })
    } catch (e) {
      console.log(e)
    }
  }
```

approveRequestAmount() approves the our requests contract to spend the specific amount of cUSD from ouraccount using the erc20 approve method. It first gets the cusd contract before calling the approve method from it.

grantRequest() completes a request that was sent to this user. It first of all calls the approveRequest function to approve the amount to be granted, before calling our request scontract to transfer the amount from this user's accout balance to the user making the request.

Additionally, we added some use effect hook to update the the variables in the use state objects we created in the app componnent. use effect is similar to use state but the difference is that is is used to make dom updates when ever somehting happens in our app.

In the last part of our app, we returned the user interface code that will be displayed when the user visits our app. We used tailwing to make it look better and add the functionalities to it by adding integerating the functions we just created above. There is nothing much in the page, it's just basic html with class names that contained tailwind styles. 

Visit the [Tailwind documentation site](https://tailwindcss.com/docs/installation) to learn how to user tailwind if you are new to using it.



## Conclusion
Now that you have a deployed contract iisntance, go to your terminal and run the command 
```bash
npm start
```
It will open a window for you in your browser where you can interact with you newly amazing application.

[This repository](https://github.com/buchifadev/requests-app) contains all the source codes used in this tutorial.

Thank you for reading!
