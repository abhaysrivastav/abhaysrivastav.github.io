---
layout: page
title: Deep Learning
permalink: /blogs/dl/
---

<style>
  .idx-topic {
    --topic-heading: #5e35b1;
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
    --a1: #7b1fa2;
    --a2: #5e35b1;
    --g: 123, 31, 162;
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 70%;
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
  .idx-thumb-wrap { width: 78px; height: 78px; margin-bottom: 11px; border-radius: 11px; border: 1px solid rgba(255,255,255,0.42); background: rgba(255,255,255,0.18); }
  .idx-thumb { width: 100%; height: 100%; object-fit: cover; border-radius: 10px; display: block; }
  .idx-title { font-size: 0.85rem; text-align: center; line-height: 1.28; }
  .idx-sub { margin-top: 6px; text-align: center; font-size: .67rem; opacity: .94; line-height: 1.4; }
  .idx-cta { margin-top: auto; padding-top: 11px; font-size: .63rem; font-weight: 700; letter-spacing: .35px; text-transform: uppercase; }
  @media (max-width: 1100px) { .idx-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
  @media (max-width: 760px) { .idx-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 16px; } }
  @media (max-width: 480px) { .idx-grid { grid-template-columns: 1fr; } }
</style>

<section class="idx-topic">
  <h2 class="idx-topic-title">Deep Learning</h2>
  <div class="idx-grid">

    <a href="/blogs/sequence_modelling/" class="idx-card" style="--a1:#7b1fa2; --a2:#5e35b1; --g:123,31,162;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/RNN1.jpg" alt="Sequence Modelling" class="idx-thumb">
      </span>
      <strong class="idx-title">Sequence Modelling</strong>
      <span class="idx-sub">RNN, LSTM, GRU</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/transformer/" class="idx-card" style="--a1:#8e24aa; --a2:#6a1b9a; --g:142,36,170;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/transformer.JPG" alt="Transformer" class="idx-thumb">
      </span>
      <strong class="idx-title">Transformer</strong>
      <span class="idx-sub">Architecture and attention</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/transformerfamily/" class="idx-card" style="--a1:#5e35b1; --a2:#4527a0; --g:94,53,177;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/multihead.JPG" alt="Transformer Family" class="idx-thumb">
      </span>
      <strong class="idx-title">Transformer Family</strong>
      <span class="idx-sub">Vision and language models</span>
      <span class="idx-cta">Read article -></span>
    </a>

    <a href="/blogs/pretrainedmodels/" class="idx-card" style="--a1:#3949ab; --a2:#283593; --g:57,73,171;">
      <span class="idx-thumb-wrap">
        <img src="/blogs/assests/vgg16.JPG" alt="Pretrained Models" class="idx-thumb">
      </span>
      <strong class="idx-title">Pretrained Models</strong>
      <span class="idx-sub">AlexNet, VGG, ResNet</span>
      <span class="idx-cta">Read article -></span>
    </a>

  </div>
</section>
