# STORMCAST

**A desktop trading terminal for weather markets.** It reads the full ECMWF ensemble forecast, builds a calibrated probability distribution for a given temperature outcome, and compares that against what a prediction market is pricing.

Cross-platform desktop app (macOS, Windows, Linux) with auto-update. **[Download the latest release](https://github.com/isashahid10/stormcast-releases/releases/latest).**

> This repository hosts releases and documentation. STORMCAST is a commercial product and the application source is private. What follows is the engineering, in as much detail as I can give without handing over the strategy.

---

## The idea

Ask your phone for tomorrow's high and it gives you one number. That number is a lie of omission.

What the model actually produced was a spread of possible outcomes. The ECMWF ensemble runs the forecast 51 times from slightly perturbed starting conditions, and you get 51 answers. Sometimes they cluster inside a degree, which means the atmosphere is being predictable. Sometimes they spray across eight degrees, which means tomorrow is genuinely uncertain.

**Those two situations look identical if all you see is the average.** Traders working from a single-number city forecast cannot distinguish them. STORMCAST is built on the bet that the difference is worth money.

## What it does

- Ingests the **51-member ECMWF ensemble** for specific ASOS airport weather stations, which is what these markets actually settle against, rather than a city-level approximation
- Builds a **calibrated temperature distribution** instead of a point estimate
- Prices that distribution against live market odds and surfaces an **expected-value-ranked signal feed** with confidence tiering
- **Fractional Kelly position sizing**, because the failure mode of a system like this is not being wrong, it is being right on average and going broke anyway
- **Paper trading with full P&L tracking** and **calibration analytics**: when the model said 70%, did it happen about 70% of the time?
- Push notifications on high-confidence signals
- Wallet connection for balance display

## Architecture

```
┌──────────────────────────────┐
│  Electron shell              │  auto-update, secure credential
│  ┌────────────────────────┐  │  storage, machine-bound licensing
│  │  React + Vite UI       │  │
│  │  47 modules            │  │  charts, signal feed, calibration
│  └───────────┬────────────┘  │  views, portfolio state
│              │ local IPC     │
│  ┌───────────┴────────────┐  │
│  │  Python engine         │  │  forecast ingest, market data,
│  │  14 modules            │  │  model, signals, paper trader,
│  └────────────────────────┘  │  telemetry, notifications
└──────────────────────────────┘
```

**Why two languages.** The forecasting and statistics ecosystem lives in Python and I was not going to reimplement it in JavaScript for the sake of tidiness. The interface wanted to be a real desktop app. So: Node frontend, Python engine, local socket bridge between them. You pay a small tax at the boundary and it is far cheaper than the rewrite.

**Why a desktop app rather than a web app.** It runs a continuous local process against forecast data on a schedule, holds state between runs, and needs to work without me operating a server for every user.

## Engineering notes

| Concern | Approach |
| --- | --- |
| Distribution | `electron-builder` producing signed macOS (Intel + Apple Silicon), Windows and Linux artifacts |
| Updates | `electron-updater` against this repository's releases, with blockmaps for delta downloads |
| Licensing | Machine-bound activation, credentials held in the OS keychain rather than on disk |
| State | Zustand store on the UI side, persisted engine state on the Python side |
| Testing | Unit tests across the engine and UI |
| Observability | Structured logging, plus telemetry on signal generation and calibration drift |

## Built with

TypeScript, React, Vite, Electron, Tailwind, Zustand, TanStack Query, Recharts, ethers, Python, ECMWF open data.

## Why the source is private

STORMCAST is sold, and it has two properties that do not survive being published:

1. **The edge is the strategy.** A forecasting model whose signal generation is public stops being an edge roughly immediately.
2. **Licensing.** Publishing the client publishes the activation path.

If you are evaluating my work and want to go deeper than this page, I am happy to walk through the architecture, the calibration methodology, or the build and release pipeline in a call. Contact details at [isashahid.netlify.app](https://isashahid.netlify.app).

## Disclaimer

STORMCAST is a research and analysis tool. It is not financial advice, it does not guarantee returns, and prediction market access and legality vary by jurisdiction. Users are responsible for compliance where they live.

## Licence

Application source: proprietary, all rights reserved. This repository's documentation is provided for evaluation.
