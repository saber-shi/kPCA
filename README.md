 # Kernel PCA for Novelty Detection [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]
 ### Introduction
 The goal of an anomaly (outlier or novelty) detection method is to detect anomalous points within a data set dominated by the presence of ordinary background points. Anomalies are by definition rare and are often generated by different underlying processes [[2](https://www.ncbi.nlm.nih.gov/pubmed/27093601),[3](https://onlinelibrary.wiley.com/doi/abs/10.1002/bimj.4710290215)]. Numerous algorithms have been devised toward this goal, the results of which have been applied to a variety of fields to improve upon domain-specific, rule-based detection methods. Anomaly detection has applications in medicine, fraud detection, fault detection, and remote sensing, among others [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414),[2](https://www.ncbi.nlm.nih.gov/pubmed/27093601)]. 
  
  Background data is often produced by non-linear processes [[4](https://www.spiedigitallibrary.org/conference-proceedings-of-spie/10198/101980X/A-study-of-anomaly-detection-performance-as-a-function-of/10.1117/12.2264160.full?SSO=1)]. Kernel-based learning methods are motivated by the idea that there exists a better model of the data in a transformed, nonlinear, feature space (F). A kernel function allows the efficient computation of inner products in F without the explicit calculation of the mapping. The most popular amongst these methods for anomaly detection is the One-Class Support Vector Machine (OC-SVM), which separates the data from the origin in F [[5](http://rduin.nl/papers/prl_99_svdd.pdf)]. For a Gaussian radial-basis-function (rbf) kernel, this process is equivalent to spherically enclosing the data in F.
  
 ### Kernel PCA
  Hoffman claims that because samples are treated independently a OC-SVM produces a boundary that is too large to tightly model the background data, causing false positives [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]. Hoffman draws from the benefits of kernel techniques and the potential limitations of SVMs, by using kernel PCA (kPCA) to better model the relationship between background points [[4](https://www.spiedigitallibrary.org/conference-proceedings-of-spie/10198/101980X/A-study-of-anomaly-detection-performance-as-a-function-of/10.1117/12.2264160.full?SSO=1),[6](https://dl.acm.org/citation.cfm?id=2888116.2888129),[7](https://dl.acm.org/citation.cfm?id=295960)]. The separation of points in F and the background model serves as an anomaly score. 
  
  Hoffman begins by outlining the process of performing PCA in the feature space as well as how to obtain the anomaly score (reconstruction error) only from the ambient inputs using the kernel trick [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]. In short, the anomaly score is the points euclidian distance from the orgin in (F) subtracted from the distance to the points projection onto some q, number of retained principal components in feature space.
  
  Hoffman then presents several evaluations of kPCA on a number of real-world and toy data sets, kPCA demonstrating better generalization, accuracy, and robustness over linear PCA, the Parzen density estimator, and OC-SVMs [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]. The two real-world datasets Hoffman tests are the Wisconsin Breast Cancer (Diagnostic) Test [[8](https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+%28Diagnostic%29)], and a generated anomaly example from MNIST [[9](http://yann.lecun.com/exdb/mnist/)], where digit zero repersents the background and other digits serve as anomalous examples. Comparisons of specificity, sensitivity, and accuracy were made amongst kPCA, linear PCA, Parzen Density Windows, and the OC-SVM on these datasets using reciever operating characteristic (ROC) curves which plot false positive and true positive rates at different thresholds of anomaly scores. A desriable curve has a high true positive rates at low false positiver rates. An entire ROC curve can be summarized by the area under the curve (AUC). AUC can very between 0.5 (random guess) and 1.0 (perfect detection). 
  
### Objective

The objective of this study is to reproduce Hoffman's comparison between kPCA, linear PCA, Parzen Density Windows, and the OC-SVM on the 'Breast Cancer' and' Digit 0', datasets. In addition I expand the comparison to four additional (two real, two generated) datasets. ROC curves are preduced and AUCs are calculated for each dataset.

### Methods

 To begin, The 'Digit 0' and Breast Cancer datasets where obtained and augmented following the same steps specified by Hoffman [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]. Two more datasets, 'Glass' and 'Ionosphere', were obtained from ODDS [[10](http://odds.cs.stonybrook.edu/)].  Data preprocessing for these follow the same steps Hoffman used for the original Breast Cancer dataset, namely dividing each attribute by its standard deviation to ensure unit variance as well as adding a small amount of uniform noise to the data [-0.05,0.05] to help avoid numerical issues [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)]. Fianlly, two artifical, non-linear, toy datasets were generated refered to as 'Circles' and 'Roll'.

  In this reproduction study, I use an implmentation of OC-SVMS from the PYOD toolkit for python which creates a wrapper function around the base sklearn library [[11](https://github.com/yzhao062/pyod)]. On his website, Hoffman provides a MATLAB implmentation of his algorithm [[12](http://www.heikohoﬀmann.de/kpca.html)]. I ported Hoffman's code to python class and removed most of the loops via Numpy to improve efficiency. As Hoffman described, the Parzen Density window is simply kPCA with no retained eigenvectors, or otherwise stated the points distance from the orgin in (F). This can be implmentented using the same kPCA class and setting the number of retained eigenvectors to zero. I created a seperate class to perform linear PCA and calculate the reconstruction error. The purpose of the additional datasets is to further distinguish the different anomaly detection methods.  
  
  One of the issues I had with Hoffman's original evaluation was the absence of a validation set. He caputres the background model with a training set, but then performs parameter tuning directly on the test set. This is not representive of how the algorithms would be employed in the realworld application. In testing you do not know what the anomlous points are. To correct this in my reproduction, I equally, randomly split Hoffman's 'Breast cancer' and 'Digit 0' test sets as well as the additional datasets into a validation and test set. Training (creating a model of the background) is still done on clean data containing no anomalies, however parameter tuning is now done on a grid search of the validation data. The best parameters in the validation set (as measured by the AUC on the validation set) are then used for testing, where we are blind to the distribution and characteristics of the anomalies.  
  
  Parameter search:
  
  kPCA: Sigma was changed from 1e-2 to 1e2 along a log scale. The integer number of retained principal components (q) was changed from 1 to 100 (stopping at n-5) along a linear scale. Each parameter took 50 different values with each pair representing a parameter setting, so 2500 parameters in total were checked during validation for each datasets.
  
  PCA: The integer number of retained principal components was changed from 1 to D. Note: At D the anomaly score for all points equals zero.
  
  Parzen Window: Sigma was changed from 1e-2 to 1e2 along a log scale. 50 different sigmas were checked during validation for each datasets.
  
  OC-SVM: In addition to sigma, OC-SVMs have another important parameter 'nu, which represents the lower bound for the number of samples that are on the wrong side of the hyperplane, representing a trade off between overfitting and underfitting [[13](https://papers.nips.cc/paper/1723-support-vector-method-for-novelty-detection.pdf)]. Sigma was changed from 1e-2 to 1e2 along a log scale. nu was changed on a linear scale from 0.01 to 0.99. Each parameter took 50 different values with each pair representing a parameter setting, so 2500 parameters in total were checked during validation for each datasets.
  
### Results


#### Parameter Search
The figures below show the results of the grid searches for each of the four anomaly detection methods on the validation set for each data set. The star indicates the best paramter choices in the 2D grid, the colorbar represents AUC. 

![alt text][Cancer]
![alt text][Digit0]
![alt text][Glass]
![alt text][Ionosphere]
![alt text][Circles]
![alt text][Roll]

[Cancer]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Cancer.png "Cancer Parameter Search"
[Digit0]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Digit0.png "Digit0 Parameter Search"
[Glass]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Glass.png "Glass Parameter Search"
[Ionosphere]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Ionosphere.png "Ionosphere Parameter Search"
[Circles]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Circles.png "Circles Parameter Search"
[Roll]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Roll.png "Rolls Parameter Search"

#### Test ROC Curves

Next, using these best parameter settings, ROC curves are generated based on detection on the test set. Note that the false positive rate is presented on a log scale to emphasize the most important region.

![alt text][CancerR]
![alt text][Digit0R]
![alt text][GlassR]
![alt text][IonosphereR]
![alt text][CirclesR]
![alt text][RollR]

[CancerR]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Cancer%20roc.png "Cancer ROC curves"
[Digit0R]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Digit0%20roc.png "Digit0 ROC curves"
[GlassR]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Glass%20roc.png "Glass ROC curves"
[IonosphereR]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Ionosphere%20roc.png "Ionosphere ROC curvesh"
[CirclesR]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Circles%20roc.png "Circles ROC curves"
[RollR]: https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Roll%20roc.png "Rolls ROC curves"

#### Summary

This table summarizes the results of the parameter search and testing. 

| DataSet     | Method   | Best Param      | Val AUC  | Test AUC|
| ----------- |:--------:| ---------------:|:--------:|:--------:|
| Cancer |   kPCA       |  q: 1    sigma: 1.9307 | 0.9936 | **0.9992** |
| Cancer |   PCA       |  q: 1    | 0.6719 | 0.401 |
| Cancer |   ParzenWin       |  sigma: 1.9307    | 0.9936 | **0.9992** |
| Cancer |   OCSVM       |  nu: 0.83 sigma: 1.9307    | 0.9938 | **0.9992** |
| Digit0 |   kPCA       |  q: 67    sigma: 4.9417 | 1.0 | **1.0** |
| Digit0 |   PCA       |  q: 20    | 0.7203 | 0.4813 |
| Digit0 |   ParzenWin       |  sigma: 0.1151    | 0.9999 | 0.9998 |
| Digit0 |   OCSVM       |  nu: 0.51 sigma: 0.1389    | 0.9999 | 0.9998 |
| Glass  |   kPCA       |  q: 46    sigma: 12.6486 | 0.8657 | 0.6727 |
| Glass  |   PCA       |  q: 7    | 0.6971 | 0.4091 |
| Glass  |   ParzenWin       |  sigma: 0.2024    | 0.82 | 0.6045 |
| Glass  |   OCSVM       |  nu: 0.03 sigma: 0.1677    | 0.82 | **0.7364** |
| Ionosphere  |   kPCA       |  q: 25    sigma: 68.6649 | 0.9916 | 0.985 |
| Ionosphere  |   PCA       |  q: 22    | 0.6651 | 0.7036 |
| Ionosphere  |   ParzenWin       |  sigma: 0.6251    | 0.9839| **0.987** |
| Ionosphere  |  OCSVM       |  nu: 0.03 sigma: 0.7543    | 0.9832 | 0.7364 |
| Circles  |   kPCA       |  q: 25    sigma: 0.2947 | 1.0 | **1.0** |
| Circles  |   PCA       |  q: 1    | 0.5565 | 0.5183 |
| Circles  |   ParzenWin       |  sigma: 0.0176    | 0.9979| 0.9955 |
| Circles  |  OCSVM       |  nu: 0.03 sigma: 0.1389   | 1.0 | 0.9993 |
| Roll  |   kPCA       |  q: 53    sigma: 1.0985 | 0.9997 | **0.9986** |
| Roll  |   PCA       |  q: 1    | 0.5413 | 0.4163 |
| Roll  |   ParzenWin       |  sigma: 0.3556   | 0.9987| 0.9967 |
| Roll  |  OCSVM       |  nu: 0.35 sigma: 0.1389   | 0.9997 | 0.9993 |

#### Decision Boundary
For a visual illustration of thresholding in the two dimensional toy examples 'Roll' and 'Circiles', a decision boundary can be formed around the background data. To do so, a lattice sampling of surrounding points is passed through a detection algorithm and a 2D contour that passes through the threshold score is established. To establish this boundary threshold score the highest anomaly score amongst the background points in the validation set was used, so that points with a greater score during testing are considered anomalous. 

![alt text][CirclesB]
![alt text][RollB]

[CirclesB]:  https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Circles%20Boundary2.png "Circles Decision Boundary"
[RollB]:  https://github.com/Nmerrillvt/kPCA/blob/master/Figures/Roll%20Boundary2.png "Roll Decision Boundary"

### Discussion and Conclusion

An interesting result is that the three different kernel-based methods (kPCA, ParzenWindow, and OC-SVM) have similar regions of detection for the different values of sigma. For OC-SVM and kPCA, the choice of sigma dominates performance however there are different choices can significantly improve detection, as seen in the 'island' of higher AUC for kPCA in the Glass Validation.  

kPCA had the higher AUC across a greater number of datasets as Hoffman reports, but the split on the validation set shows one issue of generalization. The paramter setting for kPCA that were best during validation, were not necessarily the best during testing, as seen in the 'Glass' and, to a lesser extent, the 'Ionosphere' datasets . Additionally, a finer grid search than that shown in [[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)] allowed for a more competitive comparison between OC-SVM, kPCA, and the Parzen Window. On several datasets, each of these performed nearly perfectly, with corresponding AUCs very close to or equalling 1.0. 

Hoffman also produced a boundary comparison in his evaluation, however he used noisy data to train the fit the model. This evaluation shows the results with 'clean' data and using the same validation split. Following these methods the differences between kPCA and OC-SVM are less dramatic. However, there is still a clear advantage over the Parzen Window which shows a lack of generalization.

Overall, I was able to reproduce the results of Hoffman's paper and demonstrate the benfit of kPCA as a novelty detection algorithm. However the inclusion of a validation set for paramter tuning shows some potential issues for selecting the correct values of sigma and q. 


### References
[[1](https://www.sciencedirect.com/science/article/pii/S0031320306003414)] H. Hoﬀmann, “Kernel pca for novelty detection,” Pattern Recognition, vol. 40, no. 3, pp. 863 – 874, 2007.
1

[[2](https://www.ncbi.nlm.nih.gov/pubmed/27093601)] M. Goldstein and S. Uchida, “A comparative evaluation of unsupervised anomaly detection algorithms for multivariate data,” PLoS ONE, Apr 2016.

[[3](https://onlinelibrary.wiley.com/doi/abs/10.1002/bimj.4710290215)] G. Enderlein, “Hawkins, d. m.: Identiﬁcation of outliers. chapman and hall, london – new york 1980, 188 s., £ 14, 50,” Biometrical Journal, vol. 29, no. 2, pp. 198–198, 1987.

[[4](https://www.spiedigitallibrary.org/conference-proceedings-of-spie/10198/101980X/A-study-of-anomaly-detection-performance-as-a-function-of/10.1117/12.2264160.full?SSO=1)] C. C. Olson, M. Coyle, and T. Doster, “A study of anomaly detection performance as a function of relative spectral abundances for graph- and statistics-based detection algorithms,” in Algorithms and Technologies for Multispectral, Hyperspectral, and Ultraspectral Imagery XXIII (M. Velez-Reyes and D. W. Messinger, eds.), vol. 10198, pp. 309 – 320, International Society for Optics and Photonics, SPIE, 2017.

[[5](http://rduin.nl/papers/prl_99_svdd.pdf)] D. M. Tax and R. P. Duin, “Support vector domain description,” Pattern Recognition Letters, vol. 20, no. 11, pp. 1191 – 1199, 1999.

[[6](https://dl.acm.org/citation.cfm?id=2888116.2888129)] B. Shen, B.-D. Liu, Q. Wang, Y. Fang, and J. P. Allebach, “Sp-svm: Large margin classiﬁer for data on multiple manifolds,” in Proceedings of the Twenty-Ninth AAAI Conference on Artiﬁcial Intelligence, AAAI’15, pp. 2965–2971, AAAI Press, 2015.

[[7](https://dl.acm.org/citation.cfm?id=295960)] B. Sch¨olkopf, A. Smola, and K. Mu¨ller, “Nonlinear component analysis as a kernel eigenvalue problem,” Neural Computation, vol. 10, pp. 1299–1319, July 1998.

[[8](https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+%28Diagnostic%29)]  D. Dua and C. Graff, “UCI machine learning repository,” 2017.

[[9](http://yann.lecun.com/exdb/mnist/)]  Y. LeCun and C. Cortes, “MNIST handwritten digit database,” 2010

[[10](http://odds.cs.stonybrook.edu/)] S. Rayana, “Outlier detection datasets: ODDS,” 2016.

[[11](https://github.com/yzhao062/pyod)] Y. Zhao, Z. Nasrullah, and Z. Li, “Pyod: A python toolbox for scalable outlier detection,” Journal of Machine Learning Research, vol. 20, no. 96, pp. 1–7, 2019.

[[12](http://www.heikohoﬀmann.de/kpca.html)] http://www.heikohoﬀmann.de/kpca.html.

[[13](https://papers.nips.cc/paper/1723-support-vector-method-for-novelty-detection.pdf)] B. Scho¨lkopf, R. Williamson, A. Smola, J. Shawe-Taylor, and J. Platt, “Support vector method for novelty detection,” in Proceedings of the 12th International Conference on Neural Information Processing Systems, NIPS’99, (Cambridge, MA, USA), pp. 582–588, MIT Press, 1999.
