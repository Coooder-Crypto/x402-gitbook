# Quickstart for Buyers

This guide walks you through how to use **x402** to interact with services that require payment. By the end of this guide, you will be able to programmatically discover payment requirements, complete a payment, and access a paid resource.

### Prerequisites

Before you begin, ensure you have:

* A crypto wallet with USDC (any EVM-compatible wallet, e.g., [CDP Wallet](https://docs.cdp.coinbase.com/wallet-api-v2/docs/welcome))
* [Node.js](https://nodejs.org/en) and npm, or Python and pip
* A service that requires payment via x402

**Note**\
We have pre-configured [examples available in our repo](https://github.com/coinbase/x402/tree/main/examples), including for MCP.

### 1. Install Dependencies

{% tabs %}
{% tab title="Node.js" %}
Install [x402-axios](https://www.npmjs.com/package/x402-axios) or [x402-fetch](https://www.npmjs.com/package/x402-fetch):

```bash
npm install x402-axios
# or
npm install x402-fetch
```
{% endtab %}

{% tab title="Python" %}
Install the [x402 package](https://pypi.org/project/x402/)

```
pip install x402
```
{% endtab %}
{% endtabs %}



### 2. Create a Wallet Client

#### **Node.js**

Create a wallet client using CDP's Wallet API or [viem](https://viem.sh/):&#x20;

{% tabs %}
{% tab title="Viem" %}
First, install the required packages:

```bash
npm install viem
```

Then, instantiate the wallet account:

```typescript
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { baseSepolia } from "viem/chains";

// Create a wallet client (using your private key)
const account = privateKeyToAccount("0xYourPrivateKey"); // we recommend using an environment variable for this
```
{% endtab %}

{% tab title="CDP Wallet API" %}
First, create an account at [cdp.coinbase.com](https://cdp.coinbase.com/) and get the following API keys from the portal to store as environment variables:

```
# store in .env or using the command `export <name>="secret-info"`
CDP_API_KEY_ID=your-api-key-id
CDP_API_KEY_SECRET=your-api-key-secret
CDP_WALLET_SECRET=your-wallet-secret
```

Then, install the required packages:

```bash
npm install @coinbase/cdp-sdk dotenv
```

Finally, instantiate the CDP client as suggested by the [Wallet API Quickstart](https://docs.cdp.coinbase.com/wallet-api/v2/introduction/quickstart):

```typescript
import { CdpClient } from "@coinbase/cdp-sdk";
import { createWalletClient, http } from "viem";
import { baseSepolia } from "viem/chains";
import dotenv from "dotenv";

dotenv.config()

const cdp = new CdpClient();
const cdpAccount = await cdp.evm.createAccount();
const account = toAccount(cdpAccount);

```
{% endtab %}
{% endtabs %}

#### **Python**

Create a wallet client using CDP's Wallet API or [eth-account](https://github.com/ethereum/eth-account):

{% tabs %}
{% tab title="eth-account" %}
First, install the required packages:

```
pip install eth_account
```

Then, instantiate the wallet account:

```python
from eth_account import Account

account = Account.from_key("your_private_key") # we recommend using an environment variable fo
```
{% endtab %}

{% tab title="CDP Wallet API" %}
First, create an account at [cdp.coinbase.com](https://cdp.coinbase.com/) and get the following API keys from the portal to store as environment variables:

```
# store in .env or using the command `export <name>="secret-info"`
CDP_API_KEY_ID=your-api-key-id
CDP_API_KEY_SECRET=your-api-key-secret
CDP_WALLET_SECRET=your-wallet-secret
```

Then, install the required packages:

```
pip install cdp dotenv
```

Finally, instantiate the CDP client as suggested by the [Wallet API Quickstart](https://docs.cdp.coinbase.com/wallet-api/v2/introduction/quickstart):

```python
import asyncio
from cdp import CdpClient
from dotenv import load_dotenv

load_dotenv()

cdp = CdpClient()
account = await cdp.evm.create_account()
```
{% endtab %}
{% endtabs %}

### 3. Make Paid Requests Automatically

#### Node.js

You can use either `x402-fetch` or `x402-axios` to automatically handle 402 Payment Required responses and complete payment flows.

{% tabs %}
{% tab title="Fetch" %}
**x402-fetch** extends the native `fetch` API to handle 402 responses and payment headers for you. [Full example here](https://github.com/coinbase/x402/tree/main/examples/typescript/clients/fetch)

```typescript
import { wrapFetchWithPayment, decodeXPaymentResponse } from "x402-fetch";
// other imports...

// wallet creation logic...

const fetchWithPayment = wrapFetchWithPayment(fetch, account);

fetchWithPayment(url, { //url should be something like https://api.example.com/paid-endpoint
  method: "GET",
})
  .then(async response => {
    const body = await response.json();
    console.log(body);

    const paymentResponse = decodeXPaymentResponse(response.headers.get("x-payment-response")!);
    console.log(paymentResponse);
  })
  .catch(error => {
    console.error(error.response?.data?.error);
  });
```
{% endtab %}

{% tab title="Axios" %}
**x402-axios** adds a payment interceptor to Axios, so your requests are retried with payment headers automatically. [Full example here](https://github.com/coinbase/x402/tree/main/examples/typescript/clients/axios)

```typescript
import { withPaymentInterceptor, decodeXPaymentResponse } from "x402-axios";
import axios from "axios";
// other imports...

// wallet creation logic...

// Create an Axios instance with payment handling
const api = withPaymentInterceptor(
  axios.create({
    baseURL, // e.g. https://api.example.com
  }),
  account,
);

api
  .get(endpointPath) // e.g. /paid-endpoint
  .then(response => {
    console.log(response.data);

    const paymentResponse = decodeXPaymentResponse(response.headers["x-payment-response"]);
    console.log(paymentResponse);
  })
  .catch(error => {
    console.error(error.response?.data?.error);
  });
```
{% endtab %}
{% endtabs %}

#### Python

You can use either `httpx` or `Requests`  to automatically handle 402 Payment Required responses and complete payment flows.

* **Requests** is a well-established library for **synchronous** HTTP requests. It is simple and ideal for straightforward, sequential workflows.
* **HTTPX** is a modern library that supports both **synchronous** and **asynchronous** (async) HTTP requests. Use HTTPX if you need high concurrency, advanced features like HTTP/2, or want to leverage Python’s async capabilities

Both support a **simple** and **extensible** approach. The simple returns a pre-configured client that handles payments automatically, while the extensible lets you use an existing session/client. The simple is covered here, while the extensible is in the README of the full examples linked below.

{% tabs %}
{% tab title="HTTPX" %}
[Full example here](https://github.com/coinbase/x402/tree/main/examples/python/clients/httpx)

<pre class="language-python"><code class="lang-python">from x402.clients.httpx import x402HttpxClient
# Other imports...

# Wallet creation logic ...
<strong>
</strong><strong># Create client and make request
</strong>async with x402HttpxClient(account=account, base_url="https://api.example.com") as client:
    response = await client.get("/protected-endpoint")
    print(await response.aread())
</code></pre>
{% endtab %}

{% tab title="Requests" %}
[Full example here](https://github.com/coinbase/x402/tree/main/examples/python/clients/requests)

<pre class="language-python"><code class="lang-python">from x402.clients.requests import x402_requests
# Other imports...

# Wallet creation logic ...
<strong>
</strong><strong># Create session and make request
</strong>session = x402_requests(account)
response = session.get("https://api.example.com/protected-endpoint")
print(response.content)
</code></pre>
{% endtab %}
{% endtabs %}

### 4. Error Handling

Clients will throw errors if:

* The request configuration is missing
* A payment has already been attempted for the request
* There is an error creating the payment header

### Summary

* Install an x402 client package
* Create a wallet client
* Use the provided wrapper/interceptor to make paid API requests
* Payment flows are handled automatically for you

***

**References:**

* [x402-fetch npm docs](https://www.npmjs.com/package/x402-fetch)
* [x402-axios npm docs](https://www.npmjs.com/package/x402-axios)
* [x402 PyPi page](https://pypi.org/project/x402/)

For questions or support, join our [Discord](https://discord.gg/invite/cdp).
