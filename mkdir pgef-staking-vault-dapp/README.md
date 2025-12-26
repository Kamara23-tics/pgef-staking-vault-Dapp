markdown
# 🏦 PGEF Staking Vault DApp

A decentralized application for staking ADA on Cardano and earning 5% APY.

## 🚀 Live Demo

**[Try it now →](https://Kamara23-tics.github.io/pgef-staking-vault/)**


## ✨ Features

- 💰 Stake ADA and earn 5% APY
- 🎁 Claim rewards anytime
- 🔒 Secure Plutus V2 smart contracts
- 🎨 User-friendly interface
- 💼 Lace wallet integration

## 🧪 Testing Instructions

This DApp runs on **Cardano Preprod Testnet**.

### Prerequisites

1. **Install Lace Wallet**
   - Download: https://www.lace.io/
   - Create a wallet or import existing

2. **Switch to Preprod Network**
   - Open Lace → Settings → Network
   - Select "Preprod"

3. **Get Test ADA**
   - Visit: https://docs.cardano.org/cardano-testnet/tools/faucet
   - Enter your Preprod address
   - Receive 10,000 test ADA

### How to Use

1. Open the DApp
2. Click "Connect Lace Wallet"
3. Approve the connection
4. Enter stake amount (e.g., 50 ADA)
5. Click "Stake"
6. Wait for confirmation
7. View your stakes anytime
8. Click "Claim" to withdraw

## 🛠️ Technology Stack

- **Frontend:** HTML5, CSS3, JavaScript (ES6+)
- **Blockchain Library:** Lucid Cardano v0.10.11
- **Smart Contract:** Plutus V2
- **API Provider:** Blockfrost
- **Wallet:** Lace

## 📜 Smart Contract Details

- **Type:** Plutus V2 Validator
- **Network:** Preprod Testnet
- **APY:** 5% (calculated per second)
- **Script Address:** `addr_test1w...` (shown in DApp)

## 🔧 Local Development

### Setup

1. Clone this repository:
   bash
   git clone https://github.com/Kamara23-tics/pgef-staking-vault.git
   cd pgef-staking-vault
   

2. Create `config.js` from template:
   bash
   cp config.template.js config.js
   

3. Add your Blockfrost API key to `config.js`

4. Open with Live Server (VS Code extension)

### Configuration

Get your free Blockfrost API key:
1. Visit https://blockfrost.io
2. Sign up
3. Create a Preprod project
4. Copy the API key
5. Paste into `config.js`

## ⚠️ Important Notes

- **Testnet Only:** This deployment is on Preprod testnet
- **Not Audited:** Smart contract has not been professionally audited
- **Educational Purpose:** For learning and testing only
- **No Real Value:** Testnet ADA has no monetary value

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details

## 🆘 Support

Having issues? 

- Check [Issues](https://github.com/Kamara23-tics/pgef-staking-vault/issues)
- Open a new issue
- Contact: graceonah79@gmail.com

## 🙏 Acknowledgments

- Cardano Foundation
- Lucid Cardano team
- Blockfrost team
- Cardano community

---

**Built with ❤️ for the Cardano ecosystem**


---
