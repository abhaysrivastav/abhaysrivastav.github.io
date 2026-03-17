---
layout: page
title: Projects
permalink: /projects/
---

<style>
	.project-board {
		margin: 6px auto 30px;
	}

	.project-title {
		margin: 0 0 22px;
		text-align: center;
		color: #0f4c81;
		font-size: clamp(1.9rem, 2.8vw, 2.6rem);
		font-weight: 800;
		letter-spacing: 0.2px;
	}

	.project-grid {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: 14px;
	}

	.project-card {
		display: flex;
		flex-direction: column;
		min-height: 280px;
		padding: 18px 16px;
		border-radius: 16px;
		color: #ffffff !important;
		text-decoration: none;
		border: 1px solid rgba(255, 255, 255, 0.24);
		background: linear-gradient(150deg, var(--c1, #1565c0), var(--c2, #1e88e5));
		box-shadow: 0 10px 24px rgba(10, 35, 64, 0.24);
		transition: transform 0.2s ease, box-shadow 0.2s ease;
	}

	.project-card:hover,
	.project-card:focus-visible {
		transform: translateY(-3px);
		box-shadow: 0 14px 28px rgba(10, 35, 64, 0.33);
	}

	.project-card h3 {
		margin: 0 0 8px;
		color: #ffffff;
		font-size: 1.15rem;
		line-height: 1.3;
	}

	.project-card p {
		margin: 0;
		color: rgba(255, 255, 255, 0.95);
		font-size: 0.93rem;
		line-height: 1.45;
	}

	.project-stack {
		display: flex;
		flex-wrap: wrap;
		gap: 6px;
		margin-top: 12px;
	}

	.project-stack span {
		font-size: 0.74rem;
		font-weight: 700;
		padding: 5px 8px;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.2);
		border: 1px solid rgba(255, 255, 255, 0.3);
		color: #ffffff;
	}

	.project-link {
		margin-top: auto;
		padding-top: 14px;
		font-size: 0.82rem;
		font-weight: 700;
		letter-spacing: 0.4px;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.95);
	}

	@media (max-width: 1050px) {
		.project-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
	}

	@media (max-width: 640px) {
		.project-grid { grid-template-columns: 1fr; }
	}
</style>

<section class="project-board">
	<h2 class="project-title">Featured Projects</h2>

	<div class="project-grid">
		<a class="project-card" style="--c1:#1565c0; --c2:#0d47a1;" href="https://github.com/abhaysrivastav/cvml" target="_blank" rel="noopener noreferrer">
			<h3>CVML Repository</h3>
			<p>Computer Vision and Machine Learning repository with practical notebooks and end-to-end project work across deep learning and LLM workflows.</p>
			<div class="project-stack">
				<span>Python</span><span>Jupyter</span><span>ML</span><span>Computer Vision</span>
			</div>
			<div class="project-link">Open repository</div>
		</a>

		<a class="project-card" style="--c1:#00897b; --c2:#00695c;" href="https://github.com/abhaysrivastav/cvml/tree/main/RAG-LangChain-Gemini" target="_blank" rel="noopener noreferrer">
			<h3>RAG with LangChain and Gemini</h3>
			<p>Builds an LLM app using Google Gemini and LangChain, including prompt experiments and a retrieval-augmented generation pipeline over PDF content.</p>
			<div class="project-stack">
				<span>LangChain</span><span>Gemini</span><span>RAG</span><span>Chroma</span>
			</div>
			<div class="project-link">Open project folder</div>
		</a>

		<a class="project-card" style="--c1:#8e24aa; --c2:#5e35b1;" href="https://github.com/abhaysrivastav/cvml/tree/main/image-captioning-project" target="_blank" rel="noopener noreferrer">
			<h3>Image Captioning Project</h3>
			<p>Deep learning project for automatic image caption generation with dataset preprocessing, CNN-RNN training, and inference notebooks.</p>
			<div class="project-stack">
				<span>PyTorch</span><span>CNN-RNN</span><span>MS COCO</span><span>NLP</span>
			</div>
			<div class="project-link">Open project folder</div>
		</a>
	</div>
</section>
