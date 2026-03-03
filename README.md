### Project title
CropWorld: Learning Driving Context Beyond the Horizon with Cropped Map
### Author
Yongjae Kim
### Academic supervisor
Hwangbo Jemin
### Keyword
Autonomous Driving, Model Predictive Control, Model-Based Reinforcement Learning 
## 1. Introduction and Motivation
![horizon](horizon.jpg)*Caption goes here*

A fundamental dilemma in autonomous off-road navigation is balancing prediction horizons with computational feasibility. In unstructured environments, traditional mathematical modeling of vehicle dynamics is exceptionally difficult due to complex, highly stochastic vehicle-terrain interactions like traction loss and suspension compression. Consequently, robust navigation systems must rely on Neural Network (NN) vehicle dynamics as the core model to accurately roll out sample trajectories.

However, utilizing NN dynamics within a Model Predictive Control (MPC) framework—such as Model Predictive Path Integral (MPPI)—incurs significant computational costs. To maintain real-time execution, the controller is forced to operate over very short, myopic horizons. This creates a critical limitation: even when rich, long-term environmental intelligence is readily provided via Bird’s Eye View (BEV) cost maps, the short-horizon planner cannot effectively leverage it. This myopia frequently funnels the vehicle into unrecoverable kinematic traps (e.g., steep ravines or dead ends) that lay just beyond its immediate predictive horizon.

The primary objective of this research is to bridge this gap. This work explores the integration of long-term BEV map intelligence into conventional, short-horizon MPPI frameworks with only a permissive computational overhead. Ultimately, the goal is to learn a Value Model that accurately predicts the future "outcome" (terminal cost) given the current driving context—specifically, latent vehicle states matched to cropped BEV maps. To validate this framework, high-fidelity simulation experiment will be conducted on various benchmark tracks including digital-twin of Changwon Proving Ground from Agency for Defense Development (ADD). Also, physical experiments is planned to be conducted using a 1/10 scale modified RC-car equipped with an NVIDIA AGX Orin for onboard edge computation, demonstrating real-time agile deployment.

## 2. Problem Statement

To achieve high-speed, safe autonomy without exceeding computational budgets, the system must learn to evaluate long-term outcomes from compressed driving contexts. Existing architectures struggle to achieve this due to three primary bottlenecks:

1. **The Computational Bottleneck of Neural Dynamics:** Rolling out Neural Network dynamics models over long horizons for thousands of MPPI samples is computationally prohibitive on edge devices, forcing the use of short, blind horizons that fail to utilize the given BEV cost maps.
2. **Learning Long-Term Outcomes from Cropped Contexts:** While world models can effectively compress state spaces, translating these latent representations—combined with spatially cropped BEV maps—into a reliable Value Model that accurately predicts future safety and task outcomes remains a significant challenge, particularly in offline settings.
3. **Gradient Instability in Multimodal Fusion:** Combining high-dimensional spatial data (cropped BEV maps) with low-dimensional task intents and latent vehicle states inherently causes internal scale conflicts and gradient explosion during Value Function approximation, destabilizing the learning of the driving context.

## 3. Proposed Methodology

This research addresses these bottlenecks through a three-pillared architectural approach, strictly optimized for real-time inference on edge computing platforms.

### 3.1. Leveraging Latent Context via World Models

To generate the driving context required for the Value Model, this research adopts the Recurrent State Space Model (RSSM) architecture, foundational to the Dreamer agent. While the RSSM is an established framework for latent dynamics, this work adapts it to continuously match stochastic latent vehicle states ($h, z$) with cropped BEV maps. This provides the Value Model with a compact, highly expressive representation of the vehicle's physical reality and immediate surroundings, without the computational burden of rolling out raw visual data.

### 3.2. Symmetrical Multimodal Representation Alignment (Late Fusion)

To resolve gradient explosion and successfully learn the future outcomes, we abandon raw tensor concatenation in favor of a normalized **Late Fusion** strategy.

* **Input Embeddings:** Every modality comprising the driving context—Dynamics ($h, z$), Spatial (Cropped Map), and Task (Goal, Action)—passes through an independent, fused projection block consisting of Linear, LayerNorm, and SiLU activations.
* **Global + Local Spatial Awareness:** The spatial feature branch is computationally optimized by pairing an Attention Context vector (local collision avoidance) with a Global Average Pooled BEV vector (heuristic terrain cost). This distills the cropped BEV map into a lightweight format, preserving long-horizon foresight.

### 3.3. Offline Value Learning via Safe Dataset Synthesis

To train a robust Implicit Q-Learning (IQL) Value Model that accurately predicts terminal outcomes, the network must understand the boundaries of safe navigation. However, generating negative examples (collisions, getting stuck) on physical hardware is highly destructive. Instead, this methodology leverages the learned stochastic dynamics and given BEV maps to generate a "Gaussian Noisy Expert" dataset offline. This consists of a 60/30/10 ratio of nominal expert driving, hallucinated recovery maneuvers, and simulated terminal failures, allowing the Value Model to learn the true cost of diverse driving contexts.

## 4. Expected Contributions and Impact

This research will provide a mathematically sound and practically deployable framework for computationally efficient autonomous navigation. The key contributions include:

1. **Bridging the Horizon Gap with Contextual Value Models:** Demonstrating that short-horizon MPPI can achieve long-horizon foresight by learning a Value Model that predicts future outcomes based on latent driving contexts and cropped BEV maps, adding minimal latency during deployment.
2. **Safe, Offline Learning for Real-World Robotics:** This framework is exceptionally well-suited for real-world vehicular applications where collecting physical obstacle avoidance and collision datasets is prohibitive due to wear-and-tear concerns. By synthesizing learned dynamics with BEV maps extracted from normal, safe driving datasets, this approach allows for the generation of an offline dataset that explores collision boundaries and recovery states. 
3. **Architectural Stability for Edge Hardware:** By formalizing multi-modal input embeddings and utilizing tensor compilation techniques, this work will prove that complex value approximation networks can execute reliably in real-time on standard edge robotics hardware.
