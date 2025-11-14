# BeeBolt

[![Blockdag](https://blockdag.network/3d.gif)](https://blockdag.network/)
[![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://example.com) <!-- Placeholder for CI badge -->
[![GitHub issues](https://img.shields.io/github/issues/Bee-Bolt/beebolt.svg)](https://github.com/Bee-Bolt/beebolt/issues)
[![GitHub stars](https://img.shields.io/github/stars/Bee-Bolt/beebolt.svg)](https://github.com/Bee-Bolt/beebolt/stargazers)

BeeBolt is a modern, high-throughput cryptocurrency exchange designed to allow users to spend their digital assets as fiat instantly using a **Just-in-Time (JIT) Funded Debit Card** program. This project leverages **Golang (Go)** for its core engines to ensure sub-second latency for payment authorizations, critical for merchant acceptance.

BeeBolt bridges the gap between crypto holdings and real-world spending, focusing on seamless, efficient transactions across EMEA regions (e.g., NGN in Nigeria, EUR in Europe). It's built for speed, security, and scalability, making crypto as usable as everyday money.

## Table of Contents

- [Core Features](#-core-features)
- [Architecture Overview](#️-architecture-overview)
- [Technology Stack](#️-technology-stack)
- [JIT Card Spending Workflow](#️-jit-card-spending-workflow-the-critical-path)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Testing](#testing)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## 🚀 Core Features

- **JIT Card Spending:** Instantly convert held cryptocurrency to fiat at the point-of-sale via real-time authorization and funding calls, enhanced with BlockDAG for near-instant asset transfers.
- **Atomic Ledger:** A robust, double-entry accounting ledger ensuring 100% financial reconciliation between crypto holdings and fiat settlement.
- **High Performance:** Built with Go for unparalleled concurrency and low-latency processing of financial events.
- **Secure Custody:** Segregated hot/cold wallet management for digital assets.
- **Fiat On/Off-Ramps:** Seamless conversion between local currencies (e.g., NGN, EUR, AED) and crypto.
- **Rewards System:** Earn 1% cashback in BEE tokens on every spend.
- **Compliance-Ready:** Built-in KYC/AML hooks and regional regulatory support for EMEA markets.

## 🏗️ Architecture Overview

The platform is structured around several independent, high-performance microservices (Engines) communicating primarily via internal APIs and message queues (e.g., Kafka/NATS).

### Key Components (Go Engines)

| Engine                  | Priority       | Primary Role                                                                 |
|-------------------------|----------------|-----------------------------------------------------------------------------|
| **Accounting/Ledger Engine** | P0 (Foundation) | The single source of truth. Manages all account balances (crypto & fiat) using strict ACID transactions. |
| **JIT Funding Service** | P1 (Product Core) | Handles real-time authorization webhooks from the Card Processor, performing risk/balance checks. |
| **Liquidation Engine**  | P1 (Product Core) | Executes the instant crypto-to-fiat trade based on JIT approval, securing the necessary fiat liquidity for the card network, with BlockDAG integration for asset transfers. |
| **Wallet/Blockchain Engine** | P2 (Operations) | Securely manages hot/cold wallets, listens for on-chain deposits, and broadcasts signed withdrawals. |
| **Trading Engine**      | P3 (CEX Standard) | Manages order books and matches internal crypto trades (providing liquidity for the Liquidation Engine). |
| **User Service**        | Support        | Handles user registration, profile management, KYC/AML status tracking.    |

## 🛠️ Technology Stack

- **Backend & Engines:** **Golang (Go)**
- **Database (Ledger):** PostgreSQL (for ACID compliance)
- **Cache/Message Broker:** Redis, Kafka/NATS (for high-speed event processing)
- **Card Processing:** Integration via **External Card Issuing Partner API** (e.g., Blusalt, Wema Bank, Mastercard)
- **Security:** Fireblocks for custody, MPC wallets
- **Blockchain Integration:** BlockDAG Network for high-throughput, near-instant asset transfers
- **Monitoring:** Prometheus + Grafana
- **Deployment:** Docker, Kubernetes for orchestration
- **Frontend (Webapp):** React.js with Tailwind CSS (Shadow Buzz theme: black background with neon yellow/electric purple accents)

## ⚙️ JIT Card Spending Workflow (The Critical Path)

The entire flow must complete within typical network authorization windows (~500 ms), enhanced by BlockDAG for ultra-fast asset movement.

1. **Authorization Request:** Merchant terminal sends authorization request to the **Card Network**.
2. **Forwarding:** Network forwards the request to the **Card Issuing Partner**.
3. **JIT Call:** Partner sends a real-time, synchronous webhook to the **JIT Funding Service (Go)**.
4. **Validation & Reservation:** The JIT Service checks the **Ledger Engine** for sufficient crypto funds and calls the **Liquidation Engine** to **reserve** the required crypto and **provision** the fiat funds on the Card Processor's side.
   - **JIT Convert & Transfer (<150ms)**: The Go-lang Liquidation Engine:
     * Authenticates the request and checks spending limits.
     * Calculates the precise amount of the user's designated crypto (e.g., 0.0016 BTC) required.
     * Instant Asset Transfer: The crypto asset is moved from the MPC Vault to the exchange settlement wallet via the BlockDAG Network, leveraging its high throughput and near-instant finality.
     * Instant Liquidation: Instantly executes a market sale via the integrated crypto exchange API upon near-confirmation of the BlockDAG transaction.
5. **Response:** JIT Service returns `APPROVED` to the Card Partner.
6. **Settlement (Asynchronous):** The **Ledger Engine** records the final fiat debit and crypto credit/debit after the merchant batch settlement.

## Prerequisites

- Go 1.21 or higher
- PostgreSQL 15+
- Redis 7+
- Kafka or NATS for messaging
- Docker for containerization
- API keys for card issuer (e.g., Blusalt/Wema/Mastercard), custody provider (e.g., Fireblocks), and BlockDAG Network integration

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/username/bebolt.git
   cd bebolt
   ```

2. Install dependencies:
   ```
   go mod download
   ```

3. Set up environment variables (copy `.env.example` to `.env` and fill in values):
   - `DB_URL`: PostgreSQL connection string
   - `REDIS_ADDR`: Redis address
   - `KAFKA_BROKERS`: Kafka brokers
   - `CARD_ISSUER_API_KEY`: API key for card partner
   - `FIREBLOCKS_API_KEY`: For wallet custody
   - `BLOCKDAG_API_KEY`: For BlockDAG Network integration

4. Run database migrations:
   ```
   go run cmd/migrate/main.go
   ```

5. Build and run services (or use Docker Compose):
   ```
   docker-compose up -d
   ```

## Usage

- Start the JIT Funding Service: `go run cmd/jit-service/main.go`
- Access the API at `http://localhost:8080` (e.g., `/health` for status)
- For testing card issuance: Use the `/cards/issue` endpoint with mock crypto balances
- Simulate a transaction: POST to `/jit/authorize` with payload mimicking card webhook

Detailed API docs available via Swagger at `/swagger/index.html` (after running the server).

## Testing

- Unit tests: `go test ./...`
- Integration tests: `go test -tags=integration ./...`
- Load testing: Use tools like Apache Bench or Locust to simulate high-throughput authorizations (aim for <100ms latency, including BlockDAG transfers)

## Roadmap

- Q4 2025: MVP Launch (Virtual NGN/USD Cards in Nigeria/SA/UAE)
- Q1 2026: Physical Cards + Expansion to Ghana/Kenya/Saudi
- Q2 2026: Europe Integration (UK/DE/FR) + Merchant Pro Features
- Ongoing: Security audits, performance optimizations, BlockDAG enhancements

## Contributing

We welcome contributions! Please follow these steps:

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

See `CONTRIBUTING.md` for more details.

## License

Distributed under the MIT License. See `LICENSE` for more information.

## Contact

- Project Lead: @algonasr on X
- Email: info@naszat.tech
- Project Link: [https://github.com/Bee-Bolt/beebolt](https://github.com/Bee-Bolt/beebolt)

🐝⚡ Built with buzz – ready to sting the crypto market!
