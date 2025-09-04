---
layout: page
title: Self-play for adversarial games
description: Ecole Polytechnique Advanced Deep Learning and Autonomous Agents Course
img: assets/img/cathedral/cathedral_game.jpg
importance: 1
category: Reinforcement Learning
related_publications: false
repository: gabriel-mercier/Deep-RL-Agents-for-Cathedral-Board-Game
---

In this project, done for Ecole Polytechnique's Advanced Deep Learning and Autonomous Agents class, we teach agents to play the Cathedral 2-player board game. We first do a review of DQN and PPO algorithms, study and optimize their performance in this setting. This also implies adapting the environment to our needs. In addition, we then test the Representation Learning method and analyze its impact on the training objective.

<div class="row justify-content-center mb-4">
    <div class="col-auto">
        <a href="https://github.com/gabriel-mercier/Deep-RL-Agents-for-Cathedral-Board-Game" class="btn btn-primary btn-sm me-2" target="_blank">
            <i class="fab fa-github"></i> View Repository
        </a>
        <a href="{{ '/assets/pdf/Projet_RL.pdf' | relative_url }}" class="btn btn-outline-primary btn-sm" target="_blank">
            <i class="fas fa-file-pdf"></i> Read Paper (PDF)
        </a>
    </div>
</div>

With this project, we wanted to study how an agent could efficiently learn to play a two-player game. What were the best methods in terms of training in this adversarial setting, as well as the best algorithms. Going further, we wanted to see how Representation Learning techniques could help to scale and accelerate this learning.

Learning to play board games was one of the first elements of the advent of RL, notably through the game of chess, which was Demis Hassabis’s main motivation behind founding Google DeepMind in 2014, which culminated with AlphaZero in 2017. These 2-player adversarial games bring a unique challenge to them because of their non-stationarity, the high branching factor, and the sparse rewards.

Training such algorithms can be complicated. Should supervision be involved? How should an opponent be crafted? Or should it be purely self-play, as is increasingly the case today. The question of structuring this adversarial self-play setting is also challenging. Should there be two policies, or just one? Choosing and tuning the optimal algorithm is also crucial.

While algorithms like **Proximal Policy Optimization** (PPO) demonstrate state-of-the-art performance on high action spaces and multi-agent learning, works like those by DeepMind and AlphaZero show that having **Monte-Carlo Tree Search** (MCTS) in addition to deep RL clearly improves performance in adversarial 2 player games like Chess, for example. **Deep Q-Networks** (DQN) is not a state-of-the-art model for adversarial games. However, it remains a viable choice with certain modifications. Its simplicity and accessibility make it particularly attractive for experimentation, as it is relatively easy to implement and requires fewer computational resources compared to policy gradient methods.

An idea we also wanted to explore was **Representation Learning**, or learning meaningful representations of the large observation space, which can help reduce dimensionality, improve generalization, and facilitate style separation in our diffusion model. Various approaches exist for learning such representations, including auto-encoders, contrastive learning, and self-supervised learning methods.

We therefore decided to study a 2-player adversarial board game called **Cathedral**, where dark and light factions battle for terrain on a grid (or fortified village), by placing pieces in turn-based fashion. This strategy game, for which we found an open source, but **very much unused**, PettingZoo environment, allowed us to adapt the environment and implement and test our algorithms. 

{% if page.repository %}
<div class="repository-card">
  <h2>Associated Repository</h2>
  {% include repository/repo.liquid repository=page.repository %}
</div>
{% endif %}