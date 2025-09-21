---
title: "Traffic Density and Episode Length-Based Curriculum Learning for Realistic Driving Simulators"
collection: publications
category: conferences
permalink: /publication/wu2025traffic
excerpt: "A new method using curriculum learning to train a self-driving car agent in a realistic simulator."
date: 2025-12-31
venue: "Under Reviewing"
# slidesurl: url
# paperurl: url
# bibtexurl: url
# code: https://github.com/ChampDBG/CL4RDS
---

## Abstract

Autonomous driving promises safer and more efficient transportation, yet level 5 autonomy remains out of reach due to the unpredictability of human road users. Reinforcement learning (RL) has emerged as a powerful framework for developing driving policies in simulators, but progress is hindered by the difficulty of handling diverse multi-agent interactions. Curriculum learning (CL) provides a principled way to adapt task difficulty to an agent’s capabilities, though most existing approaches alter environment parameters at the cost of realism. In this work, we introduce a simple yet effective CL-based RL method tailored for driving simulators with highly realistic interactive agent behaviors. Our approach dynamically adjusts scenario complexity—via traffic density and rollout horizon—based on the evolving performance of the policy, while preserving the fidelity of the base simulator.

We demonstrate our method using the TorchDriveEnv simulator, powered by the InvertedAI generative model, and evaluate it on agents that use a bird's-eye view and vectorized surrounding representations. We compare against a sequencing-based curriculum, as well as length-only and density-only based curricula. Our results show that coupling CL with a vectorized representation yields substantial improvements, achieving more than 40% higher returns and 60% better safety metrics compared to baseline policies. These findings highlight the potential of curriculum-guided RL for training robust and realistic autonomous driving agents in complex, stochastic driving simulators. 
