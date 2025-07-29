# barnacles

I explored solving the barnacles counting problem using several approaches, many of which failed to produce satisfactory results at first due to the limited dataset size despite tiling and aggressive augmentation (rotation, flips, scaling, etc.).

I avoided using standard OpenCV solutions to get a more generalized barnacles detector which did not rely on brightness and color properties of the seen and unseen images provided now. It would have been useful to use OpenCV to obscure the regions outside of the target squares instead of using trusty GIMP if I had more time (*_nice.png).

I implemented a standard tiling routine to convert the images and corresponding masks into a larger number of training samples for both models.

My CNN approach uses a standard UNet ([UNet implementation taken from here](https://github.com/milesial/Pytorch-UNet/tree/master/unet)) on a ResNet101 backbone with pretrained weights from ImageNet to allow perception of edges and shapes already. Then I trained it on 128x128 tiles with a large overlap from the barnacles dataset. After 20 epochs, I get a dice score nearing 0.8 with a fair amount of learning for barnacle edges, though it could do with more training samples.

![](cnn/11-20.png)
![](cnn/unseen_img1_masks.png)
![](cnn/unseen_img2_masks.png)

I attempted to use a Progressive Fine-Tuning Approach with the UNet model, training it on inverted grayscaled MonuSeg nuclei images first (since they appear similar to white barnacles on dark backgrounds this way) and then on the limited barnacles dataset with frozen layers later, but it failed to converge.

For my second approach, I used a SOTA model in the same spirit, pretrained on a large number of medical imaging including nuclei and cells. CPSAM (Cellpose Segment Anything) post-trained on larger tiles (512x512) gave the best results among these approaches since Segment Anything perfors better than UNet in instance segmentation. Admittedly, my UNet results often had touching outlines which would be characterized as single contours, so I used connected component analysis to obtain final counts in the CNN approach.

I would have like to experiment with an ensemble routine which used Cellpose's strength on small finer details in large images compared to UNet's strength in instance segmentation. Perhaps using a kind of averaging ensemble or a classifier to learn the properties of ground truth barnacle masks (such as area, solidity and eccentricity) might have helped. I think such an ensemble approach might be useful, especially with limited data, given how Cellpose performs much better at instance segmentation (barnacles to barnacels) while UNet performs better at semantic segmentation (barnacles to everything else).

