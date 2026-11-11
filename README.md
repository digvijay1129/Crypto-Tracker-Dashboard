# CryptoTracker Dashboard

A dark-themed cryptocurrency market dashboard for exploring live market data, viewing individual coin performance, and comparing two cryptocurrencies over selectable historical periods.

The React application uses the public CoinGecko API for market data and visualizes price, market capitalization, and trading-volume trends through interactive charts.

---

## Features

### Market dashboard
- Fetches the top 50 cryptocurrencies by market capitalization in USD.
- Displays coin name, symbol, logo, current price, 24-hour price change, total volume, and market capitalization.
- Switches between **Grid** and **List** layouts.
- Searches the loaded market list by coin name or symbol.
- Paginates the market list with 10 coins per page.

### Coin details and historical charts
- Opens an individual coin details page.
- Displays coin information and description.
- Loads historical market data for **7, 30, 60, or 90 days**.
- Switches chart metrics between Price, Market Capitalization, and Total Trading Volume.
- Shows historical values in a line chart.

### Cryptocurrency comparison
- Selects two different cryptocurrencies from the available market list.
- Compares their market information and historical performance.
- Uses a multi-series chart with separate axes for the two selected assets.
- Supports the same 7, 30, 60, and 90-day periods and chart metric options.

### Data caching and fallback
- Caches market-list and coin-detail responses in browser local storage and in-memory state for approximately 30 seconds.
- Caches historical chart data in local storage for approximately 30 seconds.
- Reuses cached data when available and falls back to previously cached values if a CoinGecko request fails.
- Displays a loading state while market or coin data is being retrieved.

### Landing page and navigation
- Provides Home, Dashboard, and Compare navigation.
- Includes an animated landing-page introduction.
- Provides a share action through the browser's Web Share integration where supported.

---

## Application architecture

The main React application currently requests CoinGecko directly through Axios-based data helper functions. The Express service in the server directory is a separate, optional proxy API; the current React data helpers do not call that proxy.

`mermaid
flowchart TD
    User[User / Browser] --> App[React Application]
    App --> Router[React Router]
    Router --> Home[Home Page]
    Router --> Dashboard[Market Dashboard]
    Router --> Coin[Coin Details]
    Router --> Compare[Compare Page]

    Dashboard --> Helpers[Axios Data Helpers]
    Coin --> Helpers
    Compare --> Helpers

    Helpers --> Cache[Browser localStorage and in-memory cache]
    Helpers -->|HTTPS REST requests| CG[CoinGecko Public API]
    CG --> Helpers
    Helpers --> Transform[Data formatting and chart preparation]
    Transform --> UI[Material UI Components and Chart.js Visualizations]

    Proxy[Optional Express Proxy in server/server.mjs] -.->|Axios requests| CG
    Proxy -.->|JSON endpoints on port 5000| ExternalClient[Optional API Client]
`

### Data flow

1. The user opens the Dashboard, a coin details page, or the Compare page.
2. The page calls the corresponding data helper.
3. The helper checks its browser cache for a recent response.
4. When fresh cached data is unavailable, Axios requests the relevant CoinGecko endpoint.
5. The response is stored in the applicable cache and passed to the page.
6. Formatting helpers normalize coin fields, dates, and chart datasets.
7. React components render the market information and Chart.js line charts.
8. If an API request fails, previously cached data may be used where available.

### Optional Express proxy

The repository also contains a small Express 5 service that forwards requests to CoinGecko and returns JSON responses. It listens on port 5000 and exposes these routes:

| Method | Route | Purpose |
|---|---|---|
| GET | /api/100-coins | Fetches the first 100 market-listed coins in USD |
| GET | /api/coin-data/:id | Fetches details for a coin ID |
| GET | /api/coin-prices/:id?days=30 | Fetches historical market-chart data for the requested number of days |

The proxy is available as a standalone service. The current frontend helper functions use CoinGecko URLs directly, so starting the proxy is not required to run the React dashboard.

---

## Technology stack

| Layer | Technologies |
|---|---|
| Frontend framework | React 19 |
| Build and development tooling | Create React App, react-scripts 5 |
| Routing | React Router DOM 7 |
| UI components | Material UI 6, MUI Lab, Material Icons |
| Charts | Chart.js 4, react-chartjs-2 |
| HTTP requests | Axios |
| Animations | Framer Motion |
| Notifications | React Toastify |
| Sharing | react-web-share |
| Backend proxy | Node.js, Express 5, Axios, CORS |
| Data provider | CoinGecko public REST API |
| Browser caching | localStorage and in-memory JavaScript cache |

---

## Pages and routes

| Route | Page | Description |
|---|---|---|
| / | Home | Landing page with navigation, dashboard call-to-action, and share action |
| /dashboard | Dashboard | Market list/grid, coin search, and pagination |
| /coin/:id | Coin Details | Coin information and historical metric chart |
| /compare | Compare | Select and compare two cryptocurrencies |

---

## Project structure

The tree below shows the main application source. Installed dependency directories and generated build output are omitted.

`text
Crypto-Tracker-Dashboard/
├── public/
│   ├── favicon.ico
│   ├── index.html                 # HTML entry point and page metadata
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
│
├── src/
│   ├── assets/
│   │   ├── gradient.png           # Landing-page visual asset
│   │   └── iphone.png             # Landing-page visual asset
│   │
│   ├── components/
│   │   ├── common/
│   │   │   ├── BackToTop/          # Back-to-top control
│   │   │   ├── Button/             # Shared button
│   │   │   ├── Footer/             # Footer component
│   │   │   ├── Header/             # Navigation and mobile drawer
│   │   │   └── Loader/             # Loading indicator
│   │   │
│   │   ├── landingpage/
│   │   │   └── MainComponenet/     # Landing-page hero and share action
│   │   │
│   │   ├── dashboard/
│   │   │   ├── grid/               # Cryptocurrency grid card
│   │   │   ├── list/               # Cryptocurrency list row
│   │   │   ├── pagging/            # Dashboard pagination
│   │   │   ├── search/             # Coin name/symbol search
│   │   │   └── tabs/               # Grid and List view switcher
│   │   │
│   │   ├── coin/
│   │   │   ├── Coininfo/           # Coin description section
│   │   │   ├── LineChart/          # Historical data visualization
│   │   │   ├── PriceType/          # Price, market cap, volume selector
│   │   │   └── SelectDays/         # Historical period selector
│   │   │
│   │   └── ComparePage/
│   │       └── SelectCoins/        # Two-coin selection controls
│   │
│   ├── functions/
│   │   ├── convertDate.js          # Timestamp-to-date formatting
│   │   ├── convertNumber.js        # Large-number formatting
│   │   ├── convertobject.js        # Coin API response normalization
│   │   ├── get100Coin.js           # Market-list request and cache
│   │   ├── getCoinData.js          # Individual coin request and cache
│   │   ├── getCoinPrices.js        # Historical metric request and cache
│   │   └── settingChartdata.js     # Chart.js dataset preparation
│   │
│   ├── pages/
│   │   ├── Home.js                 # Home route
│   │   ├── Dashboard.js            # Market dashboard route
│   │   ├── Coin.js                 # Coin details route
│   │   └── ComparePage.js          # Comparison route
│   │
│   ├── App.css
│   ├── App.js                      # Application routes
│   ├── index.css                   # Global styles and theme variables
│   └── index.js                    # React application entry point
│
├── server/
│   ├── server.mjs                  # Optional CoinGecko Express proxy
│   ├── package.json                # Proxy service dependencies and start script
│   └── package-lock.json
│
├── package.json                    # React application dependencies and scripts
└── package-lock.json
`

---

## Data source and API usage

Market data is provided by the public CoinGecko API. The frontend uses USD as the quote currency.

| Application data | CoinGecko endpoint | Use |
|---|---|---|
| Market list | /api/v3/coins/markets | Current market metrics, ordered by market capitalization |
| Coin information | /api/v3/coins/{id} | Individual coin metadata and description |
| Historical chart | /api/v3/coins/{id}/market_chart | Historical prices, market capitalization, and total volume |

The frontend market-list helper requests 50 results for page 1, ordered by market capitalization. The standalone Express proxy's /api/100-coins route requests 100 results.

Historical chart requests use the selected period and daily interval. The chart metric selector chooses the prices, market_caps, or total_volumes series returned by the market-chart endpoint.

CoinGecko request availability and rate limits depend on the provider and its current service plan. The application does not provide its own market-data feed.

---

## Run locally

### Prerequisites

- Node.js and npm
- Internet connection to retrieve CoinGecko market data

### 1. Clone the repository

`bash
git clone https://github.com/digvijay1129/Crypto-Tracker-Dashboard.git
cd Crypto-Tracker-Dashboard
`

### 2. Install frontend dependencies

From the project root:

`bash
npm install
`

### 3. Start the React application

`bash
npm start
`

Open http://localhost:3000 in your browser.

The development server reloads when source files change.

### 4. Start the optional Express proxy

The proxy is not required by the current frontend. To run it separately, open another terminal:

`bash
cd server
npm install
npm start
`

The proxy listens at http://localhost:5000.

Available local proxy routes:

`http
GET http://localhost:5000/api/100-coins
GET http://localhost:5000/api/coin-data/bitcoin
GET http://localhost:5000/api/coin-prices/bitcoin?days=30
`

---

## Available scripts

Run these commands from the project root:

| Command | Description |
|---|---|
| npm start | Starts the React development server |
| npm run build | Creates the optimized production build in build/ |
| npm test | Starts the Create React App test runner |
| npm run eject | Ejects Create React App configuration; this operation is irreversible |

Run npm start from the server directory to start the optional Express proxy.

---

## Current scope

The application focuses on public cryptocurrency market exploration and historical-data visualization.

- The dashboard displays public market data; it is not a connected brokerage or exchange account.
- The project does not execute trades or manage deposits and withdrawals.
- A personal investment portfolio, transaction ledger, and price-alert notification service are not part of the current routed application.
- The Watchlist route is not currently enabled.
- The current React frontend fetches CoinGecko directly; the separate Express proxy is not connected to the frontend data helpers.
- Market values and historical series depend on CoinGecko availability and should not be treated as guaranteed real-time exchange execution prices.

---

## License

No license file is currently included in this repository.