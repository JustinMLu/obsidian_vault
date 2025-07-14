
1. ✅ Compare YOLO+SAM vs DINO+SAM - timebox this until end of day Monday

2. PR that contains training scripts
	- Ensure Camilo and Brad can run training. Something we can run, not school server

3. PR that officially moves us over to YOLO+SAM as primary detection. With backward compatibility with seg model and grounding DINO

4. Update all ML models to run on GPU
	- YOLO
	- SAM (do not worry about converting DINO)
	- Segmentation

5. Try torch helper function to convert to TensorRT but do not spend any time debugging if it does not work right off the bat

6. Isaac SIM
	- Run on AGX
	- Hyperrealistic video to show investors
	- Maybe cool video of a thousand robots going at it
	- Ensure Camilo and I can replicate the process
