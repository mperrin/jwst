Description
===========
At its basic level this step flat-fields an input science data set by dividing
by a flat-field reference image. In particular, the SCI array from the
flat-field reference file is divided into both the SCI and ERR arrays of the
science data set, and the flat-field DQ array is combined with the science DQ
array using a bitwise OR operation. Details for particular modes are given
in the sections below.

Upon completion of the step, the step status keyword "S_FLAT" gets set
to "COMPLETE" in the output science data.

Imaging and Non-NIRSpec Spectroscopic Data
------------------------------------------
Simple imaging data, usually in the form of an ImageModel, and many
spectroscopic modes, use a straight-forward approach that involves applying
a single flat-field reference file to the science image. The spectroscopic
modes included in this category are NIRCam WFSS and Time-Series Grism,
NIRISS WFSS and SOSS, MIRI MRS and LRS. All of these modes are processed
as follows:

- If the science data have been taken using a subarray and the flat-field
  reference file is a full-frame image, extract the corresponding subarray
  region from the flat-field data.

- Find pixels that have a value of NaN or zero in the FLAT reference file
  SCI array and set their DQ values to "NO_FLAT_FIELD."

- Reset the values of pixels in the flat that have DQ="NO_FLAT_FIELD" to
  1.0, so that they have no effect when applied to the science data.

- Apply the flat by dividing it into the science exposure SCI and ERR arrays.

- Propagate the FLAT reference file DQ values into the science exposure
  DQ array using a bitwise OR operation.

Multi-integration datasets ("_rateints.fits" products), which are common
for modes like NIRCam Time-Series Grism, NIRISS SOSS, and MIRI LRS Slitless,
are handled by applying the above flat-field procedures to each integration.

NIRSpec Spectroscopic Data
--------------------------
Flat-fielding of NIRSpec spectrographic data differs from other modes
in that the flat-field array that will be applied to the science data
is not read directly from CRDS.  This is because the flat-field varies with
wavelength and the wavelength of light that falls on any given pixel
depends on the mode and which slits are open.  The flat-field array
that is divided into the SCI and ERR arrays is constructed on-the-fly
by extracting the relevant section from the reference files, and then --
for each pixel -- interpolating to the appropriate wavelength for that
pixel.  This interpolation requires knowledge of the dispersion direction,
which is read from keyword "DISPAXIS."  See the Reference File section for
further details.

For NIRSpec Fixed-Slit and MOS exposures, an on-the-fly flat-field is
constructed to match each of the slits/slitlets contained in the science
exposure. For NIRSpec IFU exposures, a single full-frame flat-field is
constructed, which is applied to the entire science image.

NIRSpec NRS_BRIGHTOBJ data are processed just like NIRSpec Fixed-Slit
data, except that NRS_BRIGHTOBJ data are stored in a CubeModel,
rather than a MultiSlitModel.  A 2-D flat-field image is constructed
on-the-fly as usual, but this image is then divided into each plane of
the 3-D SCI and ERR arrays.

In all cases, there is a step option that allows for saving the
on-the-fly flat field to a file, if desired.

Error Propagation
-----------------
The VAR_POISSON and VAR_RNOISE variance arrays of the science exposure
are divided by the square of the flat-field value for each pixel.
A flat-field variance array, VAR_FLAT, is created from the science exposure
and flat-field reference file data using the following formula:

.. math::
   VAR\_FLAT = ( SCI_{science}^{2} / SCI_{flat}^{2} ) * ERR_{flat}^{2}

The total ERR array in the science exposure is updated as the square root
of the quadratic sum of VAR_POISSON, VAR_RNOISE, and VAR_FLAT.
