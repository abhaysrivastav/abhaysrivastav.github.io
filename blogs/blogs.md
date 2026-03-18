---
layout: page
title: Blog
permalink: /blogs/blogs/
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
    gap: 10px;
    justify-items: center;
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
  <h2 class="idx-topic-title">Blog Topics</h2>
  <div class="idx-grid">

    <a href="/blogs/ml/" class="idx-card" style="--a1:#1e88e5; --a2:#1565c0; --g:30,136,229;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/word2vec.jpg" alt="Machine Learning" class="idx-thumb">
      </span>
      <strong class="idx-title">ML</strong>
      <span class="idx-sub">Machine Learning</span>
      <span class="idx-cta">Explore track -></span>
    </a>

    <a href="/blogs/dl/" class="idx-card" style="--a1:#8e24aa; --a2:#5e35b1; --g:142,36,170;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/RNN1.jpg" alt="Deep Learning" class="idx-thumb">
      </span>
      <strong class="idx-title">DL</strong>
      <span class="idx-sub">Deep Learning</span>
      <span class="idx-cta">Explore track -></span>
    </a>

    <a href="/blogs/genai/" class="idx-card" style="--a1:#00a896; --a2:#00695c; --g:0,168,150;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/transformer.JPG" alt="Generative AI" class="idx-thumb">
      </span>
      <strong class="idx-title">GenAI</strong>
      <span class="idx-sub">Generative AI</span>
      <span class="idx-cta">Explore track -></span>
    </a>

  </div>
</section>
