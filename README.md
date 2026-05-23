<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>FNB DataQuest — Interpretable Credit Models</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700;900&family=IBM+Plex+Mono:wght@400;500&family=IBM+Plex+Sans:wght@300;400;500&display=swap" rel="stylesheet" />
<style>
  :root {
    --ink:       #0d0d0d;
    --paper:     #f5f0e8;
    --accent:    #c8422a;
    --gold:      #b8922a;
    --mid:       #4a4a4a;
    --rule:      #d4ccc0;
    --tag-bg:    #e8e0d0;
    --code-bg:   #1a1a1a;
    --code-fg:   #e8d5a3;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--paper);
    color: var(--ink);
    font-family: 'IBM Plex Sans', sans-serif;
    font-weight: 300;
    line-height: 1.7;
  }

  /* ── MASTHEAD ── */
  .masthead {
    border-bottom: 3px double var(--ink);
    padding: 0;
    position: relative;
    overflow: hidden;
  }
  .masthead-inner {
    max-width: 960px;
    margin: 0 auto;
    padding: 56px 32px 40px;
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 32px;
    align-items: end;
  }
  .masthead-kicker {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: var(--accent);
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 12px;
  }
  .masthead-kicker::before {
    content: '';
    display: block;
    width: 28px;
    height: 1px;
    background: var(--accent);
  }
  .masthead h1 {
    font-family: 'Playfair Display', serif;
    font-size: clamp(2.2rem, 5vw, 3.6rem);
    font-weight: 900;
    line-height: 1.1;
    letter-spacing: -0.02em;
  }
  .masthead h1 em {
    font-style: italic;
    color: var(--accent);
  }
  .masthead-sub {
    margin-top: 14px;
    font-size: 1rem;
    color: var(--mid);
    max-width: 520px;
  }
  .badge-stack {
    display: flex;
    flex-direction: column;
    gap: 8px;
    align-items: flex-end;
  }
  .badge {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    padding: 4px 10px;
    border: 1px solid var(--ink);
    white-space: nowrap;
  }
  .badge.accent { background: var(--accent); color: #fff; border-color: var(--accent); }
  .badge.gold   { background: var(--gold);   color: #fff; border-color: var(--gold);   }

  /* ── RULE STRIP ── */
  .rule-strip {
    border-top: 1px solid var(--rule);
    border-bottom: 1px solid var(--rule);
    background: var(--ink);
    color: var(--paper);
    padding: 10px 32px;
  }
  .rule-strip-inner {
    max-width: 960px;
    margin: 0 auto;
    display: flex;
    flex-wrap: wrap;
    gap: 0;
  }
  .strip-item {
    flex: 1 1 140px;
    text-align: center;
    padding: 6px 12px;
    border-right: 1px solid #333;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.68rem;
    letter-spacing: 0.06em;
    text-transform: uppercase;
  }
  .strip-item:last-child { border-right: none; }
  .strip-item strong {
    display: block;
    font-size: 1.1rem;
    font-weight: 500;
    font-family: 'Playfair Display', serif;
    letter-spacing: 0;
    color: var(--code-fg);
  }

  /* ── MAIN CONTENT ── */
  .content {
    max-width: 960px;
    margin: 0 auto;
    padding: 0 32px 80px;
  }

  /* ── SECTION HEADERS ── */
  .section {
    padding-top: 52px;
  }
  .section-header {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-bottom: 28px;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--ink);
  }
  .section-num {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    color: var(--accent);
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }
  .section-title {
    font-family: 'Playfair Display', serif;
    font-size: 1.5rem;
    font-weight: 700;
  }

  /* ── OVERVIEW GRID ── */
  .overview-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1px;
    background: var(--rule);
    border: 1px solid var(--rule);
    margin-bottom: 32px;
  }
  .overview-card {
    background: var(--paper);
    padding: 22px 20px;
  }
  .overview-card .label {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.6rem;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: var(--mid);
    margin-bottom: 6px;
  }
  .overview-card .value {
    font-family: 'Playfair Display', serif;
    font-size: 1.1rem;
    font-weight: 700;
    line-height: 1.3;
  }

  /* ── PIPELINE ── */
  .pipeline {
    display: flex;
    flex-direction: column;
    gap: 0;
    position: relative;
    padding-left: 28px;
    border-left: 2px solid var(--ink);
    margin: 8px 0 32px;
  }
  .pipeline-step {
    padding: 16px 20px 16px 24px;
    position: relative;
    border-bottom: 1px solid var(--rule);
  }
  .pipeline-step:last-child { border-bottom: none; }
  .pipeline-step::before {
    content: '';
    position: absolute;
    left: -29px;
    top: 22px;
    width: 10px;
    height: 10px;
    background: var(--ink);
    border-radius: 50%;
  }
  .pipeline-step.accent-step::before { background: var(--accent); }
  .step-title {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.75rem;
    font-weight: 500;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: var(--accent);
    margin-bottom: 4px;
  }
  .step-body {
    font-size: 0.9rem;
    color: var(--mid);
    line-height: 1.6;
  }

  /* ── METRICS TABLE ── */
  .metrics-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 1px;
    background: var(--ink);
    border: 2px solid var(--ink);
    margin: 24px 0;
  }
  .metric-cell {
    background: var(--paper);
    padding: 20px 16px;
    text-align: center;
  }
  .metric-cell .m-label {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.6rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--mid);
    display: block;
    margin-bottom: 8px;
  }
  .metric-cell .m-value {
    font-family: 'Playfair Display', serif;
    font-size: 1.6rem;
    font-weight: 700;
    line-height: 1;
    color: var(--ink);
  }
  .metric-cell .m-value.good { color: #2a7a3b; }
  .metric-cell .m-value.warn { color: var(--accent); }

  /* ── FEATURE TABLE ── */
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.875rem;
    margin: 20px 0;
  }
  th {
    background: var(--ink);
    color: var(--paper);
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.62rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    padding: 10px 14px;
    text-align: left;
    font-weight: 500;
  }
  td {
    padding: 10px 14px;
    border-bottom: 1px solid var(--rule);
    vertical-align: top;
    line-height: 1.5;
  }
  tr:last-child td { border-bottom: none; }
  tr:hover td { background: var(--tag-bg); }
  code {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.8em;
    background: var(--tag-bg);
    padding: 1px 5px;
    border-radius: 2px;
  }

  /* ── IV CHART ── */
  .iv-bar-list {
    display: flex;
    flex-direction: column;
    gap: 6px;
    margin: 20px 0;
  }
  .iv-row {
    display: grid;
    grid-template-columns: 180px 1fr 50px;
    align-items: center;
    gap: 10px;
    font-size: 0.8rem;
  }
  .iv-name {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.68rem;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
    color: var(--mid);
  }
  .iv-track {
    height: 10px;
    background: var(--rule);
    border-radius: 1px;
    overflow: hidden;
  }
  .iv-fill {
    height: 100%;
    border-radius: 1px;
    transition: width 0.6s ease;
  }
  .iv-val {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    color: var(--mid);
    text-align: right;
  }

  /* ── TAGS ── */
  .tag-list { display: flex; flex-wrap: wrap; gap: 8px; margin: 16px 0; }
  .tag {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.06em;
    text-transform: uppercase;
    padding: 4px 10px;
    background: var(--tag-bg);
    border: 1px solid var(--rule);
  }

  /* ── CALLOUT ── */
  .callout {
    border-left: 3px solid var(--accent);
    background: var(--tag-bg);
    padding: 16px 20px;
    margin: 24px 0;
    font-size: 0.9rem;
  }
  .callout strong {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: var(--accent);
    display: block;
    margin-bottom: 6px;
  }

  /* ── TWO-COL PROSE ── */
  .two-col {
    columns: 2 400px;
    column-gap: 40px;
    font-size: 0.92rem;
    line-height: 1.75;
    color: var(--mid);
    margin-bottom: 24px;
  }

  /* ── FOOTER ── */
  footer {
    border-top: 3px double var(--ink);
    max-width: 960px;
    margin: 0 auto;
    padding: 24px 32px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 12px;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--mid);
  }

  @media (max-width: 640px) {
    .masthead-inner { grid-template-columns: 1fr; }
    .badge-stack { align-items: flex-start; flex-direction: row; flex-wrap: wrap; }
    .iv-row { grid-template-columns: 120px 1fr 44px; }
    footer { flex-direction: column; }
  }
</style>
</head>
<body>

<!-- ══════════════════ MASTHEAD ══════════════════ -->
<header class="masthead">
  <div class="masthead-inner">
    <div>
      <div class="masthead-kicker">FNB Data Quest Project · UCT</div>
      <h1>Building <em>Interpretable</em><br>Credit Models</h1>
      <p class="masthead-sub">
        A logistic regression credit-default prediction pipeline combining WoE transformation,
        Information Value feature selection, and Elastic Net regularisation — engineered for
        transparency in a regulated lending environment.
      </p>
    </div>
    <div class="badge-stack">
      <div class="badge accent">R · Statistical Modelling</div>
      <div class="badge gold">AUC 80.21%</div>
      <div class="badge">Gini 60.43%</div>
      <div class="badge">120 k observations</div>
    </div>
  </div>
</header>

<!-- ══════════════════ STRIP ══════════════════ -->
<div class="rule-strip">
  <div class="rule-strip-inner">
    <div class="strip-item"><strong>120 358</strong>Records (clean)</div>
    <div class="strip-item"><strong>26</strong>Original variables</div>
    <div class="strip-item"><strong>35+</strong>Engineered features</div>
    <div class="strip-item"><strong>80.21 %</strong>AUC</div>
    <div class="strip-item"><strong>60.43 %</strong>Gini</div>
    <div class="strip-item"><strong>73.1 %</strong>Bal. Accuracy</div>
  </div>
</div>

<!-- ══════════════════ CONTENT ══════════════════ -->
<main class="content">

  <!-- AUTHOR -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">00 — Author</span>
      <span class="section-title">Project Details</span>
    </div>
    <div class="overview-grid">
      <div class="overview-card"><div class="label">Student</div><div class="value">Sbonokuhle Myeni</div></div>
      <div class="overview-card"><div class="label">Student Number</div><div class="value">MYNSBO001</div></div>
      <div class="overview-card"><div class="label">Institution</div><div class="value">University of Cape Town</div></div>
      <div class="overview-card"><div class="label">Email</div><div class="value" style="font-size:0.85rem">mynsbo001@myuct.ac.za</div></div>
      <div class="overview-card"><div class="label">Industry Partner</div><div class="value">FNB — First National Bank</div></div>
      <div class="overview-card"><div class="label">Competition</div><div class="value">DataQuest 2026</div></div>
    </div>
  </section>

  <!-- OVERVIEW -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">01 — Overview</span>
      <span class="section-title">Project Summary</span>
    </div>
    <div class="two-col">
      Credit risk modelling assists lending institutions in evaluating the likelihood that a borrower will default on a loan. In regulated banking environments, predictive accuracy must be balanced against model interpretability — lending decisions must remain transparent and explainable to satisfy regulatory scrutiny and enable adverse-action notices.
      <br><br>
      This project developed an interpretable credit-default prediction model using a simulated FNB loan application dataset of over 120 000 observations. The study explored the trade-off between interpretability and performance by applying logistic regression enhanced with Weight of Evidence (WoE) encoding, Information Value (IV) feature selection, and Elastic Net regularisation. Extensive EDA and data preprocessing were carried out to handle missing values, outliers, inconsistent labels, and class imbalance before model fitting.
    </div>
    <div class="tag-list">
      <span class="tag">Logistic Regression</span>
      <span class="tag">Weight of Evidence</span>
      <span class="tag">Information Value</span>
      <span class="tag">Elastic Net</span>
      <span class="tag">Feature Engineering</span>
      <span class="tag">Class Imbalance</span>
      <span class="tag">Credit Risk</span>
      <span class="tag">Regulatory Compliance</span>
    </div>
  </section>

  <!-- PIPELINE -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">02 — Methodology</span>
      <span class="section-title">Modelling Pipeline</span>
    </div>
    <div class="pipeline">
      <div class="pipeline-step">
        <div class="step-title">Step 1 — Data Ingestion & Cleaning</div>
        <div class="step-body">
          Loaded 120 960 records across 26 variables. Removed the <code>applicant_id_hash</code> identifier column and 602 duplicate records. Standardised inconsistent categorical labels (e.g. <code>RENT / RENTING</code>) by uppercasing and stripping special characters. Unified four mixed date formats into <code>YYYY-MM-DD</code>.
        </div>
      </div>
      <div class="pipeline-step">
        <div class="step-title">Step 2 — Missing Value Treatment</div>
        <div class="step-body">
          <code>annual_income</code> (7.18%), <code>employment_length_years</code> (3.08%), and <code>num_open_accounts</code> (2.01%) were imputed with median values. <code>months_since_last_delinquency</code> (49.91% missing) was treated as MNAR — a binary missingness indicator was created rather than imputing directly.
        </div>
      </div>
      <div class="pipeline-step">
        <div class="step-title">Step 3 — Outlier Removal</div>
        <div class="step-body">
          Tukey fence method (1.5 × IQR) applied to <code>annual_income</code> and <code>loan_amount</code> — the two variables with extreme right tails. This reduced the dataset by approximately 8%, yielding 110 729 records for modelling.
        </div>
      </div>
      <div class="pipeline-step">
        <div class="step-title">Step 4 — Exploratory Data Analysis</div>
        <div class="step-body">
          Univariate, bivariate, and multivariate analyses performed. Key findings: <code>credit_utilisation_pct</code>, <code>interest_rate</code>, <code>dti_ratio</code>, <code>num_delinquencies_2yr</code>, and <code>num_hard_inquiries_6mo</code> showed the clearest separation between defaulters and non-defaulters. Correlation heatmap confirmed limited multicollinearity, except for the strong <code>age</code> ↔ <code>months_since_oldest_account</code> pair (r = 0.95).
        </div>
      </div>
      <div class="pipeline-step accent-step">
        <div class="step-title">Step 5 — Feature Engineering</div>
        <div class="step-body">
          35+ features derived including log-transformed financial variables, ratio-based risk indicators (<code>revolving_to_income</code>, <code>income_per_emp_yr</code>), interaction terms (<code>util_x_dti</code>, <code>interest_burden</code>), ordinal risk bands (<code>delinquency_band</code>, <code>utilisation_band</code>), and a composite <code>exposure_score</code>.
        </div>
      </div>
      <div class="pipeline-step accent-step">
        <div class="step-title">Step 6 — IV Screening & WoE Encoding</div>
        <div class="step-body">
          Information Value computed on the training set only. Features with IV &lt; 0.02 dropped. All retained features encoded via Weight of Evidence binning — mapping each variable onto a common log-odds scale that enforces monotone score functions.
        </div>
      </div>
      <div class="pipeline-step accent-step">
        <div class="step-title">Step 7 — Elastic Net Logistic Regression</div>
        <div class="step-body">
          Elastic Net regularisation (L1 + L2) applied to the WoE-encoded feature matrix. Optimal λ = 0.001443 identified via 10-fold cross-validation. Class weights applied to handle the 84 % / 16 % non-default / default imbalance. Optimal classification threshold (0.4876) selected by maximising Youden's J statistic.
        </div>
      </div>
    </div>
  </section>

  <!-- FEATURES -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">03 — Features</span>
      <span class="section-title">Top Features by Information Value</span>
    </div>
    <p style="font-size:0.88rem;color:var(--mid);margin-bottom:20px;">IV computed on training data only to prevent leakage. Features with IV &lt; 0.02 excluded from modelling.</p>

    <div class="iv-bar-list">
      <!-- suspicious (red) -->
      <div class="iv-row"><span class="iv-name">repayment_stability</span><div class="iv-track"><div class="iv-fill" style="width:93%;background:#c8422a"></div></div><span class="iv-val">0.578</span></div>
      <div class="iv-row"><span class="iv-name">interest_rate</span><div class="iv-track"><div class="iv-fill" style="width:86%;background:#c8422a"></div></div><span class="iv-val">0.534</span></div>
      <div class="iv-row"><span class="iv-name">log_loan_amount</span><div class="iv-track"><div class="iv-fill" style="width:85%;background:#c8422a"></div></div><span class="iv-val">0.526</span></div>
      <!-- strong (green) -->
      <div class="iv-row"><span class="iv-name">exposure_score</span><div class="iv-track"><div class="iv-fill" style="width:76%;background:#2a7a3b"></div></div><span class="iv-val">0.470</span></div>
      <div class="iv-row"><span class="iv-name">total_revolving_balance</span><div class="iv-track"><div class="iv-fill" style="width:72%;background:#2a7a3b"></div></div><span class="iv-val">0.444</span></div>
      <div class="iv-row"><span class="iv-name">risk_score_raw</span><div class="iv-track"><div class="iv-fill" style="width:65%;background:#2a7a3b"></div></div><span class="iv-val">0.399</span></div>
      <div class="iv-row"><span class="iv-name">ever_delinquent</span><div class="iv-track"><div class="iv-fill" style="width:57%;background:#2a7a3b"></div></div><span class="iv-val">0.349</span></div>
      <div class="iv-row"><span class="iv-name">num_delinquencies_2yr</span><div class="iv-track"><div class="iv-fill" style="width:56%;background:#2a7a3b"></div></div><span class="iv-val">0.343</span></div>
      <!-- medium (blue) -->
      <div class="iv-row"><span class="iv-name">months_since_oldest_account</span><div class="iv-track"><div class="iv-fill" style="width:46%;background:#2a5a8a"></div></div><span class="iv-val">0.283</span></div>
      <div class="iv-row"><span class="iv-name">util_x_dti</span><div class="iv-track"><div class="iv-fill" style="width:44%;background:#2a5a8a"></div></div><span class="iv-val">0.274</span></div>
      <div class="iv-row"><span class="iv-name">annual_income</span><div class="iv-track"><div class="iv-fill" style="width:36%;background:#2a5a8a"></div></div><span class="iv-val">0.221</span></div>
      <div class="iv-row"><span class="iv-name">credit_utilisation_pct</span><div class="iv-track"><div class="iv-fill" style="width:25%;background:#2a5a8a"></div></div><span class="iv-val">0.157</span></div>
      <div class="iv-row"><span class="iv-name">dti_ratio</span><div class="iv-track"><div class="iv-fill" style="width:21%;background:#2a5a8a"></div></div><span class="iv-val">0.130</span></div>
    </div>

    <div class="callout">
      <strong>IV thresholds</strong>
      Features rated: &lt;0.02 Useless · 0.02–0.1 Weak · 0.1–0.3 Medium · 0.3–0.5 Strong · &gt;0.5 Suspicious (possible data leakage). The three suspicious features were retained but flagged for further validation.
    </div>
  </section>

  <!-- RESULTS -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">04 — Results</span>
      <span class="section-title">Model Performance</span>
    </div>
    <div class="metrics-grid">
      <div class="metric-cell"><span class="m-label">AUC</span><span class="m-value good">80.21%</span></div>
      <div class="metric-cell"><span class="m-label">Gini</span><span class="m-value good">60.43%</span></div>
      <div class="metric-cell"><span class="m-label">Balanced Accuracy</span><span class="m-value good">72.88%</span></div>
      <div class="metric-cell"><span class="m-label">Sensitivity / Recall</span><span class="m-value good">74.26%</span></div>
      <div class="metric-cell"><span class="m-label">Specificity</span><span class="m-value good">71.49%</span></div>
      <div class="metric-cell"><span class="m-label">Accuracy</span><span class="m-value">71.94%</span></div>
      <div class="metric-cell"><span class="m-label">Precision (PPV)</span><span class="m-value warn">33.38%</span></div>
      <div class="metric-cell"><span class="m-label">F1 Score</span><span class="m-value warn">46.06%</span></div>
    </div>

    <div class="callout">
      <strong>Interpretation</strong>
      The model achieves strong rank-ordering performance (AUC &gt; 0.80, Gini &gt; 0.60), meeting the industry benchmark for credit scorecards. Sensitivity and specificity are well balanced at the optimal threshold (0.4876, selected via Youden's J). The lower precision reflects the class imbalance — 84% of the population are non-defaulters — and is expected in this setting. Further threshold calibration or probability re-weighting could improve precision if business requirements demand fewer false positives.
    </div>
  </section>

  <!-- REGULATORY -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">05 — Governance</span>
      <span class="section-title">Regulatory Considerations</span>
    </div>
    <table>
      <thead>
        <tr><th>Feature</th><th>Regulatory Risk</th><th>Jurisdiction</th></tr>
      </thead>
      <tbody>
        <tr>
          <td><code>age</code></td>
          <td>Age discrimination laws prohibit using age as a primary credit factor for applicants over 40</td>
          <td>US ECOA · SA NCA</td>
        </tr>
        <tr>
          <td><code>region</code></td>
          <td>Regional proxies can correlate with race or socio-economic status, constituting indirect discrimination (redlining)</td>
          <td>US FHA · UK FCA · SA FSCA</td>
        </tr>
        <tr>
          <td><code>phone_verified</code></td>
          <td>Digital-divide proxy — unverified phones correlate with lower income demographics; potential disparate impact</td>
          <td>US CFPB · UK FCA</td>
        </tr>
        <tr>
          <td><code>email_domain_type</code></td>
          <td>Free vs corporate email may proxy income or occupation — possible protected-class proxy</td>
          <td>US CFPB · UK FCA</td>
        </tr>
      </tbody>
    </table>
    <p style="font-size:0.85rem;color:var(--mid);">All model features should be subject to a <strong>Disparate Impact Analysis</strong> before production deployment to ensure no disproportionate effect on protected classes, even where discrimination is unintentional.</p>
  </section>

  <!-- STRUCTURE -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">06 — Structure</span>
      <span class="section-title">Report Structure</span>
    </div>
    <table>
      <thead><tr><th>Section</th><th>Contents</th></tr></thead>
      <tbody>
        <tr><td><strong>Part 1 — Research</strong></td><td>GLMs vs non-linear models, WoE &amp; IV theory, key metrics (AUC, Gini, F1), regulatory considerations</td></tr>
        <tr><td><strong>Part 2 — EDA</strong></td><td>Univariate, bivariate and multivariate analysis; correlation heatmap; missing-value and outlier analysis</td></tr>
        <tr><td><strong>Data Cleaning</strong></td><td>Deduplication, imputation, Tukey-fence outlier removal, categorical standardisation</td></tr>
        <tr><td><strong>Feature Engineering</strong></td><td>Log transforms, risk flags, ratio features, interaction terms, utilisation bands</td></tr>
        <tr><td><strong>Modelling</strong></td><td>IV screening, WoE encoding, Elastic Net logistic regression, threshold optimisation via Youden's J</td></tr>
        <tr><td><strong>Conclusion</strong></td><td>Summary of findings, model limitations, recommendations for improvement</td></tr>
      </tbody>
    </table>
  </section>

  <!-- TECH -->
  <section class="section">
    <div class="section-header">
      <span class="section-num">07 — Stack</span>
      <span class="section-title">Tools &amp; Technologies</span>
    </div>
    <div class="tag-list">
      <span class="tag">R</span>
      <span class="tag">tidyverse</span>
      <span class="tag">glmnet</span>
      <span class="tag">scorecard</span>
      <span class="tag">ggplot2</span>
      <span class="tag">caret</span>
      <span class="tag">pROC</span>
      <span class="tag">lubridate</span>
      <span class="tag">dplyr</span>
      <span class="tag">WoE Encoding</span>
      <span class="tag">Elastic Net</span>
      <span class="tag">10-Fold CV</span>
    </div>
  </section>

</main>

<footer>
  <span>FNB DataQuest · Building Interpretable Credit Models</span>
  <span>Sbonokuhle Myeni · MYNSBO001 · UCT</span>
  <span>AUC 80.21 % · Gini 60.43 %</span>
</footer>

</body>
</html>