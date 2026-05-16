---
layout: page
title: AWS Labs
permalink: /blogs/aws/
---

<style>
  .idx-topic {
    --topic-heading: #ef6c00;
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
    gap: 10px;
    justify-items: center;
    align-items: stretch;
  }
  .idx-card {
    --a1: #f57c00;
    --a2: #ef6c00;
    --g: 245, 124, 0;
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: calc(100% - 10px);
    max-width: 260px;
    min-height: 210px;
    padding: 17px 14px 14px;
    border-radius: 18px;
    text-decoration: none;
    color: #fff !important;
    border: 1px solid rgba(255,255,255,0.28);
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
    width: 98px;
    height: 98px;
    margin-bottom: 14px;
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
  .idx-title { font-size: 1.19rem; text-align: center; line-height: 1.28; }
  .idx-sub {
    margin-top: 6px;
    text-align: center;
    font-size: .94rem;
    opacity: .94;
    line-height: 1.4;
  }
  .idx-cta {
    margin-top: auto;
    padding-top: 11px;
    font-size: .88rem;
    font-weight: 700;
    letter-spacing: .35px;
    text-transform: uppercase;
  }
  @media (max-width: 1100px) { .idx-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
  @media (max-width: 760px) { .idx-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 10px; } }
  @media (max-width: 480px) {
    .idx-grid { grid-template-columns: 1fr; }
    .idx-card { width: 100%; max-width: 100%; }
  }
</style>

<section class="idx-topic">
  <h2 class="idx-topic-title">AWS Labs</h2>
  <div class="idx-grid">

    <a href="/blogs/aws/rag-bedrock-lab/" class="idx-card" style="--a1:#f57c00; --a2:#ef6c00; --g:245,124,0;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/aws-img/lab1/arch.png" alt="AWS Lab 1" class="idx-thumb">
      </span>
      <strong class="idx-title">Lab 1</strong>
      <span class="idx-sub">RAG with Bedrock and Aurora PostgreSQL</span>
      <span class="idx-cta">Open lab -></span>
    </a>

    <a href="/blogs/aws/aws-rds-lab/" class="idx-card" style="--a1:#fb8c00; --a2:#f57c00; --g:251,140,0;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/aws-img/lab2/image1.png" alt="AWS Lab 2" class="idx-thumb">
      </span>
      <strong class="idx-title">Lab 2</strong>
      <span class="idx-sub">Working with Relational Databases using AWS RDS</span>
      <span class="idx-cta">Open lab -></span>
    </a>

    <a href="/blogs/aws/mlflow-experiment/" class="idx-card" style="--a1:#6a1b9a; --a2:#8e24aa; --g:106,27,154;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/aws-img/lab3/mlflow-models.jpg" alt="AWS Lab 3 - MLflow" class="idx-thumb">
      </span>
      <strong class="idx-title">Lab 3</strong>
      <span class="idx-sub">Tracking, versioning and serving models with MLflow</span>
      <span class="idx-cta">Open lab -></span>
    </a>

  </div>
</section>

<section class="idx-topic">
  <div class="idx-grid">

    <a href="/blogs/aws/crewai-bedrock-lab/" class="idx-card" style="--a1:#00838f; --a2:#006064; --g:0,131,143;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/aws-img/lab4/screenshot_35.jpg" alt="CrewAI Bedrock Lab" class="idx-thumb">
      </span>
      <strong class="idx-title">Lab 4</strong>
      <span class="idx-sub">Building AI Agents with CrewAI and Amazon Bedrock Knowledge Base</span>
      <span class="idx-cta">Open lab -></span>
    </a>

  </div>
</section>
