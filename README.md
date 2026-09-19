This is a derivative pipeline made based on Filippo Biondi's Synthetic Aperture RADAR processing method to detect micro-motion on ground surfaces and reconstruct its sub-surface origin. Biondi,
together with co-authors published components of the method in several papers and submitted the entire pipeline for patenting. However, to reproduce the method requires several pieces of information
that were not obvious or explicitly detailed in these publications. This attempt, currently in its 5th version, has been put together to bridge those gaps empirically using CHAT-GPT operating Python.
In order to load the protocol you need the .json file and the document protocol file. To run a real data experiment, you also need a Single Complex Look format SAR data file and a SICD format version. 
The other files are explanatory. We have added several features not mentioned by Biondi to either bridge gaps or improve the protocol. They are:
1) Our development was based on a limited aperture Capella Space Inc. data file. Because of the short aperture, we generated a total of 50 sub-apertures using Biondi's frame-shifting Reference/Offset
   Doppler band pass filter design. We don't know how many sub-apertures Biondi uses across his entire full length synthetic aperture. Empirically we determines a B-shift of 88 Hz, but this detail
   was also missing from Biondi's published method details.
2) We used a co-registration protocol called un-normalized Phase Cross-Correlation on a 32x32 pixel patch centered on each spatially reconstructed (inverse Fourier Transform) pixel of our Reference
   and Offset pairs KR and KO. The sub-pixel over-sampling rate is 100x. Biondi appears to be using a different co-registration protocol. We empirically found that u-PCC best preserves displacement
   vector trajectories that sequentially follow elliptical excursions moving through the sub-aperture stack from K=1 to K=50.
3) We developed a custom-made flank-regional random speckle and scatter effect correction on target pixels' raw displacement vector trajectories choosing an arc of neighboring pixels 40-pixel apart
   from target and sampled therefrom.
4) We scanned vector families for harmonic elliptical excursion modes 1-24 to isolate non-random spatially co-registered pixel displacement trajectories that obey elliptical paths.
5) We further gated this selection process by creating a more stringent second protocol branch that selects only resonance modes 1-6.
6) We use a 25 sub-aperture vector family frame to look for matches with the Hermitian Adjoint (conjugate transpose) of a phase histories v. depth matrix called Steering Matrix.
7) To sharpen non-random depth focus features we empirically identified a more selective variant which requires contiguous matching over 2 pixels at +/- 3 meters. "2-window ±3 m Consensus Sharpening."

We discovered, for example, that when processing signals from stepped pyramid walls depth focused tomograms showed ghosting, i.e. the reflective properties of the steps introduced an amplitude-mediated 
effect on phase that created artifacts. The wall signal disappeared when we introduced the more stringent resonance modal gate, i.e. only modes 1-6, not 7-24. 


Addenda:
1) We now have an improved co-registration method that overcomes an issue we detected when attempting to recover a synthetic spatial shift we devised. We found that range recovery was nearly complete,
   but azimuth recovery was not. The issue only arose when shifting the image from under the 32x32 patch, not when shifting the patch itself over the image. The solution was to frame the 32x32 patch on
   all sides with a 128-pixel frame that provides image context to the 32x32 patch. The way this works now is that the operator asks, what shift of the patch region over the Offset image, at the sub-pixel level, most closely
   approximates the same image region of the Reference-image. That closest vector distance, range and azimuth, is the displacement vector for a given K-pair at a given pixel. Each pixel is evaluated in this way
   for each sub-aperture pair to extract this displacement vector.

   Therefore, each pixel is associated with a family of these displacement vectors, equal to the number of sub-aperture looks. The next step in the protocol then examines these vector families for elliptical
   structure and assigns a modal score between 1 and an upper limit we define as at least four vectors per cycle. For example, a W=100 means there are 100 displacement vectors in the frame to search for elliptical
   structure than must be defined by at least 4 vectors such that there cannot be more than 100/4=25 elliptical cycles in that frame of 100 vectors. The nomenclature to indicate that is W=100, m=1-25. By that logic,
   m=1 means a single elliptical cycle defined by a 100-displacement-vector-station trajectory. This structural spatial (range-azimuth) vector field analysis adds a discriminatory layer to the protocol to detect
   real motion and distinguish from random pixel instability due to non-ground motion factors. 
