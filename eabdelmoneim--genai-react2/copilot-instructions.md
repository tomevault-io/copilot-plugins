## genai-react2

> You are an expert in thirdweb, typescript, node.js, next.js, react, shadcn UI, tailwind css.

You are an expert in thirdweb, typescript, node.js, next.js, react, shadcn UI, tailwind css.

thirdweb SDK usage
- you should use v5 of the sdk which is in the thirdweb package
- you should use built-in extensions like erc1155, erc721, etc that are in the thirdweb/extensions package when possible instead of trying to call smart contract functions directly
- when building a front end, use hooks like useOwnedNFTs, useContract, useNFT, etc which are in the thirdweb/react package
- NEVER EVER include any functions that are in v4 of the sdk which means NEVER EVER include modules from the thirdweb-dev package
- if writing a script or using the thirdweb sdk in a server side environment, the thirdweb client should be initialized with the secret key.  otheriwse if it is a client side application, initialize the client with the client id
- if the chain being connected to is a number, you can use the thirdweb/chains defineChain(chainId) function to get the chain object
- private keys, secret keys, client IDs should be stored in .env files and loaded using the dotenv package
- never put private keys, secret keys, client IDs, or any other sensitive information in a code repository or make it public. make sure the .env files are not committed to the repository
- when using thirdweb React SDK remember that at the root of the application you need to wrap the application in the ThirdwebProvider component so you must add this ThirdwebProvider component to the application the first time you use the thirdweb sdk

Typescript usage
- always use types and interfaces
- always use type assertions when defining parameters or variables
- always use interfaces to define the shape of objects
- always use types to define the type of a variable

UI and styling
- If you are using UI components for web3 elements like a wallet selector or sending transactions on-chain, use the UI componenet found in thirdweb/react package
- Use shadcn UI and tailwind css for components and styling
- implement responsive design with tailwind css; use a mobile first approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eabdelmoneim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
