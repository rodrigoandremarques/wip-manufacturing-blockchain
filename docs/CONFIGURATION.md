# Configuration and Deployment Guide

## Environment Variables

All configuration is done through environment variables. Set them in your hosting platform or in a local .env file.

| Variable                | Required  | Default                        | Description                                                              |
|-------------------------|-----------|--------------------------------|--------------------------------------------------------------------------|
| PORT                    | No        | 3000                           | HTTP port the server listens on                                          |
| NODE_ENV                | No        | --                             | Set to 'production' for stricter settings                                |
| DATA_DIR                | No        | ./data                         | Directory for JSON persistence files                                     |
| ADMIN_USER              | Yes       | admin                          | Primary admin username                                                   |
| ADMIN_PASSWORD          | Yes       | WIP2026                        | Primary admin password. Change before deploying to any environment.      |
| VISITOR_USER            | No        | demo                           | Demo/public account username                                             |
| VISITOR_PASSWORD        | No        | (not set)                      | Demo account password. Set this variable to enable the demo account.     |
| ALLOWED_ORIGINS         | No        | localhost + project domain     | Comma-separated CORS allowed origins                                     |
| ETHEREUM_ENABLED        | No        | false                          | Set to 'true' to activate real blockchain anchoring                      |
| ETHEREUM_NETWORK        | No        | sepolia                        | 'sepolia', 'polygon', or 'mainnet'                                       |
| ETHEREUM_RPC_URL        | No*       | --                             | Alchemy or Infura RPC endpoint                                           |
| ETHEREUM_PRIVATE_KEY    | No*       | --                             | Wallet private key for signing transactions                              |
| CONTRACT_MANUFACTURING  | No*       | --                             | Deployed ManufacturingWIP contract address                               |
| SESSION_SECRET          | No        | (random)                       | Express session secret                                                   |

*Required only when ETHEREUM_ENABLED=true.

---

## Running Locally

### Prerequisites

- Node.js 18 or later
- npm 9 or later

### Setup

    git clone https://github.com/rodrigoandremarques/wip-blockchain.git
    cd wip-blockchain
    npm install --omit=dev
    cp .env.example .env

Edit .env with your local values:

    PORT=3000
    NODE_ENV=development
    ADMIN_USER=admin
    ADMIN_PASSWORD=MyLocalPassword123
    VISITOR_PASSWORD=demo123

### Start

    node server.js

Open http://localhost:3000 in your browser.

---

## Deploying to Render.com

### First Deploy

1. Create a new Web Service in the Render dashboard.
2. Connect your GitHub repository.
3. Set:
   - Runtime: Node
   - Build Command: npm install --omit=dev
   - Start Command: node server.js
4. Under Environment, add at minimum the ADMIN_PASSWORD variable.
5. Click Deploy.

### Updating Environment Variables

1. Go to Dashboard -> your service -> Environment.
2. Click Edit.
3. Add or modify variables.
4. Click Save, rebuild, and deploy. Render redeploys automatically.

### Adding a Custom Domain

1. Go to Settings -> Custom Domains -> Add Custom Domain.
2. Enter your domain (e.g. wip.yourdomain.com).
3. Add a CNAME record in your DNS provider:
   - Name:  wip
   - Value: your-service.onrender.com
4. Render provisions a Let's Encrypt SSL certificate automatically within a few minutes.

### Persistent Data (Paid Feature)

The free tier uses ephemeral storage -- data/*.json files reset on each redeploy. To retain data across deploys:

1. Go to Settings -> Disk and add a disk mounted at /app/data.
2. Set DATA_DIR=/app/data in Environment Variables.

---

## Creating Users After Deploy

### Create an Operator Account

    curl -X POST https://your-domain/api/admin/users 
      -u 'admin:YOUR_ADMIN_PASSWORD' 
      -H 'Content-Type: application/json' 
      -d '{"username": "operator1", "password": "OperatorPass2026", "role": "operator"}'

### Enable the Demo Account

Set the VISITOR_PASSWORD environment variable in Render. After redeploy, the account defined by VISITOR_USER (default: demo) becomes active with that password.

### Change the Admin Password

    curl -X PUT https://your-domain/api/admin/users/admin/password 
      -u 'admin:CURRENT_PASSWORD' 
      -H 'Content-Type: application/json' 
      -d '{"newPassword": "NewSecurePassword2026"}'

---

## Ethereum Integration (Optional)

To anchor the local blockchain to a public network:

1. Create a free Alchemy account and a Sepolia app to get an RPC URL.
2. Create an Ethereum wallet and fund it with Sepolia ETH from a faucet.
3. Deploy the ManufacturingWIP.sol contract from the contracts/ directory.
4. Set the following environment variables:

       ETHEREUM_ENABLED=true
       ETHEREUM_NETWORK=sepolia
       ETHEREUM_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY
       ETHEREUM_PRIVATE_KEY=0xYOUR_PRIVATE_KEY
       CONTRACT_MANUFACTURING=0xDEPLOYED_CONTRACT_ADDRESS

Each local block addition will also submit a transaction to the smart contract with the block hash.

---

## Security Checklist

Before deploying to any shared or production environment:

- [ ] Change ADMIN_PASSWORD from the default value WIP2026
- [ ] Use a strong password: at least 12 characters, mixed case, numbers, and symbols
- [ ] Set NODE_ENV=production
- [ ] Restrict ALLOWED_ORIGINS to your production domain only
- [ ] Never commit .env files to version control
- [ ] Keep ETHEREUM_PRIVATE_KEY out of version control and logs
- [ ] Rotate SESSION_SECRET periodically

---

## Free Tier Limitations (Render)

| Limitation          | Details                                                     |
|---------------------|-------------------------------------------------------------|
| Sleep on inactivity | Service sleeps after 15 minutes with no incoming requests   |
| Cold start          | First request after sleep takes 30-50 seconds               |
| Ephemeral storage   | data/*.json files reset on each redeploy                    |
| Persistent disk     | Paid feature (Render Disk)                                  |
| CPU                 | 0.1 shared CPU -- adequate for development and demos        |
| Memory              | 512 MB                                                      |

For production use, upgrade to a paid Render instance and add a persistent disk.
