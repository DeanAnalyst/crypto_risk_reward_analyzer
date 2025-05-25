# Crypto Horizon: Navigating Risk and Reward in the Digital Asset Landscape

## Project Overview

This project performs a cross-sectional analysis of a cryptocurrency market snapshot to profile digital assets based on custom-defined "Risk" and "Reward/Activity" scores. It visualizes these profiles on an interactive 2D scatter plot and employs an anomaly detection model (Isolation Forest) to identify cryptocurrencies exhibiting unusual market characteristics relative to their peers. The goal is to provide a framework for understanding the diverse risk-reward dynamics within the crypto market at a given point in time.

**View the interactive analysis notebook (converted to HTML):** [https://deananalyst.github.io/crypto_risk_reward_analyzer/notebooks/01_crypto_risk_reward_analysis.html]

## Dataset

The analysis uses a snapshot of the cryptocurrency market, `crypto_market_dataset.csv`, containing information for 1000 cryptocurrencies. Key features include:
*   Name, Symbol
*   Current Price (USD)
*   Market Cap
*   24h Change (%)
*   Total Volume
*   Last Updated

## Methodology

### 1. Data Cleaning and Preprocessing
*   **Loading:** The dataset was loaded using pandas.
*   **Type Conversion:** Relevant columns (`Current Price (USD)`, `Market Cap`, `24h Change (%)`, `Total Volume`) were ensured to be numeric. `Last Updated` was converted to datetime.
*   **Missing Values:**
    *   `24h Change (%)`: 8 coins had missing values. Given their `Last Updated` timestamps were also significantly older, these were initially imputed with 0. For a production system, a more robust strategy (e.g., removal or specific flagging) would be considered.
*   **Filtering:**
    *   Coins with zero Market Cap AND zero Total Volume were removed as they represent inactive assets.
    *   Specific tokenized funds or entities not representing typical actively traded cryptocurrencies (e.g., "BlackRock USD Institutional Digital Liquidity Fund", "OUSG") were filtered out to avoid skewing the analysis.
    *   Coins with extremely low Total Volume (e.g., < $100) were filtered as they are highly illiquid.
    *   After cleaning, the dataset comprised 975 cryptocurrencies.

### 2. Feature Engineering
Several new features were created to aid in the analysis and scoring:
*   **Log Transformations:** `Market Cap Log`, `Total Volume Log`, `Current Price Log` were created using `np.log1p` to handle skewness in these distributions.
*   **Liquidity Ratio:** Calculated as `Total Volume / Market Cap`. This provides an indicator of how easily an asset can be traded relative to its overall market value.
*   **Absolute 24h Change:** `abs(24h Change (%))` as a proxy for daily price volatility.
*   **Is Stablecoin (Heuristic):** A boolean flag to identify potential stablecoins (price between $0.97-$1.03 and absolute 24h change < 0.5%). These 73 identified stablecoins were excluded from the primary risk-reward scatter plot to focus on assets with appreciation potential.
*   **Market Cap Tier:** Coins were binned into 'Micro Cap', 'Small Cap', 'Mid Cap', 'Large Cap', 'Mega Cap' based on their Market Cap for descriptive purposes.

### 3. Risk & Reward Score Calculation
Custom "Risk" and "Reward/Activity" scores were calculated for each non-stablecoin:
*   **Risk Score Components (Normalized 0-1):**
    *   `Inverse Market Cap Log` (0.4 weight): Lower market cap implies higher risk.
    *   `Absolute 24h Change` (0.4 weight): Higher volatility implies higher risk.
    *   `Inverse Liquidity Ratio` (0.2 weight): Lower liquidity implies higher risk.
*   **Reward/Activity Score Components (Normalized 0-1):**
    *   `Positive 24h Change (%)` (0.4 weight): Higher positive price change implies higher reward (negative changes clipped at 0).
    *   `Total Volume Log` (0.3 weight): Higher volume implies higher market activity.
    *   `Liquidity Ratio` (0.3 weight): Higher liquidity implies better ability to realize gains.
The weights are subjective and chosen to balance different aspects of risk and reward.

### 4. Anomaly Detection
*   An **Isolation Forest** model was trained on scaled versions of `Absolute 24h Change`, `Liquidity Ratio`, and `Market Cap Log`.
*   The `contamination` parameter was set to 0.03, implying an expectation that roughly 3% of the data points might be anomalies.
*   This identified 28 coins as behaving unusually based on these fundamental metrics.

## Key Findings & Visualizations

### Interactive Risk-Reward Profile
The core visualization is an interactive scatter plot (generated with Plotly) where:
*   **X-axis:** Calculated Risk Score
*   **Y-axis:** Calculated Reward/Activity Score
*   **Color:** Indicates if a coin is an anomaly (Red) or not (Blue).
*   **Size:** Represents Market Cap (Log-transformed).
*   **Hover:** Provides detailed information for each coin.

Below is a static preview of the risk-reward profile:

![Crypto Risk-Reward Profile Preview](output/plots/risk_reward_profile.png)

[View the interactive Risk-Reward Profile](https://deananalyst.github.io/crypto_risk_reward_analyzer/output/interactive_plots/risk_reward_profile.html)


### Quadrant Analysis
Based on the mean Risk and Reward scores, the crypto assets were segmented:

*   **Low Risk, High Reward (238 coins):**
    *   These coins, according to the model, offer better potential returns for their perceived risk level.
    *   *Top Example (by Reward Score):* Alpaca Finance (Reward: 0.61, Risk: 0.47).
    *   *Further Analysis:* This quadrant is desirable. Coins here often exhibit recent positive price action with reasonable market cap and liquidity. It's worth investigating if their positive momentum is sustainable or due to specific news/developments.

*   **High Risk, High Reward (200 coins):**
    *   These assets show high activity and potential for gains but come with higher volatility or lower liquidity/market cap.
    *   *Top Example (by Reward Score):* XPLA (Reward: 0.62, Risk: 0.97), TokenFi (Reward: 0.56, Risk: 0.87). These coins showed very high 24h gains (59%, 47% respectively).
    *   *Further Analysis:* These are the "high-flyers" or potentially more speculative plays. The high reward score is often driven by significant recent price increases.

*   **Low Risk, Low Reward (144 coins):**
    *   Characterized by greater stability, lower volatility, and typically lower recent price changes or trading activity.
    *   *Top Example (by Risk Score):* LEO Token (Risk: 0.40, Reward: 0.16), Wrapped eETH (Risk: 0.42, Reward: 0.17).
    *   *Further Analysis:* These might be more established coins with less dramatic price swings, or those currently in a consolidation phase.

*   **High Risk, Low Reward (320 coins):**
    *   The largest group. These coins exhibit higher risk characteristics (e.g., high volatility, low liquidity, smaller cap) without corresponding high reward/activity scores in this snapshot.
    *   *Top Example (by Risk Score):* GENIUS AI (Risk: 0.70, Reward: 0.18), Cudos (Risk: 0.70, Reward: 0.15). Entangle (Risk: 0.69, Reward: 0.08) showed a significant loss (-23.89%).
    *   *Further Analysis:* Coins in this quadrant may be experiencing downturns, are inherently very speculative with low current positive momentum, or are very new/illiquid.

### Anomaly Detection Insights
The Isolation Forest identified 28 anomalies. The top anomalies (by market cap) included:
*   **Bitcoin (BTC):** Risk: 0.20, Reward: 0.30. Likely flagged due to its unique combination of extremely high market cap and volume, which makes its other metrics (like liquidity ratio or % change) behave differently compared to the vast majority of smaller coins. Its "anomaly" status here means it's unique, not necessarily "bad."
*   **Ethereum (ETH):** Risk: 0.27, Reward: 0.29. Similar reasoning to Bitcoin.
*   **XRP (XRP):** Risk: 0.29, Reward: 0.27.
*   **BNB (BNB):** Risk: 0.31, Reward: 0.24.
*   **Official Trump (TRUMP):** Risk: 0.45, Reward: 0.28. Showed a -5.20% change. This, combined with its other metrics, likely set it apart.
*   **Virtuals Protocol (VIRTUAL):** Risk: 0.64, Reward: 0.46. Notable for a +27.75% 24h change, contributing to its anomalous high reward for its risk profile.
*   **Pudgy Penguins (PENGU):** Risk: 0.56, Reward: 0.25. Showed a significant -14.14% 24h change, making its volatility component high.

**General Anomaly Observations:**
Anomalies were often coins with:
1.  Extremely high or low `24h Change (%)` (e.g., Virtuals Protocol, Pudgy Penguins, Entangle).
2.  Unusual `Liquidity Ratio` for their `Market Cap` (very high for small caps, or surprisingly low for larger ones).
3.  The largest cap coins (BTC, ETH) are often flagged simply because their scale is an outlier compared to the bulk of the dataset.

## How to Run
1.  Clone the repository.
2.  Ensure Python 3.x and the required libraries (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `plotly`) are installed.
    ```bash
    pip install pandas numpy matplotlib seaborn scikit-learn plotly
    ```
3.  Place the `crypto_market_dataset.csv` in the `data/` directory.
4.  Open and run the `notebooks/01_crypto_risk_reward_analysis.ipynb` Jupyter Notebook.
5.  Interactive plots will be saved to the `output/interactive_plots/` directory.

## Conclusion
This project demonstrates a methodology for profiling cryptocurrencies based on risk and reward metrics derived from a market snapshot. The interactive visualization allows for exploration of the crypto landscape, and the anomaly detection helps pinpoint assets with distinctive characteristics that warrant further investigation.

The framework highlights that "risk" and "reward" are multifaceted. For instance, large-cap coins like Bitcoin, while flagged as anomalies due to their scale, generally fall into a lower-risk profile by this model's definition compared to smaller, more volatile assets. The "High Risk, High Reward" quadrant contains many coins that saw significant recent gains, emphasizing the volatile nature of such returns.

## Future Work
*   **Dynamic Analysis:** Incorporate time-series data to track how risk/reward profiles change over time.
*   **Refined Scoring:** Experiment with different feature weights, add more nuanced features (e.g., developer activity, social sentiment).
*   **Advanced Anomaly Detection:** Explore other algorithms (e.g., One-Class SVM, Autoencoders) or ensemble methods.
*   **Qualitative Overlays:** Integrate qualitative data (e.g., project category, whitepaper analysis) for richer profiling.
*   **Interactive Dashboard:** Develop a Dash or Streamlit dashboard for more dynamic user interaction (e.g., allowing users to set their own risk/reward weights).