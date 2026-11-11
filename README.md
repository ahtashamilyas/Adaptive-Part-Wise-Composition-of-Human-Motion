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

2. Initially allow the model to establish a globally coherent body pose.

3. stitching schedule aparoaches:
    - (Late stitching: Start composition only in later denoising steps.) 
    - Progressive stitching : Gradually increase the composition influence.Gradually introduce the individual body-part constraints during the denoising process.
    - adaptive part stitching where we Train a neural network to predict optimal stitching schedules using karpathy auto-research loop (where it experiments with different aproachs and stitching schedules to improve benchmarks)


4. Compare the different stitching aproaches with the original method, which combines body-part predictions throughout denoising.
5. Evaluate whether the resulting motion is smoother and retains both intended movements.
This is particularly interesting because the authors of STMC explicitly identify their body-part stitching strategy as a limitation and suggest a more sophisticated stitching schedule as a potential improvement. 
