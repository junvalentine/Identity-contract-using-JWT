# Solidity JWT validator

Original repo: [Solidity JWT validator](https://github.com/OpenZeppelin/solidity-jwt/tree/master). This repo is to fix and make it works since some libraries have deprecated.

This project includes a set of contracts and a Vue-based demo that allow you to **deploy an identity contract** and then **recover it using Google Sign-In**. This is a proof-of-concept and is definitely not ready for production :warning:

## Project setup

**Step 1:** Install Truffle, Ganache, clone the above repo and install all dependencies using command `npm install`

**Step 2:** Create a project in the [Google Developers](https://console.developers.google.com/) console then create a new OAuth Client ID. In **Authorized JavaScript origins** and **Authorized redirect URIs** set Uri to `http://localhost:8080` . Copy the client ID to `VUE_APP_GOOGLE_CLIENT_ID` param in .env file.

**Step 3:** Start a local blockchain instance using `ganache` command.  Use below curl command to request network ID from the instance and copy it to `VUE_APP_NETWORK` param.

```python
curl -X POST http://127.0.0.1:8545 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0",
    "method":"net_version",
    "params":[],
    "id":1
  }'
  # Example response: {"id":1,"jsonrpc":"2.0","result":"1732006990905"}. 
  # The network ID is 1732006990905
```

**Step 4:** Get the Google’s JWKS key from https://www.googleapis.com/oauth2/v3/certs and updates the key in `scripts/deployKeys.js` . Deploy the JWKS contract running `npx truffle exec scripts/deployKeys.js --network local` and copy the deployment address to `VUE_APP_JWKS_ADDRESS` . You will need to Base64Urlsafe decode the public params `n` and `e` and convert it to hex before adding them to `deployKeys.js` .

**Step 5:** Run application locally with `npm run serve` . 

**Step 6:** You will need Metamask to interact with the application. In Metamask, add the ganache instance network and some accounts that the instance provides.

## 3rd party smart contracts

- https://github.com/adriamb/SolRsaVerify/
- https://github.com/chrisdotn/jsmnSol
- https://github.com/Arachnid/solidity-stringutils/
