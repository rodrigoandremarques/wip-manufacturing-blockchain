# Configuration & Deployment Guide

## Environment Variables

All configuration is via environment variables. Set them in your hosting platform (Render ? Environment) or in a local `.env` file.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3000` | HTTP port the server listens on |
| `NODE_ENV` | No | Ñ | `production` enables stricter settings |
| `DATA_DIR` | No | `./data` | Directory for JSON persistence files |
| `ADMIN_USER` | Yes | `admin` | Primary admin username |
| `ADMIN_PASSWORD` | **Yes** | `WIP2026` | Primary admin password. **Change before deploying.** |
| `VISITOR_USER` | No | `demo` | Demo/public account username |
| `VISITOR_PASSWORD` | No | *(not set)* | Demo account password. Set to enable the demo account. |
| `ALLOWED_ORIGINS` | No | localhost + rodrigoandremarques.com | Comma-separated CORS allowed origins |
| `ETHEREUM_ENABLED` | No | `false` | Set `true` to activate real blockchain anchoring |
| `ETHEREUM_NETWORK` | No | `sepolia` | `sepolia` &#124; `polygon` &#124; `mainnet` |
| `ETHEREUM_RPC_URL` | No* | Ñ | Alchemy or Infura RPC endpoint |
| `ETHEREUM_PRIVATE_KEY` | No* | Ñ | Wallet private key for signing transactions |
| `CONTRACT_MANUFACTURING` | No* | Ñ | Deployed ManufacturingWIP contract address |
| `SESSION_SECRET` | No | *(random)* | Express session secret |

*Required only when `ETHEREUM_ENABLED=true`.

---

## Running Locally

### Prerequisites

- Node.js 18+ (`node --version`)
- npm 9+

### Setup

```bash
# Clone the private repo
git clone https://github.com/rodrigoandremarques/wip-blockchain.git
cd wip-blockchain

# Install dependencies (skip dev deps for production)
npm install --omit=dev

# Create a local config file
cp .env.example .env
```

Edit `.env`:

```env
PORT=3000
NODE_ENV=development
ADMIN_USER=admin
ADMIN_PASSWORD=MyLocalPassword123
VISITOR_PASSWORD=demo123
```

### Start

```bash
node server.js
```

Open `http://localhost:3000` in your browser.

---

## Deploying to Render.com

### First Deploy

1. Create a **New Web Service** in Render
2. Connect your GitHub repository (private repo)
3. Set:
   - **Runtime:** Node
   - **Build Command:** `npm install --omit=dev`
   - **Start Command:** `node server.js`
4. Under **Environment**, add the required variables (at minimum `ADMIN_PASSWORD`)
5. Click **Deploy**

### Updating Environment Variables

1. Go to **Dashboard ? your service ? Environment**
2. Click **Edit**
3. Add or update variables
4. Click **Save, rebuild, and deploy** Ñ Render will redeploy automatically

### Custom Domain

1. Go to **Settings ? Custom Domains**
2. Click **Add Custom Domain**
3. Enter your domain (e.g., `wip.yourdomain.com`)
4. Add a CNAME record in your DNS provider:
   - **Name:** `wip`
   - **Value:** `your-service.onrender.com`
5. Render provisions a Let's Encrypt SSL certificate automatically (2Ð5 minutes)

### Persistent Data (Optional Ñ Paid)

The free tier uses ephemeral storage Ñ all `data/*.json` files reset on each redeploy. To persist data across deploys:

1. Go to **Settings ? Disk**
2. Add a Disk mounted at `/app/data` (or your `DATA_DIR`)
3. Set `DATA_DIR=/app/data` in Environment Variables
4. Data survives restarts and redeploys

---

## User Management Post-Deploy

### Create an Operator Account

```bash
curl -X POST https://your-domain/api/admin/users \
  -u 'admin:YOUR_ADMIN_PASSWORD' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "operator1",
    "password": "OperatorPass2026!",
    "role": "operator"
  }'
```

### Enable the Demo/Visitor Account

Set the `VISITOR_PASSWORD` environment variable in Render. After redeploy, the `demo` user (or whatever `VISITOR_USER` is set to) will be accessible with that password.

### Change the Admin Password

```bash
curl -X PUT https://your-domain/api/admin/users/admin/password \
  -u 'admin:YOUR_CURRENT_PASSWORD' \
  -H 'Content-Type: application/json' \
  -d '{"newPassword": "NewSecurePassword2026!"}'
```

---

## Ethereum Integration (Optional)

To anchor the local blockchain to Sepolia testnet:

1. Get a free Alchemy account at [alchemy.com](https://alchemy.com) and create a Sepolia app
2. Create an Ethereum wallet (MetaMask or cast)
3. Get Sepolia ETH from a faucet
4. Deploy the `ManufacturingWIP.sol` contract (in the `contracts/` directory of the private repo)
5. Set environment variables:

```env
ETHEREUM_ENABLED=true
ETHEREUM_NETWORK=sepolia
ETHEREUM_RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY
ETHEREUM_PRIVATE_KEY=0xYOUR_PRIVATE_KEY
CONTRACT_MANUFACTURING=0xDEPLOYED_CONTRACT_ADDRESS
```

Each local block addition will then also submit a transaction to the smart contract with the block hash.

---

## Security Checklist

Before going to production:

- [ ] Change `ADMIN_PASSWORD` from the default `WIP2026`
- [ ] Use a strong password (12+ characters, mixed case, numbers, symbols)
- [ ] Set `NODE_ENV=production`
- [ ] Set `ALLOWED_ORIGINS` to your production domain only
- [ ] Never commit `.env` files to git
- [ ] Keep `ETHEREUM_PRIVATE_KEY` out of version control
- [ ] Enable HTTPS (automatic on Render with custom domain)
- [ ] Rotate `SESSION_SECRET` periodically

---

## Free Tier Limitations (Render)

| Limitation | Details |
|-----------|---------|
| Sleep after inactivity | Service sleeps after 15 min with no requests |
| Cold start | First request after sleep takes 30Ð50 seconds |
| Ephemeral storage | `data/*.json` files reset on each redeploy |
| No persistent disk | Paid feature (Render Disk) |
| Shared CPU | 0.1 CPU Ñ adequate for demo/development |
| Memory | 512 MB |

For production use, upgrade to a paid Render instance or use a persistent storage solution.
