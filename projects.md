---
layout: page
title: Projects
permalink: /projects/
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
		width: 100%;
		min-height: 210px;
		padding: 17px 14px 14px;
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
	.idx-sub { margin-top: 6px; text-align: center; font-size: .94rem; opacity: .94; line-height: 1.4; }
	.idx-stack {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 6px;
		margin-top: 10px;
	}
	.idx-stack span {
		font-size: 0.74rem;
		font-weight: 700;
		padding: 5px 8px;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.2);
		border: 1px solid rgba(255, 255, 255, 0.3);
		color: #ffffff;
	}
	.idx-cta { margin-top: auto; padding-top: 11px; font-size: .88rem; font-weight: 700; letter-spacing: .35px; text-transform: uppercase; }
	@media (max-width: 1100px) { .idx-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
	@media (max-width: 760px) { .idx-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 10px; } }
	@media (max-width: 480px) { .idx-grid { grid-template-columns: 1fr; } }
</style>

<section class="idx-topic">
	<h2 class="idx-topic-title">Featured Projects</h2>

	<div class="idx-grid">
		<a class="idx-card" style="--a1:#00897b; --a2:#00695c; --g:0,137,123;" href="https://github.com/abhaysrivastav/cvml/tree/main/RAG-LangChain-Gemini" target="_blank" rel="noopener noreferrer">
			<span class="idx-thumb-wrap">
				<img src="/blogs/assests/RAG.JPG" alt="RAG with LangChain and Gemini" class="idx-thumb">
			</span>
			<strong class="idx-title">RAG with LangChain and Gemini</strong>
			<span class="idx-sub">LLM app with Gemini, prompt tuning, and retrieval-augmented generation over PDF content.</span>
			<div class="idx-stack">
				<span>LangChain</span><span>Gemini</span><span>RAG</span><span>Chroma</span>
			</div>
			<div class="idx-cta">Open project folder</div>
		</a>

		<a class="idx-card" style="--a1:#8e24aa; --a2:#5e35b1; --g:142,36,170;" href="https://github.com/abhaysrivastav/cvml/tree/main/image-captioning-project" target="_blank" rel="noopener noreferrer">
			<span class="idx-thumb-wrap">
				<img src="/blogs/assests/encoderdecoder.jpg" alt="Image Captioning Project" class="idx-thumb">
			</span>
			<strong class="idx-title">Image Captioning Project</strong>
			<span class="idx-sub">Automatic image caption generation with dataset preprocessing, CNN-RNN training, and inference notebooks.</span>
			<div class="idx-stack">
				<span>PyTorch</span><span>CNN-RNN</span><span>MS COCO</span><span>NLP</span>
			</div>
			<div class="idx-cta">Open project folder</div>
		</a>
	</div>
</section>
