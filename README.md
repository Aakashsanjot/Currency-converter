# DecodeLabs Currency Converter — Java Programming Project 4

> **Industrial Training Kit | Batch 2026 | Powered by DecodeLabs**
> *Project 4 — Optional Mastery Phase: The Financial Translation Engine*

An enterprise-grade, console-based currency converter written in Java. Built
to satisfy the DecodeLabs Project 4 brief: precise financial arithmetic,
defensive input handling, a clean menu-driven UX, and an optional live
exchange-rate API integration.

---

## Table of Contents

- [Project Goal](#project-goal)
- [Requirements Checklist](#requirements-checklist-mapped-to-implementation)
- [Architecture](#architecture)
- [Folder Structure](#folder-structure)
- [Getting Started](#getting-started)
- [Troubleshooting](#troubleshooting-package-does-not-exist--class-main-is-public-should-be-declared-in)
- [Usage / Sample Run](#usage--sample-run)
- [Running the Tests](#running-the-tests)
- [Live Exchange Rates (Optional)](#live-exchange-rates-optional)
- [Pushing to GitHub](#pushing-to-github)
- [Tech Stack](#tech-stack)
- [License](#license)

---

## Project Goal

Create a Java program that converts currency from one type to another,
letting the user select a base and target currency, enter an amount, and
see the converted amount displayed clearly — using either predefined static
rates or live fetched exchange rates.

## Requirements Checklist (mapped to implementation)

The project brief ends with a "Gatekeeper Checklist" of four criteria. Here
is exactly where each one is satisfied in the code:

| # | Requirement | Where it's implemented |
|---|-------------|-------------------------|
| 1 | **Multi-Currency Support** — handle at least 3 currencies and route cross-rates accurately | `Currency` enum (8 currencies) + `CurrencyConverter.convert()` routes every pair through a USD pivot |
| 2 | **Input Integrity** — block negative numbers and survive `InputMismatchException` / typos | `InputHandler` (buffer-safe reads, retry loops) + `InvalidAmountException` (Security Gate) |
| 3 | **Financial Accuracy** — strictly accurate calculations using `BigDecimal` to avoid the `double` precision crisis | Every number in `CurrencyConverter` and `StaticExchangeRateProvider` is a `BigDecimal` built from `String`, never a `double` literal |
| 4 | **Formatting Precision** — final output uniformly formatted to exactly two decimal places | `CurrencyFormatter` (`RoundingMode.HALF_EVEN` + `%,.2f`) |

Additional brief requirements also implemented:

- do-while loop menu (runs at least once, restarts until the user exits)
- switch statement "switch board" with a default trap for invalid choices
- Security Gate rejects negative amounts and routes back to the menu instead of crashing
- Buffer-trap-safe input reading (no infinite loop on bad tokens)
- RoundingMode.HALF_EVEN ("Banker's Rounding") throughout
- Optional "Turbocharger" stage: live REST API rates via HttpURLConnection + GSON JSON parsing

## Architecture

The engine follows the **IPO model** described in the project brief —
**I**ntake -> **P**rocess -> **O**utput — plus an optional **Turbocharger**
stage for enterprise scale:

```
 +-----------+     +-----------------+     +-----------+     +----------------+
 |  INTAKE   | --> |   COMBUSTION    | --> |  EXHAUST  |     |  TURBOCHARGER  |
 |  (Input)  |     |    (Process)    |     | (Output)  |     | (Enterprise)   |
 +-----------+     +-----------------+     +-----------+     +----------------+
 Scanner reads      CurrencyConverter        CurrencyFormatter   LiveExchangeRateProvider
 menu choices,      does BigDecimal          renders 2-decimal   fetches real-time rates
 currency codes,    cross-rate math via      symbol-prefixed     via HttpURLConnection +
 and the amount.    the USD pivot.           output.             GSON JSON parsing.
      |                     |                                            |
      v                     v                                            |
 InputHandler         InvalidAmountException                             |
 (buffer-safe,        ("Security Gate" - negative                        |
 retry-on-error)      amounts rejected, routed                           |
                       back to menu)                                     |
                                                                          |
                       ExchangeRateProvider (interface) <-----------------+
                       implemented by StaticExchangeRateProvider (offline)
                       and LiveExchangeRateProvider (online) interchangeably.
```

Swapping between static and live rates never touches the conversion math —
that's the entire point of the `ExchangeRateProvider` interface.

## Folder Structure

```
decodelabs-currency-converter/
├── .github/
│   └── workflows/
│       └── maven-ci.yml              # GitHub Actions: build + test on every push
├── sample-output/
│   └── sample_run.txt                # Captured console transcript of a real run
├── src/
│   ├── main/
│   │   └── java/tech/decodelabs/currencyconverter/
│   │       ├── Main.java                          # Entry point, do-while menu, switch board
│   │       ├── core/
│   │       │   └── CurrencyConverter.java          # BigDecimal cross-rate conversion engine
│   │       ├── exception/
│   │       │   └── InvalidAmountException.java     # Security Gate exception
│   │       ├── model/
│   │       │   └── Currency.java                   # Supported currency enum
│   │       ├── service/
│   │       │   ├── ExchangeRateProvider.java        # Rate-source abstraction (interface)
│   │       │   ├── StaticExchangeRateProvider.java  # Hardcoded offline rates
│   │       │   └── LiveExchangeRateProvider.java    # Live REST API + GSON parsing
│   │       └── util/
│   │           ├── InputHandler.java                # Buffer-safe Scanner wrapper
│   │           └── CurrencyFormatter.java            # Two-decimal financial formatting
│   └── test/
│       └── java/tech/decodelabs/currencyconverter/
│           └── CurrencyConverterTest.java           # JUnit 5 test suite
├── .gitignore
├── LICENSE
├── pom.xml
└── README.md
```

## Getting Started

### Prerequisites

- JDK 17 or newer
- Maven 3.8+ (or use your IDE's built-in Maven support)

### Build

```bash
mvn clean package
```

This compiles the project, runs the test suite, and produces a runnable
fat jar (with GSON bundled in) at `target/currency-converter.jar`.

### Run (development mode, no jar needed)

```bash
mvn compile exec:java
```

## Troubleshooting: "package does not exist" / "class Main is public, should be declared in..."

If you see errors like these, you (or an editor extension) compiled a
**single file in isolation** instead of building the whole project:

```
tempCodeRunnerFile.java:28: error: class Main is public, should be declared in a file named Main.java
tempCodeRunnerFile.java:3: error: package tech.decodelabs.currencyconverter.core does not exist
```

**Cause:** VS Code's **"Code Runner"** extension, when you click its ▶ Run
button on an open `.java` file, copies that file to `tempCodeRunnerFile.java`
in the *same folder* and compiles it alone with plain `javac` — with no
knowledge of the other packages (`core`, `model`, `service`, `util`,
`exception`) or of GSON. This project is a multi-package Maven project, so
that approach cannot work; it isn't a bug in the code itself.

**Fix — don't use Code Runner's ▶ button for this project.** Use one of:

1. **Terminal (recommended):**
   ```bash
   mvn clean package
   java -jar target/currency-converter.jar
   ```
2. **VS Code, correctly:** install the official **"Extension Pack for Java"**
   (Microsoft/Red Hat) and **"Maven for Java"** extensions, open the project
   folder root (not a single file), then press **F5** — this repo's
   `.vscode/launch.json` is already configured to run `Main` with the full
   classpath.
3. **If Code Runner is already installed:** this repo's
   `.vscode/settings.json` overrides Code Runner's Java command to call
   Maven instead, so its ▶ button becomes safe to use too.

If a stray `tempCodeRunnerFile.java` file exists anywhere under `src/`,
delete it — it is never part of this project and will break compilation
every time it's present.

## Usage / Sample Run

```
==============================================================
   DecodeLabs Industrial Training Kit -- Java Project 4
   The Financial Translation Engine (Currency Converter)
==============================================================

Main Menu:
  1. Convert Currency (Static Rates)
  2. Convert Currency (Live Rates - requires API key)
  3. View Supported Currencies & Static Rates
  4. Exit
Enter your choice: 1
Enter source currency (e.g. USD): USD
Enter target currency (e.g. INR): INR
Enter amount to convert: 1500

--- Conversion Result (Static Offline Rates (Project 4 baseline)) ---
Source Amount   : $    1500.00
Converted Amount: ₹  125119.20
------------------------------------------------------------
```

See `sample-output/sample_run.txt` for a full captured transcript,
including the negative-amount Security Gate and invalid-input recovery
in action.

## Running the Tests

```bash
mvn test
```

`CurrencyConverterTest` verifies:
- Direct conversion accuracy against known rates
- Cross-rate routing through the USD pivot (e.g. EUR to INR)
- Negative amounts are rejected via `InvalidAmountException`
- Every result is scaled to exactly two decimal places
- Zero amounts are handled without error

## Live Exchange Rates (Optional)

The "Turbocharger" stage connects to exchangerate-api.com for real-time
rates instead of the static table.

1. Get a free API key from exchangerate-api.com.
2. Set it as an environment variable:
   ```bash
   export EXCHANGE_RATE_API_KEY=your_key_here
   ```
3. Choose option **2** from the main menu.

If the key is missing or the API is unreachable, the app tells you clearly
and lets you fall back to static rates — it never crashes.

## Pushing to GitHub

This repository is ready to push as-is:

```bash
cd decodelabs-currency-converter
git init
git add .
git commit -m "Initial commit: DecodeLabs Currency Converter (Project 4)"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/decodelabs-currency-converter.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username (also update it in
`pom.xml`'s `<url>` tag). The included `.github/workflows/maven-ci.yml`
will automatically build and test the project on every push once it's on
GitHub — no extra setup needed.

## Tech Stack

- **Java 17**
- **Maven** — build & dependency management
- **BigDecimal / MathContext / RoundingMode** — precision-safe financial arithmetic
- **GSON** — JSON parsing for the live exchange-rate API
- **JUnit 5** — automated testing
- **GitHub Actions** — CI

## License

MIT — see LICENSE file.

---

*Submitted as part of the DecodeLabs Industrial Training Kit, Batch 2026.*

