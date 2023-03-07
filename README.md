# How to Build a Requests App Using Tailwind and Celo

## Table of Content
- [How to Build a Requests App Using Tailwind and Celo](#how-to-build-a-requests-app-using-tailwind-and-celo)
  - [Table of Content](#table-of-content)
- [Introduction](#introduction)
- [Prerequisite](#prerequisite)
  - [Requirement](#requirement)
- [React App Development](#react-app-development)
  - [The Smart Contract](#the-smart-contract)
  - [Deploying smart contract to Celo](#deploying-smart-contract-to-celo)
- [Conclusion](#conclusion)


# Introduction
Hi guys, let's build a Requests application that runs on the blockchain. You can use this application to request for money from your loved ones, or family, or even friends.

One good thing about the application is that anybody who you are sending request to don't have to be registered in the app. Once they open the applicaion, they will immediately see your request, and with the click of a button, voila! your request will be granted.

Follow me through this amazing journey as we learn how to write smart contracts with Solidity, deploying the smart contracts to the Celo blockchain using Remix IDE, building a frontend for our application using React and Tailwind CSS, and a lots more, all from building this one app.

# Prerequisite
To graps the concepts and technologies used in the tutorial, you need the following:
- Knowledge of writing smart contracts with Solidity
- Knowledge of writing programs with JavaScript
- Knowledge of working with JavaScript frameworks such as React
- Knowledge of using code editors like VSCode
- Be open to gaining new knowledge

## Requirement
In order to follow this tutoiral seamlessly, you need the following:
- [VSCode]()
- [Chrome]() or [Brave]() browser
- [Celo Extension Wallet]()
- [React]()
- [Tailwind]()
- [NodeJS]()

# React App Development
The first step we will take in building this amazing app is building the frontend user interface for the app. We will be using react and tailwind CSS for the frontend development. 

Create a new directory and give it a name of your choice.
Open the newly created directory inside a code editor.
Open the directory in an terminal and run the command
```bash
npx create-react-app .
```
The command above will create a react application boilder plate inside the current directory. A react boiler plate contains the following files:

[[Plug an image of React folder structure from vscode here]()]

The next thing to do is to add tailwind to your react application. There are two ways of doing this:
1. Add tailwing to the installed packages by installing it from the command line
2. Use tailwind cdn by adding the cdn link at the top of your html file. CDN stands for Content Delivery Network

We will go for the second option (using the CDN) because it's faster and easier to use.

To use tailwind cdn, follow the steps below:
1. Navigate to your `/public` folder in the boilerplate we created
2. Open the `index.html` file and locate the meta tags in the heading.
3. Add the following line anywhere inside the `head` tag:
```html
    <script src="https://cdn.tailwindcss.com"></script>
```
4.Optionally, you can update the content of your title tag to anything of your choice. In our case, we changed it from React App to Requests App.

And that's it. You have successfully added Tailwind to your React Application.

We will now continue with building the frontend.

j

Chances are high that if ou installed the react app using the command line, you are most likely installed the latest version of react (version 18). Unfortunately, some packages we will be using for this tutorial won't be compatible with version 18 so we wll be downgrad it lower version. Open the terminal again and run teh command:

```bash
npm uninstall react react-dom react-scripts
```
The command will removed the versions of react we installed earlier an dgive room for us to instal new version of react.

```bash
npm install react@17.0.2 react-dom@17.0.2 react-scripts@4.0.1
```
The comand we wrote above will install the versions of react that is compativle with the packages will insetall next.

The next package to install is the Celo contractkit package. Run the following command to iistall it in our react project:

```bash
npm install web3 @celo/contractkit
```
also install `bignumber.js` to handle large numbers on the blockchain.
```bash
npm install bignumber.js
```

Aftr everything is done installing, run the command to start up your local server:
```bash
npm start
```
It will open a new window in your browser with your web page ready to be updated.

Open your App.js file and replace the boilerplate code inside with this:

```javascript
import React, { useState, useEffect } from 'react'
import { newKitFromWeb3 } from '@celo/contractkit'
import BigNumber from "bignumber.js";
import Web3 from 'web3';
import erc20ABI from "./contracts/erc20.abi.json"
import requestsABI from "./contracts/requests.abi.json"
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

## The Smart Contract
As stated earlier, we will use Solidity smart contract for write our smart contract. Let me show you the finished code before breaking it down into smaller snippets:

```
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
    uint256 requestsTracker;
    address cusdaddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    event MakeRequestEvent(address indexed from, address indexed to, uint256 amount);
    event CompleteRequestEvent(uint256 requestId);

    // Make new request
    function makeRequest(address _to, uint256 _amount) public {
        Request storage request = requests[requestsTracker];
        request.requestId = requestsTracker;
        request.from = payable(msg.sender);
        request.to = payable(_to);
        request.amount = _amount;
        request.completed = false;

        requestsTracker++;
        emit MakeRequestEvent(msg.sender, _to, _amount);
    }

    // Complete an existing request
    function completeRequest(uint256 _requestId) public {
        Request storage request = requests[_requestId];
        require(_requestId >= 0, "Invalid request ID");
        require(
            IERC20Token(cusdaddress).transferFrom(
                msg.sender,
                request.from,
                request.amount * 10**18
            ),
            "Transfer Unsuccessful"
        );
        request.completed = true;
        emit CompleteRequestEvent(_requestId);
    }

    // Load incoming requests from smart contract
    function loadIncomingRequests() public view returns (Request[] memory) {
        uint256 requestsCount = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].to == msg.sender) {
                requestsCount++;
            }
        }

        Request[] memory _requests = new Request[](requestsCount);
        uint256 index = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].to == msg.sender) {
                _requests[index] = requests[i];
                index++;
            }
        }

        return _requests;
    }

    // Load outgoing requests from smart contract
    function loadOutgoingRequests() public view returns (Request[] memory) {
        uint256 requestsCount = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].from == msg.sender) {
                requestsCount++;
            }
        }

        Request[] memory _requests = new Request[](requestsCount);
        uint256 index = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].from == msg.sender) {
                _requests[index] = requests[i];
                index++;
            }
        }

        return _requests;
    }
}
```

Let's break down the code into smaller snippets before digesting it.

```
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

First of all, we declared the license to use for our solidity code which is the MIT license. We then declared the solidity compiler version to be used for our code which is 0.8.7.

The next step we create an interface to be used for accessing our cUSD token on the Celo blockchain. We will be making use of the `transferFrom` method of this interface to transfer cusd tokens between usrs of our application.

After the interface, we create the body of our contract which is as below:

```
contract Requests {}
```

Solidity is similar is structure to languages like Python and C++. It has the class like construct and by default, the classes are named title cases.

This is the point where we write the code for our contract body.

```
 struct Request {
        uint256 requestId;
        address payable from;
        address payable to;
        uint256 amount;
        bool completed;
    }

    mapping (uint256 => Request) internal requests;
    uint256 requestsTracker;
    address cusdaddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    event MakeRequestEvent(address indexed from, address indexed to, uint256 amount);
    event CompleteRequestEvent(uint256 requestId);
```

Inside the contract body, we created a struct and we call it `Request`. This struct will store all teh data related to a requst made by a user.
We then created a mapping and called it `requests`. This mapping will store allthe requests ever created. The next variable is `requrestTracker` which tracks the number of requests that have been made. One last variable is `cusdAddress` this variable stores the address of cusd contract on the Celo Alfajores testnet. If yo plan to build an app for celo mainnet, check the [celo docs](https://docs.celo.org) for the mainnet address. The events we created are emitted when a request is made and a request is completed.

We will start defining the functions in our contract below:

```
 // Make new request
    function makeRequest(address _to, uint256 _amount) public {
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
Make request function above creats a new request object is got from its parameters. It sets' the `to` and `amount` value of the request from its input and sets the rest values to their default value. It then imcrememnts the requests tracker by 1 and then emit the Make requst event.

```
   // Complete an existing request
    function completeRequest(uint256 _requestId) public {
        Request storage request = requests[_requestId];
        require(_requestId >= 0, "Invalid request ID");
        require(
            IERC20Token(cusdaddress).transferFrom(
                msg.sender,
                request.from,
                request.amount * 10**18
            ),
            "Transfer Unsuccessful"
        );
        request.completed = true;
        emit CompleteRequestEvent(_requestId);
    }
```
Complete requests function completes the request sent by a user by sending the amount of tokens specified in the request and updating the request state, then emitting an event to indicate the request is completed. the function validates input, then send tokens using the ierc20 token interface we created earlier to to connect to the cusd contract using the address variable we created earlier. It throws an error if the transfer did not go through.

```
    // Load incoming requests from smart contract
    function loadIncomingRequests() public view returns (Request[] memory) {
        uint256 requestsCount = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].to == msg.sender) {
                requestsCount++;
            }
        }

        Request[] memory _requests = new Request[](requestsCount);
        uint256 index = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].to == msg.sender) {
                _requests[index] = requests[i];
                index++;
            }
        }

        return _requests;
    }
```
loadincoming request helps us to fetch all the requests that has been sent to the current user using the app. If first uses for loop to get how many request has been sent to this user and then creates a static array of that length. It then fills the array with the reqursts.

```
    // Load outgoing requests from smart contract
    function loadOutgoingRequests() public view returns (Request[] memory) {
        uint256 requestsCount = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].from == msg.sender) {
                requestsCount++;
            }
        }

        Request[] memory _requests = new Request[](requestsCount);
        uint256 index = 0;
        for (uint256 i = 0; i < requestsTracker; i++) {
            if (requests[i].from == msg.sender) {
                _requests[index] = requests[i];
                index++;
            }
        }

        return _requests;
    }
```

load outgoing requests helps us to fetch all the the requests that has been sent from this user to other users. If first loops through to get teh number of requests, then create a static length array to store this requests before returning them.

## Deploying smart contract to Celo
After completing the smart contract, what is left is to deploy this smart contract to Celo. We will archive this using Remix IDE and Celo Extenson Wallet. Follow the steps below.

1. Open remix in browser using this [link](https://remix.ethereum.org)
2. Crate a new file and paste the code from above into the file
3. Save annd compile the code
4. Download Celo Extenson wallet inyour browser
5. Download the Celo plugin in remix
6. Click on the Deploy button to deploy your contract to Celo
7. Copy the address and paste it in `requestsContract` variable inside `App.js` file

# Conclusion
Now that you have a deployed contract iisntance, go to your terminal and run the command 
```bash
npm start
```
It will open a window for you in your browser where you can interact with you newly amazing application.

[This repository](https://github.com/buchifadev/requests-app) contains all the source codes used in this tutorial.

Thank you for reading!
