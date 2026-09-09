..  
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. role:: key
    :class: key



.. _align:

Preparing for XRR
=================


XRD mode of the photon delivery system
--------------------------------------

The very first step for setting up the goniometer for XRR is to move
the photo delivery system to the correct position for photo delivery
to the goniometer position.

This involves moving
 
+ the monochromator to the specified energy
+ the focusing mirror to the correct pitch and bend
+ the harmonic rejection mirror out of the way
+ the hutch slit assembly (:numref:`Section %s <blslits>`) to
  the correct height
+ the XAFS table to the correct height for supporting the
  flight path.

To set up the photon delivery system for scattering at 8600 eV:

.. code-block::

   RE(xrdmode())

or specify an energy:

.. code-block::

   RE(xrdmode(12000))

The single argument is the target energy in eV units.  The default is
8600 eV, the normal operating energy for experiments on the
goniometer.

This plan will look up the correct positions of all motors in the
`beamline lookup table
<https://github.com/NSLS2/bmm_tools/blob/main/src/bmm_tools/optics/mode_data.py>`__
and set all those axes moving to their correct positions.

Once all axes have arrived in position, a scan of the rocking curve of
the monochromator will be performed and ``dcm_pitch`` will be moved to
the peak of that scan.

Finally, the hutch slits will be opened wide, 7 mm wide by 1 mm tall,
allowing the beam size to be determined by the :numref:`gomiometer
slits (see Section %s) <goniometer_slits>`.

.. admonition:: Future tech!

   This plan will eventually be used to perform scattering
   measurements at any energy above 8000 eV without having to do a
   time-consuming realignment of the goniometer.

   To obtain consistency in lateral position of the focused beam,
   Bruce is working with BLOP team in DSSI to optimize ``dcm_roll``
   and the orientation of the focusing mirror to provide stable beam
   position over the energy range from 8 keV to 20 keV.  

   To obtain consistency in vertical position, a scan of the pitch of
   the focusing mirror into the goniometer slits will deliver
   consistent beam height.




Goniometer alignment strategy
-----------------------------

.. note:: A few things that are explicit steps in SPEC are handled
	  differently in Bluesky.  For example, the Mythen
	  ``full_mca``, ROI1, is set at Bluesky startup and does not
	  need to be explicitly set.  All alignment steps and
	  associated data processing are discussed in detail in
	  :numref:`Section %s <plans>`.

#. Place the Mythen in the most downstream position on the
   :olive:`(what is the arm called?)`. Measure and record the gap value
   |nd| typically around 90 mm.  See :numref:`Figure %s <fig-gap>` for a
   photo identifying what the gap is.  To record the gap in a way that
   the data acquisition software can use, do: 

   .. code-block:: python
   
      xrduser.gap = 90.0

#. Using the YAG camera, center the pin under the beam.

   a. Open the slits wide

      .. code-block:: python

	 RE(mv(slits.vsize, 4))
	 RE(mv(slits.hsize, 4))


   b. Adjust ``samplez`` to put the pin in the beam by seeing its shadow
      on the YAG.  

      .. code-block:: python

	 RE(mvr(samplez, <amount>))

   c. Mark the position of the pin in the beam
   d. Rotate ``phi`` stage by 180 degrees
   e. Mark pin again, then mark the geometric center of those two
      markings
   f. Move ``table.lateral`` so that the center of the two markings is in
      the center of the beam
   g. Rotate ``phi`` by -180 degrees to verfiy this alignment
   h. Rotate ``chi`` by -90 degrees: 

      .. code-block:: python

	 RE(mvr(chi, -90))

   i. Repeat steps (c) to (g) for this orientation
   j. Rotate ``chi`` back to 0 degrees: 

      .. code-block:: python

	 RE(mvr(chi, 90))

   .. admonition:: Future Tech!

      Automate the pin centering procedure using a camera that is
      supported by AreaDetector.  Automate the angle motions and
      determination of pin shadow positions.  Compute and move to
      target position in each direction.

#. Align the slits to be centered around the beam and define the 0 of
   each slit to be in the position that cuts the beam in half.  This is
   done by: 

   .. code-block:: python

      RE(align_slits())

   :numref:`See Section %s <slit_align>` for more details.

#. Set slit sizes: 

   .. code-block:: python

      RE(mv(slits.vsize, 0.15, slits.hsize, 1.0))

   This vertical size |nd| 150 |mu|\ m |nd| is considerably smaller
   than the focused beam, but appropriate for an XRR measurement.

#. Align the table in the beam:
  
   .. code-block:: python

      RE(linescan(table.vertical, 'monitor', -1, 1, 51))
      RE(linescan(table.lateral, 'monitor', -2, 2, 51))

#. Do a linescan (:numref:`Section %s <linescan>`) of the ``dethor``
   motor to center the Mythen around the beam in the horizontal
   direction. 
  
   .. code-block:: python

      RE(linescan(dethor, 'mythen', -3, 3, 61))

   :numref:`See Section %s <dethor_align>` for more details.

#. Perform the Mythen calibration scan:
  
   .. code-block:: python

      RE(mythen_calibration(-4, 1, 1001))

   This will set the bounds of the ``dir`` and ``refl`` ROIs and write
   a calibration report to the proposal folder.  It will also record
   the calibration parameters.  :numref:`See Section %s <mythen_cal>`
   for more details.

   .. admonition:: Question
      :class: attention

      What is the CHESS calibration?  This needs to be written.

#. Verify the alignment of beam, goniometer, and detector are
   acceptable by scanning the ``delta`` arm and plotting the signal
   from both ``dir`` and ``refl``.  The ``dir`` plot should be narrower
   than **and** well centered in the ``refl`` plot.

   .. code-block:: python

      RE(linescan(delta, 'mythen', -0.15, 0.15, 61))

You are now ready for sample alignment.

Sample alignment strategy
-------------------------

A sample for XRR is usually a large, flat wafer.  The correct
alignment has the sample surface parallel to the beam path and at a
height such that it blocks half the beam.  With that alignment, the
center of the beam will be on the center of the sample as the incident
angle changes and the beam will spread symmetrically over the length
of the sample as the angle changes.

.. todo:: Need example screenshots of the results of both sample
          alignment scans.

1. Start by aligning the sample vertically.

   .. code-block:: python

      RE(sample_vertical())

   This will run a linescan (:numref:`Section %s <linescan>`) of
   ``samplez`` against the signal in direct beam ROI then fit an error
   function to the measurement to find the position where the sample
   blocks half the beam.  That position will be defined as 0 of
   ``samplez`` by setting the EPICS offset accordingly.

2. Then align the pitch of the sample.

   .. code-block:: python

      RE(sample_eta())

   This will run a linescan (:numref:`Section %s <linescan>`) of
   ``eta`` against the signal in direct beam ROI then do an
   appropriate analysis (more discussion below) to find the zero of
   ``eta``.  Move to that position and define it as 0 by setting the
   EPICS offset accordingly.

3. Iterate those two steps as needed.

The interpretation of the pitch scan is a bit subtle.  In the case of
a very rough surface, the correct choice for ``eta`` will be very close
to the peak of the measured scan.

However, in the case of a very smooth sample, the total external
reflection will be intense enough that the structure near the peak
will be such that the maximum intensity is not necessarily the proper
0 of ``eta``.  In that case, a more elaborate analysis is required.

.. todo:: Fully explain the smooth sample algorithm once it is
          implemented in code.  Show the result of that analysis.


