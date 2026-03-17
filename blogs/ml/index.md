---
layout: page
title: Machine Learning
permalink: /blogs/ml/
---

<style>
  .idx-topic {
    --topic-heading: #1565c0;
    margin: 0 auto;
    padding: 8px 0 34px;
  }
  .idx-topic-title {
    margin: 0 0 24px;
    text-align: center;
    color: var(--topic-heading);
    font-size: clamp(1.8rem, 2.6vw, 2.5rem);
    font-weight: 800;
    letter-spacing: 0.3px;
  }
  .idx-grid {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 22px;
    align-items: stretch;
  }
  .idx-card {
    --a1: #1e88e5;
    --a2: #1565c0;
    --g: 30, 136, 229;
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 250px;
    padding: 18px 16px 16px;
    border-radius: 18px;
    text-decoration: none;
    color: #fff !important;
    border: 1px solid rgba(255, 255, 255, 0.28);
    background: radial-gradient(circle at 12% 8%, rgba(255,255,255,0.28), rgba(255,255,255,0) 42%), linear-gradient(150deg, var(--a1), var(--a2));
    box-shadow: 0 10px 22px rgba(var(--g), 0.3), inset 0 1px 0 rgba(255,255,255,0.2);
    transition: transform .25s ease, box-shadow .25s ease, filter .25s ease;
  }
  .idx-card:hover,
  .idx-card:focus-visible {
    transform: translateY(-4px);
    box-shadow: 0 14px 28px rgba(var(--g), 0.38), inset 0 1px 0 rgba(255,255,255,0.25);
    filter: saturate(1.08);
  }
  .idx-thumb-wrap {
    width: 84px;
    height: 84px;
    margin-bottom: 12px;
    border-radius: 14px;
    border: 1px solid rgba(255,255,255,0.42);
    background: rgba(255,255,255,0.18);
  }
  .idx-thumb {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 13px;
    display: block;
  }
  .idx-title { font-size: 1.03rem; text-align: center; line-height: 1.32; }
  .idx-sub { margin-top: 6px; text-align: center; font-size: .84rem; opacity: .92; line-height: 1.4; }
  .idx-cta { margin-top: auto; padding-top: 14px; font-size: .82rem; font-weight: 700; letter-spacing: .35px; text-transform: uppercase; }
  @media (max-width: 1100px) { .idx-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
  @media (max-width: 760px) { .idx-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 16px; } }
  @media (max-width: 480px) { .idx-grid { grid-template-columns: 1fr; } }
</style>

<section class="idx-topic">
  <h2 class="idx-topic-title">Machine Learning</h2>
  <div class="idx-grid">

    <a href="/blogs/aimlazure/" class="idx-card" style="--a1:#1e88e5; --a2:#1565c0; --g:30,136,229;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/BERT.JPG" alt="Azure AI/ML" class="idx-thumb">
      </span>
      <strong class="idx-title">Azure AI/ML</strong>
      <span class="idx-sub">Azure ML Infrastructure</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/nlpconcepts/" class="idx-card" style="--a1:#0288d1; --a2:#0277bd; --g:2,136,209;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/word2vec.jpg" alt="NLP Concepts" class="idx-thumb">
      </span>
      <strong class="idx-title">NLP Concepts</strong>
      <span class="idx-sub">Tokenization, Embeddings</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/decision-tree-blog/" class="idx-card" style="--a1:#1976d2; --a2:#0d47a1; --g:25,118,210;">
      <span class="idx-thumb-wrap">
        <img src="/assets/ml-images/alice-crossroads.png" alt="Decision Tree Blog" class="idx-thumb">
      </span>
      <strong class="idx-title">Decision Trees</strong>
      <span class="idx-sub">Splits, Gini, Pruning</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/random-forest-blog/" class="idx-card" style="--a1:#00a896; --a2:#00695c; --g:0,168,150;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/ml-img/fig1_random_forest_ensemble.png" alt="Random Forest Blog" class="idx-thumb">
      </span>
      <strong class="idx-title">Random Forest</strong>
      <span class="idx-sub">Bagging, OOB, Importance</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/linear-regression-blog/" class="idx-card" style="--a1:#8e24aa; --a2:#6a1b9a; --g:142,36,170;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/ml-img/fig1_linear_regression_line.png" alt="Linear Regression Blog" class="idx-thumb">
      </span>
      <strong class="idx-title">Linear Regression</strong>
      <span class="idx-sub">MSE, GD, Assumptions</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/k-means-clustering/" class="idx-card" style="--a1:#ff9800; --a2:#ef6c00; --g:255,152,0;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/ml-img/01_intuition.png" alt="K-Means Clustering Blog" class="idx-thumb">
      </span>
      <strong class="idx-title">K-Means Clustering</strong>
      <span class="idx-sub">WCSS, Elbow, K-Means++</span>
      <span class="idx-cta">Read article -></span>
    </a>

  </div>
</section>
