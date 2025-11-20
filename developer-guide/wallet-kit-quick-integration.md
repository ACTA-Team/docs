# Stellar Wallet Kit - Quick Integration

Build a wallet management system for Stellar in React/Next.js with secure connection, disconnection, and persistent state. No API key is required.

## Overview

- Wallet Provider: global state and persistence
- Wallet Hook: connection/disconnection business logic
- Wallet Kit Configuration: Stellar integration setup

## Step 1: Install Dependencies

```bash
npm install @creit.tech/stellar-wallets-kit
```

## Step 2: Configure the Stellar Wallet Kit

```ts
import {
  StellarWalletsKit,
  WalletNetwork,
  FREIGHTER_ID,
  AlbedoModule,
  FreighterModule,
} from "@creit.tech/stellar-wallets-kit";

export const kit: StellarWalletsKit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: FREIGHTER_ID,
  modules: [new FreighterModule(), new AlbedoModule()],
});

export async function signTransactionWithKit(unsignedTransaction: string, address: string, networkPassphrase: string): Promise<string> {
  const { signedTxXdr } = await kit.signTransaction(unsignedTransaction, {
    address,
    networkPassphrase,
  });
  return signedTxXdr;
}
```

## Step 3: Create the Wallet Context Provider

```tsx
"use client";
import { createContext, useContext, useState, useEffect, ReactNode } from "react";

type WalletContextType = {
  walletAddress: string | null;
  walletName: string | null;
  setWalletInfo: (address: string, name: string) => void;
  clearWalletInfo: () => void;
};

const WalletContext = createContext<WalletContextType | undefined>(undefined);

export const WalletProvider = ({ children }: { children: ReactNode }) => {
  const [walletAddress, setWalletAddress] = useState<string | null>(null);
  const [walletName, setWalletName] = useState<string | null>(null);

  useEffect(() => {
    const storedAddress = localStorage.getItem("walletAddress");
    const storedName = localStorage.getItem("walletName");
    if (storedAddress) setWalletAddress(storedAddress);
    if (storedName) setWalletName(storedName);
  }, []);

  const setWalletInfo = (address: string, name: string) => {
    setWalletAddress(address);
    setWalletName(name);
    localStorage.setItem("walletAddress", address);
    localStorage.setItem("walletName", name);
  };

  const clearWalletInfo = () => {
    setWalletAddress(null);
    setWalletName(null);
    localStorage.removeItem("walletAddress");
    localStorage.removeItem("walletName");
  };

  return (
    <WalletContext.Provider value={{ walletAddress, walletName, setWalletInfo, clearWalletInfo }}>
      {children}
    </WalletContext.Provider>
  );
};

export const useWalletContext = () => {
  const ctx = useContext(WalletContext);
  if (!ctx) throw new Error("useWalletContext must be used within WalletProvider");
  return ctx;
};
```

## Step 4: Create the Wallet Hook

```ts
import { kit } from "@/config/wallet-kit";
import { useWalletContext } from "@/providers/wallet.provider";
import type { ISupportedWallet } from "@creit.tech/stellar-wallets-kit";

export function useWallet() {
  const { walletAddress, walletName, setWalletInfo, clearWalletInfo } = useWalletContext();

  async function connect() {
    const { address, name } = await kit.openModal();
    setWalletInfo(address, name as ISupportedWallet);
    return { address, name };
  }

  async function disconnect() {
    clearWalletInfo();
  }

  return { walletAddress, walletName, connect, disconnect };
}
```

## Step 5: Use with ACTA SDK Hooks

The hooks in the ACTA SDK expect a signer function of type `(unsignedXdr: string, opts: { networkPassphrase: string }) => Promise<string>` (see `sdk/src/hooks/useCreateVault.ts:1-5`, `sdk/src/hooks/useAuthorizeIssuer.ts:1-5`).

```ts
import { useActaClient, useCreateVault } from "@acta-team/acta-sdk";
import { useWalletContext } from "@/providers/wallet.provider";
import { signTransactionWithKit } from "@/config/wallet-kit";

export function useInitializeVault() {
  const client = useActaClient();
  const { walletAddress } = useWalletContext();
  const { createVault } = useCreateVault();

  async function initialize(ownerDid: string) {
    const { networkPassphrase } = client.getDefaults();
    const signTransaction = async (unsignedXdr: string, opts: { networkPassphrase: string }) => {
      return signTransactionWithKit(unsignedXdr, walletAddress!, opts.networkPassphrase);
    };
    return createVault({ owner: walletAddress!, ownerDid, signTransaction });
  }

  return { initialize };
}
```

- The same signer function can be passed to other on-chain hooks like `authorizeIssuer`.
- Defaults come from `client.getDefaults()` (`sdk/src/client.ts:55-63`).

## Networks

- Use `WalletNetwork.TESTNET` during development.
- Switch to `WalletNetwork.MAINNET` for production.