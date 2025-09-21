---
title: "Navigating assistance system for quadcopter with deep reinforcement learning"
collection: publications
category: conferences
permalink: /publication/wu2018navigating
excerpt: "Introduces a new deep reinforcement learning method for quadcopters to navigate around obstacles."
date: 2018-08-07
venue: '2018 1st International Cognitive Cities Conference (IC3)'
paperurl: 'https://arxiv.org/abs/1811.04584'
# slidesurl: url
bibtexurl: /files/wu2018navigating.bib
# code: https://github.com/ChampDBG/AirSim_train
---

## Abstract

This paper presents a novel deep reinforcement learning (DRL) method for autonomous quadcopter navigation and obstacle avoidance. Prior research in this domain has been limited by algorithms that only control the forward direction of the vehicle. Our proposed system integrates two distinct control functions: a path-planning-based navigation function and a collision avoidance function implemented via a Deep Q-Network (DQN). The navigation function computes a direct path to the goal coordinates, while the DQN model provides the capacity for the quadcopter to rotate and adjust its altitude to bypass obstacles. The final flight command is a combined output from both functions. Experimental results from 500 simulated flights demonstrate a low collision rate of 14%. This work lays the foundation for future research aimed at training the model on more complex environmental data and transferring the learned policy to a physical quadcopter.
