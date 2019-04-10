+++
date = "2019-04-03"
title = "Eikonal Based Region Growing for MRI Organ Segmentation "
slug = "region-growing"
+++

ECE 435: Medical Image Processing Final Project

Applying the Eikonal equation to segment organs at risk (OARs) surrounding the prostate when planning prostate radiotherapy using the [Gold Atlas - Male Pelvis data set](https://aapm-onlinelibrary-wiley-com.ezproxy.library.uvic.ca/doi/epdf/10.1002/mp.12748). 

This project takes user input as the seed pixel and proceeds to grow a superpixel region. The figure below illustrates the algorithm working to segment the bladder.  

{{< figure src="/images/regiongrowing/erg_bladder.gif" caption="">}}
*Bladder Segmentation*

The basis for this project is ["Eikonal-Based Region Growing for Efficient Clustering" by P. Buyssens et. al.](https://www.sciencedirectcom.ezproxy.library.uvic.ca/science/article/pii/S0262885614001565). The paper proposes creating superpixels based on the Eikonal equation. Solving Eikonal equation gives shortest path between pixels based on a weight function and can be solved efficiently using Fast Marching Algorithm. In this paper the weight function is computed based on the pixel colour or intensity. The main advantages of this approach are excellent boundary adherence and low computational intensity. 

Checkout the source code on [github](https://github.com/tmastrom/EikonalBasedRegionGrowing)




