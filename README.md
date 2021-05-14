# walksignal
walksignal is a set of tools to plot OpenCellID data on street maps and other
images. The project was originally motivated by interest in comparing
real-world data to random walk models in the context of signal reception and
quality. It relies heavily on the use of freely-available mobile apps and
online services:

1. [Network Cell Info Lite](https://play.google.com/store/apps/details?id=com.wilysis.cellinfolite&hl=en_CA&gl=US)
2. [OpenStreetMap](https://www.openstreetmap.org)
3. [OpenCellID](https://www.opencellid.org)

## Usage

### Requirements

See [requirements.txt](requirements.txt) , or just run `pip install -r requirements.txt`.

At least one measurement set and one reference set are required for use.

#### The Measurement Set

In order to compile a dataset for use, the following things are required:

1. At least one set of OpenCellID data, such as that exported by the Network
   Cell Info Lite app
2. A map image exported from the OpenStreetMap site, which contains the area of
   interest (if plotting spatial data such as in the example)
3. A map boundary box, defined as min/max lat/long readings, defined in the
   same path as the map file and the data as "bbox.txt". This can be
   obtained on the same page as the export of the map image

Examples can be found in the `data` directory.

#### The Reference Data Set

A reference set is required to be set for analysis of the measurement
set. An example is provided in `data/oci_ref`

If a tower in the measurement set is not in the reference set, the Path
Loss plot will not display the measurement data until a tower position
is manually set. Doing this will draw the tower in the specified
location on the tower plot.

### Using gws

Run the following to start the GUI:
`./gws`

## Screenshots

### Path Loss

![path loss](example_pathloss.png?raw=true "Path Loss")

### Tower Map

![tower map](example_towermap.png?raw=true "Tower Map")

### Random Walk Model Plot

![random_walk](example_rwm.png?raw=true "Random Walk Model")

## Path Loss Models

The two path loss models provided originate in [[1]](#1) and are known
as the Alpha-Beta-Gamma (ABG) and the Close-In (CI) models,
respectively. They have the following expressions:

**ABG:**

`10 * alpha * np.log10(dist/ref_dist) + beta + 10 * gamma * np.log10(freq/ref_freq) + np.random.normal(0, sigma)`

**CI:**

`pl_fs(ref_dist, freq) + 10 * pl_exp * np.log10(dist/ref_dist) + np.random.normal(0, sigma)`

Where: 

- **alpha** is the distance-dependency coefficient
- **beta** is an offset value in dB
- **gamma** is the frequency-dependency coefficient
- **sigma** is the standard deviation for a zero-mean Gaussian random
  variable in dB
- **pl_exp** is the path loss exponent
- **dist** is the measurement distance
- **ref_dist** is the free space reference distance
- **freq** is the carrier frequency
- **ref_freq** is the reference frequency

It is important to note that while the alpha parameter has some physical
basis in the ABG model, the **beta** and **gamma** parameters are primarily for
curve-fitting[[1]](#1, p. 2847) and do not have obvious physical
origins. Also stated there are the conditions for equivalence between
the ABG and CI models:

- Equate the **alpha** and **pl_exp** values
- Set **gamma** to 2 (the free space **pl_exp** value)
- Set **beta** to `20 * np.log10((4 * np.pi * 10^9)/c)`, where **c** is
  the speed of light

## Random Walk Models

The random walk model is taken from [[2]](#2), specifically from
equation 13, which is a radio link expression for the power received by
an antenna at a distance r from the transmitter. The full expression is
too complex to list here, but it has the following parameters:

- **density** of the obstacles in the environment
- **absorption** absorptive rate of the environment
- **gain factor** accounting for TX power, gain of the TX antenna,
  effective area of the receiving antenna, and miscellaneous hardware
  loss in the link, in Wm^2
- **offset** to manually adjust the model curve by a factor in dB

## Challenges

- Accurate tower positioning info is still difficult to obtain - the
  Network Cell Info Lite app shows a tower icon at the location of the
  cell, not the tower itself[[3]](#3)
    - Means that there is outstanding effort remaining to collect full
      data set for analysis (i.e. find, photograph, and get approximate
      GPS position of each tower projecting to a cell)
- Data collection is somewhat erratic and inconsistent, as the same
  campaign route can provide significantly different results on
  subsequent runs
- Still an outstanding issue with accurate positioning in "street
  canyons" and even in low-density non-line-of-sight (NLOS) areas,
  leading to inaccurate positioning readings. This can be corrected with
  the addition of a windowing algorithm that "smoothes" the data via
  dead-reckoning
    - Existing datasets for more urban areas are still useful, but
      require more repeated measurement campaigns in the same locations
      to have sufficient numbers of data points for this to work
- Need to expand and compare this with other devices - a 5G-capable
  mobile device would be a valuable addition, but the Network Cell Info
  app does not clearly state support for 5G yet
- Measurements thus far do not adequately compare to the stochastic
  nature of the PL and RWM models. In [[1]](#1, p. 2847), it is stated
  that their experiments used 30 data sets to compare with the models,
  so more campaigns should be undertaken in the previously-visited areas
  to ensure level comparisons

## References

<a id="1">[1]</a> 
S. Sun, T. S. Rappaport, et al.,
*Investigation of Prediction Accuracy, Sensitivity, and
Parameter Stability of Large-Scale Propagation Path
Loss Models for 5G Wireless Communications*,
IEEE TRANSACTIONS ON VEHICULAR TECHNOLOGY, VOL. 65, NO. 5, MAY 2016

<a id="2">[2]</a> 
M. Franceschetti, J. Bruck, L. J. Schulman,
*A Random Walk Model of Wave Propagation*,
IEEE TRANSACTIONS ON ANTENNAS AND PROPAGATION, VOL. 52, NO. 5, MAY 2004

<a id="3">[3]</a>
M2Catalyst, LLC.,
*Network Cell Info: The Ultimate Network Cell Signal Information Tool*,
https://m2catalyst.com/apps/network-cell-info
