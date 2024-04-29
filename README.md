# Thesis-Synthetic-Data-Generation
Synthetic Data Generation for Autonomous Driving using Novel View Synthesis

## Abstract

The latest research in the area of autonomous driving focuses on an end-to-end approach, where neural networks learn to control vehicles directly from environmental perception. However, training these agents requires a lot of data covering various scenarios. Synthetic data generation from real-world recordings offers a promising solution to address this issue. The core of this solution uses novel view synthesis (NVS) algorithms. The thesis delves into the process of generating novel viewpoints using a modular approach. An existing state-of-the-art NVS pipeline was selected that combines various modules such as depth estimation, point cloud transformation, point cloud renderer, and image completion model. The impact of each component on the final output was analyzed. 

CARLA, an open-source driving simulator was used to create a dataset with a wide range of camera angles and positions. The quality of the NVS output obtained from the Synsin end-to-end trained depth estimation model was compared with that of two pre-trained models, LeRes and Midas, which are considered state-of-the-art in depth estimation and were pre-trained on mixed datasets. The effectiveness of different refinement techniques based on Generative Advesarial Network (GAN) and Fast Fourier Convolutions (FFC) were compared after combining them with depth estimation modules. The improved pipeline was tested with publicly available datasets, KITTI and PandaSet, and was found to be adaptable with proper scaling of estimated depth and transformation values for datasets on which the model was not originally trained. 

Analysis showed that the end-to-end trained depth estimation module produced better-quality novel images compared to pre-trained depth models. Integrating the image completion method (LaMa) into the end-to-end pipeline resulted in a 20% improvement in Peak Signal to Noise Ratio (PSNR) value and and 7% increase in perceptual quality of novel views compared to the baseline.

## Summary of thesis

The thesis aimed to explore novel view synthesis in a modular approach. An existing pipeline was selected [[Synsin](https://github.com/facebookresearch/synsin)] that integrated all the subtasks needed for generating a new view into a single linear stream, which was then trained together. The primary objective was to break down these submodules to examine the impact of each on the final output. 

To accomplish this, a large dataset of driving scene images with corresponding camera positions was necessary. The dataset should also include images captured from various camera angles for the same vehicle position. Unfortunately, publicly available datasets lacked such diversity. As a result, the required dataset was created using [CARLA](https://github.com/carla-simulator/carla), an open-source driving simulator. Variations in angles were recorded within the range of ±30 degrees, and translations within the range of ±2, which was assumed to cover the standard variations in viewing angles in most driving scenarios. A total of 10,000 image variations were recorded. 

The modules being evaluated include depth estimation, point cloud transformation and rendering, GAN refinement, and image completion. Various pre-trained depth estimation models have been studied, each with its unique architecture and cutting-edge techniques. For the end-to-end trained approach, a [UNET](https://github.com/zhixuhao/unet)-based architecture is used in the base pipeline. The [LeRes](https://github.com/aim-uofa/AdelaiDepth/tree/main/LeReS) pre-trained depth model has also been examined, which utilizes a ResNext-based architecture trained on a mixed dataset using multicurriculum learning techniques. Additionally, the depth estimated from the vision transformer-based depth estimation model ([Midas](https://github.com/isl-org/MiDaS)) is compared with the ground truth depth obtained from the CARLA dataset. 

The point clouds are obtained directly from the depth maps, and the required transformations for the novel view are applied directly to them. Experiments have shown the importance of correctly scaling and shifting during transformation. Due to the parallax effect, points closer to the origin experience a much larger translation compared to those further away. Therefore, the transformation values applied should match the scaling applied, which greatly affects the generation of the novel viewpoint. The transformed point clouds are rendered using a differentiable renderer. A comparison was made between this method and a normal renderer, which showed that the differentiable renderer, which considers an area of pixels during splatting to the z-buffer and accumulation method, is more suitable for the view synthesis process, as it produces fewer missing pixels during large view changes. The effect of increasing and decreasing the radius of this area was also studied. A too-large area could cause the image to be blurry and lose the quality of the artefacts in the scene, while a too-small area could lead to missing pixel values and unwanted artefacts during the refinement process. 

The study compared the effectiveness of refinement models when combined with both custom end-to-end trained and pre-trained estimated depths. Surprisingly, the depth estimated by the end-to-end trained prediction module, which evaluated lower depth accuracy compared to pre-trained models, enabled better quality novel view synthesis. The image completion method (LaMa) was successfully integrated with the end-to-end pipeline, resulting in a 20% improvement in PSNR values and significantly better perceptual quality of the generated novel views. The improved pipeline was tested with publicly available datasets like KITTI and PandaSet, and it was observed that with appropriate scaling of estimated depth and transformation values, the modified pipeline could be used with other datasets on which the model was not trained. Overall, the study analyzed the significance of each module in a base novel view synthesis pipeline and replaced them with advanced modules and techniques to improve the quality of the synthesized novel view.

## Modified Pipeline

![Untitled](https://github.com/sonyjf/Thesis-Synthetic-Data-Generation/assets/102171203/d8dac942-2ea6-4735-8c47-f3eee62cde09)


Modified Synsin pipeline with image completion module integrated during inference. The image is taken by feature extraction(f ) and depth estimation network(d). The output is used to create point cloud that is transformed based on the input transformation. The novel view image is rendered from point cloud by accumulation, with splatting and compositing techniques. The generator network(g) improves the image further, filling in gaps or adding semantically meaningful artifacts. The image completion model enhances the image for large view changes by combining output from the generator and calculating the mask.

## Results

e2e-GAN - end to end trained depth Synsin pipeline with depth estimation and GAN refinement model  
gtDepth-GAN - Synsin pipeline with ground truth depth input and GAN refinement model  
Midas-GAN_LaMa - Pipeline with Midas depth estimation and LaMa refinement model  
LeRes-GAN_LaMa - Pipeline with LeRes depth estimation and LaMa refinement model  
e2e-GAN_LaMa -  end to end trained depth Synsin pipeline with depth estimation and LaMa refinement model  
gtDepth-GAN_LaMa - Synsin pipeline with ground truth depth input and GAN + LaMa refinement model  
gtDepth-GAN_LaMa-finetune - Synsin pipeline with ground truth depth input and GAN + LaMa refinement model, further fine-tuned for missing regions  

<img width="613" alt="1" src="https://github.com/sonyjf/Thesis-Synthetic-Data-Generation/assets/102171203/f412f24e-b3f8-4669-975d-b64f96f4eae9">


The output obtained for each pipeline is shown below

<img width="318" alt="2" src="https://github.com/sonyjf/Thesis-Synthetic-Data-Generation/assets/102171203/caf20c84-1150-4901-b9bc-24d620921e71">


Rendered images from the Synsin end-to-end trained model, LaMa integrated pipeline, and LeRes pretrained depth estimation model respectively compared to the ground truth novel view image (top).   
The end-to-end model (second image ) fails to properly outpaint the new regions, marked by the red box.  
The pipeline integrated with LaMa model does a better job at outpainting those regions, improving the quality of 
Synthesised Novel views.  

## Use case - Simulation Tool with NVS

Base open source data-driven simulator from MIT research- [Vista Driving Simulator](https://github.com/vista-simulator/vista) 

The simulator takes camera recordings, odometry data (IMU sensor and Speed) and creates an interactive environment for training autonomous driving agents. However, the view synthesis technique fails to synthesize views for large camera angle changes, as seen in the video below (top-left window). This problem has been solved by integrating the modified NVS technique to outpaint for larger viewing angles (bottom-left window). The simulation environment window ( top-right )displays the orientation of the agent in the environment. This orientation is given as a keyboard input by the user. The intermediate depth estimation model output is shown in the bottom-right

https://github.com/sonyjf/Thesis-Synthetic-Data-Generation/assets/102171203/a17c355c-8d74-49f4-b29a-320668b502c7


