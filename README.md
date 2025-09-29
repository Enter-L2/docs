# Enter L2 Documentation

Comprehensive documentation for Enter L2 - the ZK Rollup-based stablecoin payment network with merchant-paid fees and consumer-friendly wallets.

## 📚 Documentation Structure

This repository contains all public documentation for Enter L2, organized into the following sections:

- **[Whitepaper](whitepaper/)** - Technical specification and architecture
- **[User Guide](user-guide/)** - End-user documentation for wallets and payments
- **[Developer Guide](developer-guide/)** - Integration guides and API documentation
- **[Node Operators](node-operators/)** - Running and maintaining Enter L2 nodes
- **[Smart Contracts](smart-contracts/)** - Contract documentation and ABIs
- **[SDK Reference](sdk/)** - Multi-language SDK documentation
- **[API Reference](api/)** - REST API and RPC documentation
- **[Tutorials](tutorials/)** - Step-by-step guides and examples
- **[FAQ](faq/)** - Frequently asked questions

## 🚀 Quick Start

### For Users
- [Getting Started](user-guide/getting-started.md) - Create your first wallet
- [Making Payments](user-guide/payments.md) - Send and receive payments
- [Bridge Guide](user-guide/bridge.md) - Move funds between L1 and L2

### For Developers
- [Integration Guide](developer-guide/integration.md) - Integrate Enter L2 into your app
- [SDK Quickstart](sdk/quickstart.md) - Start building with our SDKs
- [API Reference](api/reference.md) - Complete API documentation

### For Node Operators
- [Node Setup](node-operators/setup.md) - Run an Enter L2 node
- [Configuration](node-operators/configuration.md) - Node configuration options
- [Monitoring](node-operators/monitoring.md) - Monitor node health

## 🏗️ Architecture Overview

Enter L2 is a ZK Rollup-based Layer 2 solution designed specifically for stablecoin payments with the following key features:

### **Role-Based Wallets**
- **Consumer Wallets**: Receive-only wallets that can only accept payments from merchant wallets
- **Merchant Wallets**: Full-featured wallets with payment capabilities and fee management

### **Merchant-Paid Fees**
- Merchants automatically pay transaction fees for consumer transactions
- Enables gasless payments for end users
- Supports USDC/USDT fee payments with automatic conversion

### **ZK Proof System**
- Halo2-based circuits for privacy and scalability
- Batch proving for efficient transaction processing
- Independent verification by public nodes

### **Naming System**
- ENS-like name registration and resolution
- Phone number verification and linking
- Primary name management for addresses

### **Staking & Rewards**
- Token staking with lock period multipliers
- Profit-sharing from network transaction fees
- Automated reward distribution

## 📖 Key Concepts

### Wallet Types
```
Consumer Wallet:
├── Receive payments from merchants
├── Cannot initiate transactions
├── No gas fees required
└── Perfect for end users

Merchant Wallet:
├── Full transaction capabilities
├── Pays fees for consumer transactions
├── Whitelist management
├── Daily spending limits
└── Multi-operator support
```

### Fee Structure
```
Transaction Fees:
├── Paid by merchant wallets
├── Automatic fee calculation
├── USDC/USDT support
├── Price oracle integration
└── Gas optimization
```

### Network Architecture
```
Enter L2 Network:
├── Sequencer (Private)
│   ├── Transaction ordering
│   ├── Batch creation
│   └── State management
├── Prover (Private)
│   ├── ZK proof generation
│   ├── Circuit optimization
│   └── Verification keys
├── Public Nodes
│   ├── Independent verification
│   ├── Data availability
│   └── RPC services
└── Smart Contracts (L1)
    ├── State management
    ├── Bridge operations
    └── Proof verification
```

## 🔧 Development Tools

### SDKs
- **TypeScript/JavaScript**: Full-featured SDK for web and Node.js
- **Go**: High-performance SDK for backend services  
- **Python**: Comprehensive SDK with async support
- **Rust**: Coming soon
- **Java**: Coming soon

### APIs
- **JSON-RPC**: Ethereum-compatible RPC interface
- **REST API**: RESTful API for aggregated data
- **WebSocket**: Real-time event subscriptions
- **GraphQL**: Flexible data querying (coming soon)

### Tools
- **Explorer**: Blockchain explorer at [explorer.enterl2.com](https://explorer.enterl2.com)
- **Faucet**: Testnet token faucet at [faucet.enterl2.com](https://faucet.enterl2.com)
- **Bridge**: L1-L2 bridge interface at [bridge.enterl2.com](https://bridge.enterl2.com)
- **Staking**: Staking dashboard at [stake.enterl2.com](https://stake.enterl2.com)

## 🌐 Network Information

### Mainnet
- **Chain ID**: 42161
- **RPC URL**: https://rpc.enterl2.com
- **Explorer**: https://explorer.enterl2.com
- **Bridge**: https://bridge.enterl2.com

### Testnet
- **Chain ID**: 421613
- **RPC URL**: https://testnet-rpc.enterl2.com
- **Explorer**: https://testnet-explorer.enterl2.com
- **Bridge**: https://testnet-bridge.enterl2.com
- **Faucet**: https://faucet.enterl2.com

## 📋 Documentation Standards

This documentation follows these standards:

- **Markdown**: All documentation written in GitHub Flavored Markdown
- **Structure**: Consistent heading hierarchy and navigation
- **Code Examples**: Working code samples in multiple languages
- **Diagrams**: Mermaid diagrams for architecture and flows
- **Versioning**: Semantic versioning for API changes
- **Accessibility**: Screen reader friendly formatting

## 🤝 Contributing

We welcome contributions to improve our documentation:

1. **Fork** the repository
2. **Create** a feature branch
3. **Make** your changes
4. **Test** all code examples
5. **Submit** a pull request

### Writing Guidelines

- Use clear, concise language
- Include working code examples
- Add diagrams for complex concepts
- Test all instructions thoroughly
- Follow the existing structure

### Review Process

All documentation changes go through:
1. Technical review for accuracy
2. Editorial review for clarity
3. Community feedback period
4. Final approval and merge

## 📞 Support

### Community
- **Discord**: [discord.gg/enterl2](https://discord.gg/enterl2)
- **Telegram**: [t.me/enterl2](https://t.me/enterl2)
- **Twitter**: [@EnterL2](https://twitter.com/EnterL2)
- **Reddit**: [r/EnterL2](https://reddit.com/r/EnterL2)

### Developer Support
- **GitHub Issues**: [github.com/enter-l2/docs/issues](https://github.com/enter-l2/docs/issues)
- **Developer Forum**: [forum.enterl2.com](https://forum.enterl2.com)
- **Email**: dev@enterl2.com

### Business Inquiries
- **Partnerships**: partnerships@enterl2.com
- **Enterprise**: enterprise@enterl2.com
- **Press**: press@enterl2.com

## 📄 License

This documentation is licensed under [Creative Commons Attribution 4.0 International License](LICENSE).

Code examples are licensed under [MIT License](LICENSE-CODE).

## 🔄 Updates

Documentation is continuously updated. Major changes are announced via:

- **GitHub Releases**: Version tags and changelogs
- **Discord Announcements**: Real-time updates
- **Newsletter**: Monthly development updates
- **Blog**: Technical deep-dives and tutorials

---

**Last Updated**: December 2024  
**Version**: 1.0.0  
**Next Review**: January 2025
