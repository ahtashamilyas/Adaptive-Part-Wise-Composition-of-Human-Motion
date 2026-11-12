# Optimizing Part-Wise-Composition-of-Human-Motion
In the original paperMulti-Track Timeline Control for Text-Driven 3D Human Motion Generation with repository: nv-tlabs/stmc:
- Multi-Track Timeline Control. 
- leveraging a Pretrained MDM-SMPL diffusion model to create Motion generation/composition.
- Motion generation from multiple action instructions.
- Body-part-specific motion composition.
- MTT benchmark.
- Existing SINC-style baselines.
- Evaluation scripts and visualization



### Our proposed idea is Leveraging the same motion diffusion models  (MDM) and investigate whether changing the stitching schedule during the denoising process has and effect on the results Does it give smoother motion?: 

2. Initially allow the model to establish a globally coherent body pose. by must rank your 3+  motion crops by how much they impact the body's center of mass. and that's is how you eleminalte the effect of the order of the movement of the body parts in the timeline, we establish a hierarchy staggering their introduction based on physical hierarchy rather than injecting them all at once

3. stitching schedule aparoaches:

    - (Late stitching: Start composition only in later denoising steps.) 
Hierarchical Timestep Scheduling: Because early diffusion timesteps (heavy noise) dictate the global structure and later timesteps (low noise) refine high-frequency details, you must rank your 3+ crops by how much they impact the body's center of mass. To prevent the multiple conditions from fighting for structural dominance, the foundational global movement (e.g., walking or jumping) would dictate the earliest, heavily noised structural denoising steps. Secondary and tertiary localized limb movements (e.g., waving an arm, then kicking a leg) would be staggered and injected sequentially only during the later detail-refinement steps 

The SMPL model provides the hardcoded physical map, but you must actively calculate the ranking of your specific crops to determine when to inject them.Here is how the model's hierarchy and your crop calculations interact to create the schedule:

The SMPL Built-in Hierarchy (The Map): The SMPL model explicitly defines a strict "kinematic tree." At the absolute base of this tree is the pelvis (the root joint), which acts as the body's physical center of mass. Every other joint is a "child" or "grandchild" radiating outward from the pelvis (e.g., Pelvis $\rightarrow$ Spine $\rightarrow$ Collarbone $\rightarrow$ Shoulder $\rightarrow$ Elbow $\rightarrow$ Wrist).

Ranking the Crops (The Calculation): You cannot just feed actions into the model randomly. Because early diffusion timesteps dictate the global structure of the pose, you must rank your 3+ motion crops based on how heavily they dictate the movement of that SMPL root joint (the center of mass).

The Execution (Timestep Scheduling): You combine the map and the calculation. You evaluate which SMPL joints are required for each of your textual crops, measure how close those joints are to the root pelvis, and stagger their introduction into the diffusion loop based on that distance.

Structuring your schedule so that the primary crop runs all the way to $T_0$, while secondary and tertiary crops join mid-flight, creates a **cumulative spatial mask**. This is the correct architectural approach for progressive composition.

If you stopped processing the primary crop at $T_{500}$, the avatar's legs and torso would stop denoising, leaving them as a blurry, jittery mess in the final animation. By keeping the primary crop running to $T_0$, you ensure the foundational movement fully resolves, while the localized movements are injected on top.

Here is exactly how the math and physical rendering play out under this specific schedule:

### Phase 1: Global Dominance ($T_{1000}$ to $T_{501}$)

* **The Math:** The diffusion loop ignores the secondary and tertiary crops entirely. The pose vector for the entire body (indices 0 through 263) is dictated 100% by the Primary crop ("Walking").
* **The Physical Result:** The model spends its heaviest noise-reduction steps establishing a rock-solid center of mass, global momentum, and stride length. During this phase, the avatar's arms are doing a standard, relaxed walking swing, and the head is facing forward.

### Phase 2: The Secondary Override ($T_{500}$ to $T_{201}$)

* **The Math:** Spatial stitching activates. The algorithm continues pulling the leg and torso numbers from the Primary crop ("Walking") to ensure they keep refining, but it now overrides the arm indices with the numbers from the Secondary crop ("Waving").
* **The Physical Result:** At step 500, the avatar's arm suddenly swaps its latent trajectory from a "relaxed swing" to a "raised wave." Because $T_{500}$ still contains a moderate amount of noise, the diffusion model has enough flexibility to naturally reconcile this swap. It will subtly adjust the spine and shoulder in the subsequent steps to balance the weight of the newly raised arm.

### Phase 3: The Tertiary Polish ($T_{200}$ to $T_{0}$)

* **The Math:** The second spatial mask activates. The legs come from Walking, the arm from Waving, and now the neck/head indices are overridden by the Tertiary crop ("Nodding"). All three crops are now being processed simultaneously to drive their respective body parts.
* **The Physical Result:** Because $T_{200}$ represents very low noise, the model treats the new head movement as a high-frequency surface detail. The head nods perfectly without sending structural shockwaves down the spine that could disrupt the established walking and waving mechanics.

### The Architectural Catch: "Latent Shock"

If you implement this schedule via a hard conditional statement (`if t == 500: inject_arm()`), you will introduce a severe mathematical discontinuity. Instantly swapping the arm's vector data from one crop to another at a single timestep will cause a visual glitch or artifact in the final generation.

**How to fix it:** You must use **alpha-blending** across a transition window. Instead of a hard swap at $T_{500}$, you blend the vectors from $T_{550}$ down to $T_{500}$.

* At $T_{550}$, the arm is 90% Walking crop, 10% Waving crop.
* At $T_{525}$, it is 50/50.
* By $T_{500}$, it is 100% Waving crop.
This eases the latent numbers into their new trajectory, allowing the diffusion model to seamlessly stitch the movements together.

  ##  Progressive stitching :
   Gradually increase the composition influence.Gradually introduce the individual body-part constraints during the denoising process.
Progressive Spatial Masking: Instead of a single mask shift, you would continuously and dynamically shift the attention masks across the specific SMPL joints for each distinct movement as the diffusion process steps forward. This allows the multiple isolated latent features from the different crops to smoothly blend into the global motion one by one without structural interference.

    
   What the autoresearch loop should (and shouldn’t) do

Automate:
- Sweep schedule parameters: switch step, ramp shape, order, pair, seed.
- Generate motions, compute metrics, log everything, and produce a summary table or plot.
- Propose the next batch of configs, and flag anomalies (NaNs, foot-skate spikes). 
An agent that edits the pipeline or the metrics on its own is where this goes wrong. Let it change configs only, never code.
    
    
    
4. experiment with adaptive part stitching where we Train a neural network to predict optimal stitching schedules using karpathy auto-research loop (where it experiments with different aproachs and stitching schedules to improve benchmarks)


4. Compare the different stitching aproaches with the original method, which combines body-part predictions throughout denoising.
5. Evaluate whether the resulting motion is smoother and retains both intended movements.
This is particularly interesting because the authors of STMC explicitly identify their body-part stitching strategy as a limitation and suggest a more sophisticated stitching schedule as a potential improvement. 



