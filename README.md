# Incentives Optimization and Bid Formulation

## 1.1 General Information and Purpose
- **Service Name/Title:** `incentives_optimization_and_bid_formulation`
- **Description and purpose:** Calculating optimal incentives to encourage consumer participation in flexibility actions and formulates an optimal bid (quantity, price) for the Local Flexibility Market.
- **Owner/Contact Information:** ICCS

---

## 1.2 Functional Requirements
1. The system should determine the optimal incentives for consumer participation.
2. The system should be able to provide specific incentives in euros for all the different house groups.
3. The system should be able to provide such incentives that it makes sense for the customers to participate.
4. The system should be able to provide such incentives that the aggregator has profit.
5. The system should take into account the behavior of the customers and their response probability.
6. The system should calculate an aggregated bid in terms of quantity and price.
7. The system should handle real-time data from multiple consumers efficiently to quickly formulate incentives and bids.

---

## 1.3 Non-Functional Requirements
- The calculated incentives should be available in a few seconds.
- Access to the incentive calculation should be limited to registered users.
- Periodic backups with automated recovery in case of failure.
- 99.9% uptime for the service.

---

## 1.4 Service Interfaces

### 1.4.1 API Endpoints

#### Endpoint 1 — Calculate Incentives to Houses and the Bid to the LFM
- **URL:** `/incentives/optimize-bid`
- **Method:** `POST`
- **Description:** Calculates optimal incentives and formulates a bid for submission in the Local Flexibility Market (LFM).

**Headers**
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**Request Example**
```json
{
  "consumer_portfolio": "portfolio_1",
  "mtu": "2025-05-12T14:00:00Z",
  "forecasted_data": [
    {
      "house_id": "C001",
      "forecasted_demand": 100.0,
      "forecasted_production": 50.0,
      "response_probability": 0.75,
      "mtu": "2025-05-12T14:00:00Z",
      "battery_capacity": 20,
      "battery_soc": 0.2
    },
    {
      "house_id": "C002",
      "forecasted_demand": 150.0,
      "forecasted_production": 75.0,
      "response_probability": 0.58,
      "mtu": "2025-05-12T14:00:00Z",
      "battery_capacity": 10,
      "battery_soc": 0.25
    }
  ],
  "forecasted_market_price": 0.18,
  "incentive_price": 0.1,
  "incentive_levels": [5.0, 10.0, 15.0]
}
```

**Response Example**
```json
{
  "optimization_id": "string",
  "consumer_portfolio": "portfolio_1",
  "incentive_per_house": [
    { "house_id": "C001", "incentive_in_euros": 15.5, "expected_reduction": 50.0 },
    { "house_id": "C002", "incentive_in_euros": 20.0, "expected_reduction": 75.0 }
  ],
  "total_incentive": 35.5,
  "bid_details": {
    "bid_id": "B456",
    "quantity": 125.0,
    "price_per_kWh": 0.18,
    "mtu": "2025-05-12T14:00:00Z",
    "bid_direction": "SELL",
    "consumer_portfolio": "portfolio_1"
  },
  "message": "Incentives calculated successfully for all consumers.",
  "status": "calculated",
  "timestamp": "2023-09-28T15:30:00Z"
}
```

**Error Handling**
- `400 Bad Request`: Invalid bid data format  
- `401 Unauthorized`: Invalid token  
- `403 Forbidden`: Caller role not authorized  
- `404 Not Found`: MTU or consumer_portfolio not found  

---

#### Endpoint 2 — Retrieve Past Optimization Run Details
- **URL:** `/incentives/optimize-bid/{optimization_id}`
- **Method:** `GET`
- **Description:** Fetch details of a previously executed optimization run.

**Request Example**
```json
{ "optimization_id": "opt-7890AB" }
```

**Response Example**
```json
{
  "optimization_id": "opt-7890AB",
  "consumer_portfolio": "portfolio_1",
  "incentive_per_house": [
    { "house_id": "C001", "incentive_in_euros": 15.5, "expected_reduction": 50.0 },
    { "house_id": "C002", "incentive_in_euros": 20.0, "expected_reduction": 75.0 }
  ],
  "total_incentive": 35.5,
  "bid_details": {
    "bid_id": "B456",
    "quantity": 125.0,
    "price_per_kWh": 0.18,
    "mtu": "2025-05-12T14:00:00Z",
    "bid_direction": "SELL",
    "consumer_portfolio": "portfolio_1"
  },
  "message": "Incentives calculated successfully for all consumers.",
  "status": "calculated",
  "timestamp": "2023-09-28T15:30:00Z"
}
```

**Error Handling**
- `400 Bad Request`: Invalid optimization_id format  
- `401 Unauthorized`: Invalid token  
- `403 Forbidden`: Caller role not authorized  
- `404 Not Found`: No record found for optimization_id  

---

## 1.5 Data Model

### Input Example
```json
{
  "consumer_portfolio": "portfolio_1",
  "mtu": "2025-05-12T14:00:00Z",
  "forecasted_data": [
    { "house_id": "C001", "forecasted_demand": 100.0, "forecasted_production": 50.0, "response_probability": 0.75, "battery_capacity": 20, "battery_soc": 0.2 },
    { "house_id": "C002", "forecasted_demand": 150.0, "forecasted_production": 75.0, "response_probability": 0.58, "battery_capacity": 10, "battery_soc": 0.25 }
  ],
  "forecasted_market_price": 0.18,
  "incentive_price": 0.1,
  "incentive_levels": [5.0, 10.0, 15.0]
}
```

### Output Example
```json
{
  "optimization_id": "opt-7890AB",
  "consumer_portfolio": "portfolio_1",
  "incentive_per_house": [
    { "house_id": "C001", "incentive_in_euros": 15.5, "expected_reduction": 50.0 },
    { "house_id": "C002", "incentive_in_euros": 20.0, "expected_reduction": 75.0 }
  ],
  "total_incentive": 35.5,
  "bid_details": {
    "bid_id": "B456",
    "quantity": 125.0,
    "price_per_kWh": 0.18,
    "mtu": "2025-05-12T14:00:00Z",
    "bid_direction": "SELL",
    "consumer_portfolio": "portfolio_1"
  },
  "message": "Incentives calculated successfully for all consumers.",
  "status": "calculated",
  "timestamp": "2023-09-28T15:30:00Z"
}
```

---

## 1.6 Entities and Relationships
- **Houses**: Store basic info (name, address, area, year, floor, energy_class, heating_type, cooling_type, invoice_provider).  
- **Forecasted_data**: Predictions for consumption, production, battery, and response probability.  
- **Portfolio**: Collection of forecasted energy data, market price, associated Incentives.  
- **Incentive**: Defines schemes, prices, reductions, and values in euros.  
- **Optimization**: Stores output of optimization, incentives, bids, status, messages.  
- **Bid**: Market bids linked to optimization results.  

---

## 1.7 Integration and Dependencies
- **Internal Data Sources:** historical bid data, real-time forecasts.  
- **Edge and Cloud Integration:** residential UIs, aggregators, smart meters, forecasting modules, cloud storage.  
- **Trading Service:** generates optimal bid for aggregator submission.  
- **App Store/Data Space Integration:** to be updated.  

---

## 1.8 Security and Privacy
- **Data Sensitivity:** includes market data, incentives, forecasts, personally identifying consumption profiles.  
- **Encryption:** TLS 1.2+ in transit, AES-256 at rest, encrypted backups.  
- **RBAC:** Operator (optimize only), Administrator (full access).  
- **Audit Logging:** immutable logs of optimization events.  
- **Anomaly Detection:** alerts for abnormal bids or unauthorized access.  

