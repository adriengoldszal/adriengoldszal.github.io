---
layout: page
title: Automatic Puzzle Solver
description: Project for Ecole Polytechnique Image Analysis and CV Course
img: assets/img/puzzle/bon_match_scaling.jpg
importance: 2
category: Graphics & Computer Vision
repository: adriengoldszal/INF573_Puzzle
---

The challenge of puzzle solving is a subject that has occupied the scientific community working on computer vision for decades. Indeed, this 'problem', and the methods used to solve it, are often presented as the first step of a possible extension towards more serious subjects, such as the reconstruction of archaeological artifacts for example. It is a more playful approach that is at the origin of this project. Seeing the possibility of using computer vision to help with puzzle solving, this project proposes to develop an 'automatic' puzzle solving method, in real-time, observing the pieces with the user and indicating where to place them.

This project therefore implements numerous image analysis tools and seeks a certain form of robustness necessary due to its real-time operation. Our method proposes to function iteratively in real-time, allowing us to accompany the user in solving the puzzle. This approach, combined with better feature detection, allows for more robust puzzle resolution with a larger number of pieces, surpassing all benchmarks. 

<div class="row justify-content-center mb-4">
    <div class="col-auto">
        <a href="https://github.com/adriengoldszal/INF573_Puzzle" class="btn btn-primary btn-sm me-2" target="_blank">
            <i class="fab fa-github"></i> View Repository
        </a>
        <a href="{{ '/assets/pdf/CSC_51073_EP_Puzzle.pdf' | relative_url }}" class="btn btn-outline-primary btn-sm" target="_blank">
            <i class="fas fa-file-pdf"></i> Read Paper (PDF)
        </a>
    </div>
</div>

The overall puzzle solver pipeline is as follows:

1. A phone, attached to a stand, allows real-time observation of a certain number of pieces (about 10 to 15) on a white background. This phone is connected to the computer through an application to retrieve the camera feed. Additionally, the global image of the puzzle (on the box) is given as input.

2. An image is taken by the camera at a given regular interval and processed by the algorithm.

3. On the computer screen, an interface displays the phone's video, as well as the puzzle under construction, indicating each time which piece to move and its location on the final puzzle.

All implementation details: morphological operations, feature detection and matching, homographies, verification after piece selection, are in the attached paper.

{% if page.repository %}
<div class="repository-card">
  <h2>Associated Repository</h2>
  {% include repository/repo.liquid repository=page.repository %}
</div>
{% endif %}