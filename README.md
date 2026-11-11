# Implementing AutoResearch to optimize Adaptive-Part-Wise-Composition-of-Human-Motion
leveraging a pretrain Motion Diffusion Model, to create Motion Prediction.
STMC
Multi-Track Timeline Control · CVPR Workshop 2024
Official repository: nv-tlabs/stmc
the original paper provides:
- Pretrained MDM-SMPL diffusion model.
- Motion generation from multiple action instructions.
- Body-part-specific motion composition.
- MTT benchmark.
- Existing SINC-style baselines.
- Evaluation scripts and visualizatio

- Our proposed small contribution
Instead of designing an entirely new motion-generation model, we can build on an existing diffusion-based composition method and investigate (Late stitching
Start composition only in later denoising steps.) and Progressive stitching
Gradually increase the composition influence.then we can try adaptive part stitching where we Train a neural network to predict optimal stitching schedules in karpathy auto-research loop

The idea is:
1. Initially allow the model to establish a globally coherent body pose.
2. Gradually introduce the individual body-part constraints during the denoising process.
3. Compare this gradual integration with the original method, which combines body-part predictions throughout denoising.
4. Evaluate whether the resulting motion is smoother and retains both intended movements.
This is particularly interesting because the authors of STMC explicitly identify their body-part stitching strategy as a limitation and suggest a more sophisticated stitching schedule as a potential improvement. 
The Computer Vision Foundation+1


That gives us a specific starting point in existing research rather than inventing a problem.
