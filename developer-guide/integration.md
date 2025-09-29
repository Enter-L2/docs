# Integration Guide

This guide will help you integrate Enter L2 into your application, whether you're building a payment processor, e-commerce platform, or financial application.

## Quick Start

### 1. Choose Your Integration Method

**SDK Integration (Recommended)**
- Use our official SDKs for TypeScript, Go, or Python
- Handles wallet management, transactions, and events
- Built-in error handling and retry logic

**Direct API Integration**
- REST API for backend services
- JSON-RPC for blockchain interactions
- WebSocket for real-time events

**Smart Contract Integration**
- Deploy your own contracts on Enter L2
- Interact with existing wallet contracts
- Build custom payment flows

### 2. Set Up Your Environment

```bash
# Install the SDK
npm install @enter-l2/sdk
# or
go get github.com/enter-l2/sdk-go
# or
pip install enterl2-sdk
```

### 3. Initialize the Client

```typescript
import { EnterL2Client } from '@enter-l2/sdk';

const client = new EnterL2Client({
  rpcUrl: 'https://rpc.enterl2.com',
  l1RpcUrl: 'https://mainnet.infura.io/v3/YOUR_KEY',
  apiUrl: 'https://api.enterl2.com',
});

// Connect with your private key
await client.connect(wallet);
```

## Integration Patterns

### E-commerce Platform

Perfect for online stores that want to accept crypto payments:

```typescript
class PaymentProcessor {
  private client: EnterL2Client;
  
  constructor() {
    this.client = new EnterL2Client({
      rpcUrl: process.env.ENTER_L2_RPC_URL,
    });
  }
  
  async processPayment(order: Order): Promise<PaymentResult> {
    // Create merchant wallet if needed
    const merchantWallet = await this.ensureMerchantWallet();
    
    // Generate payment request
    const paymentRequest = this.client.payment.createPaymentRequest(
      order.customerAddress,
      order.amount.toString(),
      order.token,
      `Order #${order.id}`
    );
    
    // Monitor for payment
    return this.waitForPayment(paymentRequest);
  }
  
  private async waitForPayment(request: string): Promise<PaymentResult> {
    return new Promise((resolve) => {
      this.client.on('paymentReceived', (payment) => {
        if (payment.description?.includes(request)) {
          resolve({
            success: true,
            txHash: payment.hash,
            amount: payment.amount,
          });
        }
      });
    });
  }
}
```

### Payment Gateway

For businesses that process payments for multiple merchants:

```typescript
class PaymentGateway {
  private clients: Map<string, EnterL2Client> = new Map();
  
  async createMerchantAccount(merchantId: string): Promise<string> {
    const client = new EnterL2Client(this.config);
    await client.connect(this.generateMerchantWallet());
    
    // Create merchant wallet with specific settings
    const walletAddress = await client.createMerchantWallet(
      true, // whitelist enabled
      '1000000000000000000000000' // 1M USDC daily limit
    );
    
    this.clients.set(merchantId, client);
    return walletAddress;
  }
  
  async processPayment(
    merchantId: string,
    payment: PaymentRequest
  ): Promise<TransactionResponse> {
    const client = this.clients.get(merchantId);
    if (!client) throw new Error('Merchant not found');
    
    return await client.sendPayment(
      payment.to,
      payment.amount,
      payment.token,
      payment.description
    );
  }
}
```

### DeFi Application

For applications that need to interact with DeFi protocols:

```typescript
class DeFiIntegration {
  private client: EnterL2Client;
  
  async stakingDashboard(userAddress: string) {
    // Get staking positions
    const positions = await this.client.staking.getPositions(userAddress);
    
    // Get available rewards
    const rewards = await this.client.staking.getRewards(userAddress);
    
    // Calculate APY
    const apy = await this.calculateAPY(positions);
    
    return {
      positions,
      rewards,
      apy,
      totalStaked: positions.reduce((sum, p) => sum + p.amount, 0),
    };
  }
  
  async autoCompound(userAddress: string) {
    // Claim rewards
    const claimTx = await this.client.claimRewards();
    await this.client.waitForTransaction(claimTx.hash);
    
    // Restake rewards
    const balance = await this.client.getBalance();
    if (balance > 0) {
      return await this.client.stake(balance, 31536000); // 1 year lock
    }
  }
}
```

## Wallet Management

### Creating Wallets

```typescript
// Consumer wallet for end users
const consumerWallet = await client.createConsumerWallet();

// Merchant wallet for businesses
const merchantWallet = await client.createMerchantWallet(
  true, // whitelist enabled
  '10000000000000000000000' // 10K USDC daily limit
);

// Get wallet information
const walletInfo = await client.wallet.getWalletInfo(merchantWallet);
console.log('Wallet type:', walletInfo.type);
console.log('Daily limit:', walletInfo.dailyLimit);
```

### Managing Operators

```typescript
// Add operator to merchant wallet
await client.wallet.addOperator(merchantWallet, operatorAddress);

// Remove operator
await client.wallet.removeOperator(merchantWallet, operatorAddress);

// Update daily limit
await client.wallet.updateDailyLimit(merchantWallet, newLimit);
```

## Payment Processing

### Basic Payments

```typescript
// Send payment
const tx = await client.sendPayment(
  recipientAddress,
  '1000000', // 1 USDC (6 decimals)
  usdcTokenAddress,
  'Payment for services'
);

// Wait for confirmation
const receipt = await client.waitForTransaction(tx.hash);
console.log('Payment confirmed:', receipt.status);
```

### Batch Payments

```typescript
// Process multiple payments efficiently
const payments = [
  { to: 'alice', amount: '100000000', token: usdcAddress },
  { to: 'bob', amount: '200000000', token: usdcAddress },
  { to: 'charlie', amount: '300000000', token: usdcAddress },
];

const results = await Promise.allSettled(
  payments.map(payment => client.sendPayment(
    payment.to,
    payment.amount,
    payment.token
  ))
);

console.log('Successful payments:', results.filter(r => r.status === 'fulfilled').length);
```

### Payment Monitoring

```typescript
// Monitor all payments for a merchant
client.payment.subscribeToPayments(merchantAddress);

client.on('paymentSent', (payment) => {
  console.log('Payment sent:', payment);
  // Update database, send notifications, etc.
});

client.on('paymentReceived', (payment) => {
  console.log('Payment received:', payment);
  // Process order, update inventory, etc.
});
```

## Bridge Integration

### Deposits from L1

```typescript
// Monitor for L1 deposits
client.bridge.subscribeToDeposits(userAddress);

client.on('depositInitiated', async (deposit) => {
  console.log('Deposit initiated on L1:', deposit.l1TxHash);
  
  // Wait for L1 confirmations
  await client.bridge.waitForDepositConfirmation(deposit.l1TxHash);
  
  // Process on L2
  console.log('Deposit completed on L2:', deposit.l2TxHash);
});

// Initiate deposit
const depositTx = await client.deposit(
  usdcAddress,
  '1000000000', // 1000 USDC
  l2RecipientAddress
);
```

### Withdrawals to L1

```typescript
// Initiate withdrawal
const withdrawalTx = await client.withdraw(
  usdcAddress,
  '500000000', // 500 USDC
  l1RecipientAddress
);

// Monitor withdrawal status
const status = await client.bridge.getWithdrawalStatus(withdrawalTx.hash);
console.log('Withdrawal status:', status);

// Wait for finalization (can take 7 days)
await client.bridge.waitForWithdrawalFinalization(withdrawalTx.hash);
```

## Name Service Integration

### Name Registration

```typescript
// Register name for user
const nameTx = await client.registerName('alice');
await client.waitForTransaction(nameTx.hash);

// Set as primary name
await client.naming.setPrimaryName('alice');

// Register phone number
await client.naming.registerPhone('+1234567890');

// Verify phone
await client.naming.verifyPhone('+1234567890', verificationCode);
```

### Name Resolution

```typescript
// Resolve name to address
const address = await client.naming.resolveName('alice');

// Reverse resolve address to name
const name = await client.naming.resolveAddress(address);

// Get name information
const nameInfo = await client.naming.getNameInfo('alice');
console.log('Owner:', nameInfo.owner);
console.log('Phone verified:', nameInfo.phoneVerified);
```

## Error Handling

### Common Error Patterns

```typescript
import { EnterL2Error, TransactionError, NetworkError } from '@enter-l2/sdk';

try {
  await client.sendPayment(to, amount, token);
} catch (error) {
  if (error instanceof TransactionError) {
    console.error('Transaction failed:', error.txHash);
    // Handle transaction-specific errors
  } else if (error instanceof NetworkError) {
    console.error('Network error:', error.statusCode);
    // Handle network connectivity issues
  } else if (error instanceof EnterL2Error) {
    console.error('Enter L2 error:', error.code);
    // Handle protocol-specific errors
  } else {
    console.error('Unexpected error:', error);
  }
}
```

### Retry Logic

```typescript
async function sendPaymentWithRetry(
  to: string,
  amount: string,
  token: string,
  maxRetries: number = 3
): Promise<TransactionResponse> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await client.sendPayment(to, amount, token);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Testing

### Testnet Setup

```typescript
const testnetClient = new EnterL2Client({
  rpcUrl: 'https://testnet-rpc.enterl2.com',
  l1RpcUrl: 'https://goerli.infura.io/v3/YOUR_KEY',
  chainId: 421613,
});

// Get test tokens from faucet
await fetch('https://faucet.enterl2.com/api/faucet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ address: userAddress }),
});
```

### Unit Testing

```typescript
import { jest } from '@jest/globals';

describe('Payment Processing', () => {
  let client: EnterL2Client;
  
  beforeEach(() => {
    client = new EnterL2Client(testConfig);
  });
  
  test('should send payment successfully', async () => {
    const mockTx = { hash: '0x123...', status: 'confirmed' };
    jest.spyOn(client, 'sendPayment').mockResolvedValue(mockTx);
    
    const result = await client.sendPayment('alice', '1000000', usdcAddress);
    expect(result.hash).toBe('0x123...');
  });
});
```

## Performance Optimization

### Connection Pooling

```typescript
class EnterL2Pool {
  private clients: EnterL2Client[] = [];
  private currentIndex = 0;
  
  constructor(poolSize: number, config: EnterL2Config) {
    for (let i = 0; i < poolSize; i++) {
      this.clients.push(new EnterL2Client(config));
    }
  }
  
  getClient(): EnterL2Client {
    const client = this.clients[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.clients.length;
    return client;
  }
}
```

### Caching

```typescript
class CachedEnterL2Client {
  private cache = new Map<string, any>();
  private client: EnterL2Client;
  
  async getBalance(address: string, token?: string): Promise<string> {
    const key = `balance:${address}:${token || 'ETH'}`;
    
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }
    
    const balance = await this.client.getBalance(token);
    this.cache.set(key, balance);
    
    // Cache for 30 seconds
    setTimeout(() => this.cache.delete(key), 30000);
    
    return balance;
  }
}
```

## Security Considerations

### Private Key Management

```typescript
// Use environment variables
const client = new EnterL2Client({
  rpcUrl: process.env.ENTER_L2_RPC_URL,
});

// Connect with encrypted private key
const encryptedKey = process.env.ENCRYPTED_PRIVATE_KEY;
const privateKey = decrypt(encryptedKey, process.env.ENCRYPTION_PASSWORD);
await client.connect(privateKey);
```

### Input Validation

```typescript
function validateAddress(address: string): boolean {
  return /^0x[a-fA-F0-9]{40}$/.test(address);
}

function validateAmount(amount: string): boolean {
  try {
    const parsed = BigInt(amount);
    return parsed > 0;
  } catch {
    return false;
  }
}

// Always validate inputs
if (!validateAddress(recipientAddress)) {
  throw new Error('Invalid recipient address');
}

if (!validateAmount(amount)) {
  throw new Error('Invalid amount');
}
```

## Next Steps

- **[API Reference](../api/reference.md)** - Complete API documentation
- **[SDK Documentation](../sdk/)** - Language-specific SDK guides
- **[Smart Contracts](../smart-contracts/)** - Contract interfaces and ABIs
- **[Examples](../tutorials/)** - Complete integration examples
