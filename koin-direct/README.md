# Koin Direct

A mobile app that lets casino players fund and cash out slot machines from their phone over Bluetooth, QR, or player card, without a kiosk or paper tickets.

**Role:** Senior frontend engineer, payment integration lead  
**Client:** Koin Payments  
**Period:** 2024 – 2025  
**Source:** Private (client work)

<p align="center">
  <img src="./fund-machine.png" alt="Fund slot machine screen" width="300"/>
</p>

## Overview

Moving money onto a slot machine normally means cash or a ticket from a kiosk. Koin Direct lets a player pair their phone with a machine, fund it from their Koin Pay balance, and later withdraw back to Koin Pay or print a cashier ticket. The goal was to cut kiosk trips and make casino funding faster and easier to track.

## What I built

- The Bluetooth (BLE) pairing flow in React Native for connecting to game terminals
- Koin Pay API integration for deposits, withdrawals, and ticket printing
- The funding, balance, and disconnect screens
- Ardent Connect integration for real-time game sessions and transaction handoffs
- Receipt, player card management, and transaction history screens
- Transaction integrity checks and user feedback across the funding and cash-out flows

## Technical highlights

### Three ways to connect

A player can connect to a machine by Bluetooth, by scanning a QR code, or with a player card. Cards for each connection type are stored in the app, and the most recent one is surfaced first.

### Session handoff

Once paired, the real-time game session and each transaction handoff run through the Ardent Connect integration.

### Two ways out

Disconnecting offers a withdrawal back to Koin Pay or a cash-out printed as a cashier ticket. Both end on a receipt with the amount, timestamp, transaction ID, source, and destination.

## Architecture

A simplified view based on the integrations described on this page.

```mermaid
flowchart LR
  subgraph Phone
    APP["Koin Direct app<br/>React Native"]
  end

  subgraph Casino["Casino floor"]
    SLOT["Slot machine"]
    TICKET["Cashier ticket"]
  end

  subgraph Services
    ARDENT["Ardent Connect<br/>game sessions"]
    KOINPAY["Koin Pay API<br/>balance, deposits, withdrawals"]
  end

  APP <-->|"BLE, QR, or player card"| SLOT
  APP --> ARDENT --> SLOT
  APP --> KOINPAY
  KOINPAY -->|cash-out| TICKET
```

## Stack

| Layer | Technology |
|---|---|
| Mobile | React Native, TypeScript, Tailwind |
| Connectivity | Bluetooth Low Energy, QR codes, player cards |
| Integrations | Koin Pay API, Ardent Connect, REST |
| Backend and infrastructure | Node.js, AWS, Supabase |

## Results

- Transfers to a machine complete in under 5 seconds over BLE
- Funding conversion rose 21% after the streamlined flow shipped
- Less dependence on casino kiosks
- Used in live casino demos in Las Vegas and accepted by multiple resort partners

## Screenshots

### Connecting to a machine

<p align="center">
  <img src="./bluetooth-pairing.png" alt="Bluetooth pairing" width="220"/>
  <img src="./connecting.png" alt="Connecting animation" width="220"/>
  <img src="./player-cards.png" alt="Player cards" width="220"/>
</p>

Hold the phone near the machine to pair over Bluetooth, a slot-reel animation while the session connects, and stored player cards for card, Bluetooth, and QR connections.

### Funding and cashing out

<p align="center">
  <img src="./fund-machine.png" alt="Fund slot machine" width="220"/>
  <img src="./fund-machine-connected.png" alt="Fund slot machine while connected" width="220"/>
  <img src="./disconnect.png" alt="Disconnect options" width="220"/>
</p>

Preset and custom funding amounts against the Koin Pay balance, the connected-machine state, and the disconnect sheet with withdrawal or cashier-ticket cash-out.

### Wallet

<p align="center">
  <img src="./home-balance.png" alt="Koin balance and transactions" width="220"/>
  <img src="./receipt.png" alt="Transaction receipt" width="220"/>
</p>

The Koin balance with recent transactions, and a receipt for a transfer to a slot game.
