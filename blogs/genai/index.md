---
layout: page
title: Generative AI
permalink: /blogs/genai/
---

<style>
  .idx-topic {
    --topic-heading: #00695c;
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
    --a1: #00a896;
    --a2: #00796b;
    --g: 0, 168, 150;
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
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
  .idx-thumb-wrap { width: 98px; height: 98px; margin-bottom: 14px; border-radius: 14px; border: 1px solid rgba(255,255,255,0.42); background: rgba(255,255,255,0.18); }
  .idx-thumb { width: 100%; height: 100%; object-fit: cover; border-radius: 13px; display: block; }
  .idx-title { font-size: 1.19rem; text-align: center; line-height: 1.28; }
  .idx-sub { margin-top: 6px; text-align: center; font-size: .94rem; opacity: .94; line-height: 1.4; }
  .idx-cta { margin-top: auto; padding-top: 11px; font-size: .88rem; font-weight: 700; letter-spacing: .35px; text-transform: uppercase; }
  @media (max-width: 1100px) { .idx-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
  @media (max-width: 760px) { .idx-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 10px; } }
  @media (max-width: 480px) { .idx-grid { grid-template-columns: 1fr; } }
</style>

<section class="idx-topic">
  <h2 class="idx-topic-title">Generative AI</h2>
  <div class="idx-grid">

    <a href="/blogs/llms/" class="idx-card" style="--a1:#00a896; --a2:#00796b; --g:0,168,150;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/softmax.JPG" alt="LLMs" class="idx-thumb">
      </span>
      <strong class="idx-title">LLMs</strong>
      <span class="idx-sub">GPT, Language Models</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/foundationmodels/" class="idx-card" style="--a1:#00acc1; --a2:#00838f; --g:0,172,193;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/BERT.JPG" alt="Foundation Models" class="idx-thumb">
      </span>
      <strong class="idx-title">Foundation Models</strong>
      <span class="idx-sub">GPT-1, BERT, T5</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/rag/" class="idx-card" style="--a1:#00897b; --a2:#00574b; --g:0,137,123;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/RAG.JPG" alt="RAG" class="idx-thumb">
      </span>
      <strong class="idx-title">RAG</strong>
      <span class="idx-sub">Retrieval-Augmented Generation</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/diffusion/" class="idx-card" style="--a1:#26a69a; --a2:#00796b; --g:38,166,154;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/DiffusionTraning.JPG" alt="Diffusion" class="idx-thumb">
      </span>
      <strong class="idx-title">Diffusion Models</strong>
      <span class="idx-sub">Image generation</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/gan/" class="idx-card" style="--a1:#43a047; --a2:#1b5e20; --g:67,160,71;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/gan.JPG" alt="GAN" class="idx-thumb">
      </span>
      <strong class="idx-title">GAN</strong>
      <span class="idx-sub">Generative Adversarial Networks</span>
      <span class="idx-cta">Read article -></span>
    </a>

  </div>
</section>
