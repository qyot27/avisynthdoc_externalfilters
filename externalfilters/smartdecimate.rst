
SmartDecimate
=============


Abstract
::::::::

| **author:** SmartDecimate by Kevin Atkinson
| **version:** v0.23 (C plugin, to be loaded with LoadCPlugin)
| **dowload:** `<http://www.avisynth.org/warpenterprises/>`_
| **category:** Deinterlacing & Pulldown Removal
| **requirements:** YV12 &  YUY2 Colorspace

--------

Smart Decimate removes telecine by combining telecine fields and decimating
at the same time, which is different from the traditional approach of
matching telecine frames and then removing duplicates.

| The latest version of Smart Decimate can be found at
| `<http://kevin.atkinson.dhs.org/tel/>`_.


.. toctree::
    :maxdepth: 3


Basic Usage
-----------

::

    LoadPlugin("...\avisynth_c.dll")
    LoadCPlugin("...\SmartDecimate.dll")
    AssumeTFF() # or AssumeBFF()
    SmartDecimate()

Note that the second plugin is LoadCPlugin not LoadPlugin. (There is a C after load)

**IT IS VERY IMPORTANT THAT THE FIELD ORDER IS CORRECT.**

Guide to Usage
--------------

For this guide I will assume that your material is 3:2 pulldown with some
possible video mixed in. However, SmartDecimate can handle any fixed
decimation ratio.

To use this filter, you must first determine the correct field order. If the
field order is wrong, my filter will not work correctly. To determine the
field order:
::

    AviSource(...)
    AssumeTFF()
    Bob()

Now preview the video and look for backwards motion. If you don't see any, then you should use "assumetff()". If you do see any, try:
::

    AviSource(...)
    AssumeBFF()
    Bob()

And once again look for backwards motion. If you don't see any this time, then you should use "assumebff()".

Now its time to set up my filter:
::

    LoadPlugin("...\avisynth_c.dll")
    LoadCPlugin("...\smartdecimate.dll") # notice the C in loadCplugin
    AviSource(...)
    AssumeTFF() # or AssumeBFF() as determined above
    SmartDecimate()

If you are happy with the result, then that is all that is needed. If not, read on.

If your material is anime with lots of repeated frames, then you should try:
::

    SmartDecimate(tel=0.60)

This should avoid any bobbing on long sequences with no motion.

The "tel" value controls how aggressive the filter is when matching fields.
The default value is 0.50.

If you have a mixed video with a large amount of true video, then you might
want to lower "tel" to as low as 0.25. This will risk bobbing some
progressive frames, but should not weave true video frames, which may leave
combing artifacts.

If you have a video that is pure 3:2 pulldown with very few true video
frames, then you might try raising "tel" to as high as 0.75. This will cause
SmartDecimate to be really aggressive when matching progressive fields.
However, it will likely end up leaving serious combing artifacts behind if
there is any pure video in your clip.

If you have a really noisy video, then you might need to raise the noise
factor such as:
::

    SmartDecimate(noise=0.80)

The default value is 0.50. Leave
it at the default unless your clip is very noisy. Higher values of noise will
often leave behind more combing artifacts for pure video frames.

If you have a really clean clip and you notice some combing artifacts in pure
video frames, then you might want to lower the noise factor. But be careful;
if it is too low, SmartDecimate won't be able to match the progressive fields
and will thus bob them.

Finally, if you have a hybrid video (mixed 3:2 pulldown and true video), you
should consider replacing the dumb bob with something like DgBob or
SmoothDeinterlace. To do this, use the "bob" option. For example:
::

    SmartDecimate(bob=DgBob(...))

This will greatly improve the quality of the
pure video frames. Note that the bob source only affects how bobbed frames
are rendered. It is not used to compare fields. Nor is it used by any other
part of SmartDecimate.

Filter Reference
----------------

The filter will work with either YUY2 or YV12 input.

The usage is:

``SmartDecimate`` ([numr, denm], options)

The numerator and denominator are for the decimation ratio for the video
**after** it has been separated into fields. The default is 24 and 60 for
numr and denm respectfully.

The following options may be used to fine tune ``SmartDecimate``:

| **bob**
| an alternative source of bob deinterlaced frames. The default is "``Bob()``"
  which is AviSynths built in dumb bob filter. For better results, use a smart
  bob such as `DgBob`_ or SmoothDeinterlace. The bobbed source is used
  whenever SmartDecimate determines that a field is not part of a progressive
  frame.

| **weave**
| an alternate source of weaved frames. The default is "``DoubleWeave()``".

| **tel**
| a number between 0 and 1 which controls how aggressive the filter is when
  matching fields. 0.50 (the default) will work well with most clips. The
  higher the value, the more risk there is of leaving combing artifacts in true
  interlaced material. The lower the value the more risk there is to bobbing
  (or in the extreme case skipping or duplicating) Telecine frames.

Currently tel  toggles various internal options depending on what it is set
to. Currently there are switches at: 0.45, 0.55, 0.65 and 0.72.

| **noise**
| The noise factor. The default value, 0.50, should work in most cases.

| **t_max**
| An alternate method of setting the noise factor. You need to understand how
  my filter works in order to use it.

| **cpu**
| Force the CPU to a particular type. It is normally auto detected. To see if
  it is detected correctly turn logging on. The first line of the output will
  display the CPU type. Current valid values for this value are:

- 0 - Generic
- 2 - Integer SSE
- 3 - SSE
- 4 - SSE2

| **unaligned**
| Allow reading of unaligned data. Normally ``SmartDecimate`` may ignore a few
  pixels at the beginning or end of each row, so that reads are aligned nicely.
  Setting this value to true will prevent this.

The following options can be used to control the printing of useful
information:

| **log_level**
| The verbosity of the information printed. Default is 2.

| **log_file**
| If set, all output will be appended to the filename specified.

| **console**
| If set to true, then a console window will pop up and all output will be sent to it.

| **debug_print**
| If set, then all output will be printed using the OutputDebugString system
  call. You can view this output with a utility such as DebugView.

The meaning of the information printed is as follows:

-   "Reseting to NUM" means that the internal variables are reset and any
    patterns are reset. This will happen when ever the video is accessed in a
    non-linear way.
-   "FRAME 3 = [3, 4]" indicates how frame number 3 (for example) is
    being rendered. Here 3 indicates the primary source field that is chosen.
    If the frame is to be rendered by weaving two fields together, the
    numbers in [] indicate which source fields are chosen. If the frame is to
    be bob deinterlaced that the [] will instead be "BOB".
-   "Diff NUM: WHAT DIFF" indicates the difference between to source
    field NUM and NUM+1. WHAT is the classification of the difference with 0
    meaning the same, 1 similar, and 2 different. DIFF is the numerical
    difference.

The following options may be used to tune internal parameters. In order to
understand what they do, you will need to look at the source. They may be
changed between releases.

-   max_last_set - positive integer
-   t1_def, t2_def, t1_max, t2_max - float between 0.0 and 1.0
-   floor - float between 0.0 and 1.0
-   floor_adj = float larger than 0.0
-   sml_peak, lrg_peak - float larger than 1.0
-   sml_tail_diff, lrg_tail_diff - float larger than 0.0


Understanding how SmartDecimate Works
-------------------------------------

This discussion assumes that the reader is familiar with interlaced video,
and knows the meaning of such terms as frame, field, top field, bottom field,
BFF, TFF, telecide, telecine, 3:2 pulldown, etc..

Other telecide filters generally remove telecine in a two step process. First
telecine fields are matched up but the frame rate is not changed, then
duplicate frames are removed. SmartDecimate does not work this way.
SmartDecimate instead, combines telecine fields and decimates the frame rate
at the same time.

The very first thing SmartDecimate does is to use "SeparateFields()", which
will split the video into individual fields, which will also double the frame
rate. Therefore field number 9 really is the bottom field of frame 4 (assume
the video is TFF) before separating it into fields. For simplicity, I will
always refer to fields from the source video in this way.

SmartDecimate then selects fields in a regular pattern, trying to avoid
selecting duplicate fields. More precisely, let N be the destination frame
number and R be the ratio which for 3:2 pulldown will be 24/60 = 2/5, then
SmartDecimate choses between field floor(N/R) and floor(N/R) + 1. For
example, for destination frame number 5, using typical 3:2 pulldown,
SmartDecimate will chose between field 12 and 13. Which one it choses is
rather complicated, and something I will not get into here.

After a source field is chosen, it needs to decide how to render it.
SmartDecimate choses between: 1) matching it up with the previous field, 2)
matching it up with the next field, or 3) bob deinterlacing it. Which one it
choses is based on how similar the field is to the previous or next field. If
the field is the same as the previous or next field it will match it up with
that field by weaving the two fields together. If it can't find a matching
field, it will bob deinterlace the current field. SmartDecimate doesn't
actually do the final rendering. Instead it uses anther AviSynth filter to do
the work for it. In the case of weaving it will use DoubleWeave(), and in the
case of bobbing it will use Bob() or a smart bob filter if one is provided.

Tuning T_Max
------------

In order to discover which fields are different from each other and which are
the same, SmartDecimate looks for peaks in the difference string. That is
given three values representing the difference between four consecutive
fields A B C D, if BC (ie the difference between B and C) > AB and BC > CD
and AB and CD are close in value than the fields A and B are likely to be the
same, fields B and C are likely to be different, and fields C and D are
likely to be the same.

However, this method is not perfect and can sometimes classify high motion
scenes when similar difference values between frames (that is AB and CD are
the similar) as being the same. So, to control this SmartDecimate simply
assumes all fields with a difference greater than a fixed threshold can not
possibly be the same. Unfortunately there is no optimal value for this
threshold so it is set at a reasonable value which should do well on most
clips that are not extremely noisy. However, this value will generally let
some different fields through which will lead to combing artifacts. To avoid
this, the threshold, "t_max", should be set as low as possible.

To discover what the best value for "t_max" is for a particular clip you will
need to know what the differences between frames are. The easiest way to do
this is to set the "log_file" option (with the log level set at 2 or higher)
and run SmartDecimate on a significant portion of your clip. Once that is
done you should see something like the following in the log file:
::

    ...
    Diff 827: 0 6.11586e-008
    FRAME 331 = [828,827]
    Diff 828: 2 8.21554e-005
    Diff 829: 0 3.69098e-008
    Diff 830: 0 3.27322e-008
    FRAME 332 = [830,831]
    Diff 831: 2 0.000124936
    Diff 832: 0 6.48069e-008
    FRAME 333 = [832,833]
    Diff 833: 2 0.000102379
    Diff 834: 0 3.54472e-008
    ...

The lines you are interested in are the ones that begin with
"Diff ...". The first number after the Diff is the source field number.
The second number is the classification of the difference with 0 meaning
the same, 1 similar, and 2 different. The final number is the actual
difference. What you are interested in is the difference of frames
classified as the same. You want to set t_max to slightly above the
maximum value of all differences classified as the same. For this clip
7.0e-8 may be a good value. But since I only looked at a small portion of
the clip it may need to be higher as differences can vary by a large
amount. It is best to look at the value for several different areas of
the clip to get a safe value. Assuming 7.0e-8 is a good value I can use
it as follows:
::

    SmartDecimate(t_max=0.000000070)

I wrote 0.000000070 instead of 7.0e-8 because AviSynth does not seem to
support scientific notation. To be sure that you converted the number
correctly set the log level to 3 or higher and look for a line like:
::

    t1_max = 7.00000e-008
    t2_max = 2.10000e-007

max_last_set = 13 The value you are interested in
is t1_max which is is 7e-8. Thus I converted the number correctly. The
other values are for other internal thresholds.

Once you think you've found a good value for t_max, rerun your clip though
SmartDecimate with a log level set at 3 or higher to make sure that you did
not set it too low. If you set it to low an excessive number of frames will
be bobbed and you will see messages such as:
::

    2001: T1 Maxed Out at 2.00000e-008

These messages are normal for the true video parts of your clip
but should not be seen in the telecine parts of your clip. If you see them,
it means t_max is to small and needs to be raised. The exact value can be
discovered by looking at the differences for the surrounding frames. For
example:
::

    ...
    Diff 1999: 2 3.08506e-008
    Diff 2000: 1 4.03734e-008
    Diff 2001: 2 8.89810e-006
    Diff 2002: 1 2.93221e-008
    ...

indicates that t_max should be at least 4.04e-8, but 4.5e-8 would be a safer value.

Finally, please note that "t_max" and "noise" both control the same internal
parameter which is "t1_max". They just do it in a different way. "t_max" sets
it directly while "noise" sets it indirectly based on an exponential formula.
As as SmartDecimate 0.21 this formula is:
::

    t1_max = exp(17.65*noise - 20.71)

But the exact parameters can change between any release. The idea is that a
noise value of 0.50 (the default) should work well with most clips while 0.80
can be used for really noisy clips, etc.

Using a Post Deinterlacer
-------------------------

Ideally, the fields rendered by SmartDecimate should not need to be
deinterlaced if the parameters are properly tuned. Realistically, a post
deinterlacer might be useful. However, since SmartDecimate works on the field
rather than the frame level there are some things you should be aware of.

Most deinterlacers work by always choosing the top field (or perhaps the
bottom) as the dominate field and selectively throwing information in the
other field out and then interpolating or blending. For normal 30 fps (or 25
fps pal) this is the correct thing to do. However, for the output of
SmartDecimate, this is not correct because the dominant field is not always
the same. It will depend on which field SmartDecimate originally chose from
the source video, and can either be the top or the bottom field. Thus, a
traditional deinterlacer might throw out the wrong field. This might not even
be noticeable, but you should be aware of it.

Authors of deinterlace filers can correct this problem by paying attention to
the parity of the frame. SmartDecimate will always report the parity of the
original field chosen as the parity of the final rendered frame. If the
parity is true, then the top field should be chosen. If it is false, the
bottom field. I will also be willing to pass hints on to the deinterlacer to
indicate which fields are bobbed if someone will tell me how.

Dealing With Unwanted Motion
----------------------------

By using the weave option, you can use different clips for the input and the
output. This is useful for dealing with subtitles or other motion that you do
not want to be considered when matching frames. For example, to crop off
subtitles you might want to try.
::

    b = Bob()
    w = DoubleWeave()
    CropBottom(64)
    SmartDecimate(bob = b, weave = w)

Then, the output video will not be
cropped, but the last 64 lines will not be seen by SmartDecimate. If you
use this method, then the bob source must also be specified. If you don't
specify a bob source, then it will attempt to use the input clip for
"Bob()". But the resulting clip will not have the same dimensions as the
weave source.

Smart Decimate vs Decomb
------------------------

Before `Decomb 5`_ by Donald Graft (aka Neuron2), traditional telecine
filters (that work by first matching telecine fields up and then decimating
by removing duplicate frames) had a tendency to duplicate frames. With Decomb
5 Donald seems to have that problem solved. Nevertheless, the SmartDecimate
approach does have a number of advantages over the Decomb approach and vice
versa. So neither approach is necessarily better than the other. Which filter
you use depends on the source material and the target frame rate.


Clean 3:2 pulldown material
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Both Smart Decimate and Decomb will work great. Decomb may still leave some
duplicate frames for highly irregular 3:2 pulldown material. Smart Decimate
on the other hand, by the nature of how it works, will generally not, but it
may end up bobbing some frames that it shouldn't. In my informal tests Smart
Decimate is slightly faster than Decomb when Decomb's post processing is
turned off. With post processing Smart Decimate is a lot faster.


Hybrid 3:2 pulldown and interlaced video
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If for whatever reason you wish to leave the video at 30fps, then use Decomb.
Smart Decimate will be able to match telecine frames up when going from 60 ->
30fps but it will also duplicate frames. Decomb has a special mode to deal
with this by blurring frames together, which can lead to smoother motion.

If your target frame rate is 24fps, then the choice is based on personal
taste, as both Smart Decimate and Decomb handle the situation completely
differently, and with very different results.

Smart Decimate will handle video by selecting frames from a bobbed source
(twice the frame rate) in a 3:2 pattern. That is, it goes from 60 -> 24fps.
This leads to fairly smooth motion without having to resort to blurring
frames together. However, the video is still slightly jerky. In my
experience, the slight jerkiness is generally not noticeable except when
there is scrolling text. For example, when decimating scrolling credits in
this fashion, the results are awful. Since bobbed frames are used, you will
also have some bobbing artifacts. The results will not be very good unless a
smart bob is used.

Decomb, on the other hand, will try to decimate true video by going from a
deinterlaced 30 fps source to 24 fps. It does this in one of two ways. The
first thing it can do is to simply drop one in every 5 frames. This approach
will lead to extremely jerky results. The other thing it can do it is to
blend frames together. This approach will lead to smoother video than the
Smart Decimate approach, but not without undesirable artifacts. Moving areas
of the image will generally be blurry. With very high motion, double images
can appear which can make it hard for the eye to follow the motion. Blurring
frames together also negatively effects compression because it makes
following motion more difficult.


Hybrid with 30 fps progressive
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your video has a decent amount of true 30 fps progressive material in it,
then you're better off with Decomb. By the nature of how my filter works, it
will drop one frame in every 5 when converting it to 24 fps.

Decomb on the other hand sees interlaced video and 30 fps progressive as the
same thing and will thus be able to blur frames together for smother motion.
If your clip is mostly video/30 fps progressive with some 3:2 pulldown Decomb
can also keep it at 30 fps and blend the pulldown material for better
results.


Strange Telecine Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~

If your telecine pattern is not 3:2 pulldown (or simple 25 fps progressive in
the case of pal) and you want to attempt to restore it to its original frame
rate you should try both Smart Decimate or Decomb and see which one works out
better. I have no experience in this area, so I really don't know.


Other Things to Try with Smart Decimate
---------------------------------------

Here are some other things you might want to try with Smart Decimate that
can't be done using traditional telecine filters such as Decomb.


Better Bob of Telecine Material
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your material contains any telecine material and you want to bob
deinterlace it to get 60 fps than you should try:
::

    SmartDecimate(1,1,tel=0.25, bob=DgBob(...))

which will lead to better results than using DgBob
(or most other smart bob filters) alone, since telecine frames are duplicated
rather than being bob deinterlaced. This could make a major difference with
anime.


Dealing with Telecine with Blurred Frames
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are dealing with a lot of blurred frames, here is something to try.
Use SmartDecimate(1,1) and then attempt to pick out the non-blurred frames by
only choosing frames that are duplicated at least once. You might be able to
do the latter step with Decimate() from the Decomb package, or
`MultiDecimate`_. I have no idea how well this will work as I have never
tried it but I thought I would mention it in case someone is interested in
pursuing it.

Caveats
-------

When MMX/SSE optimization is used SmartDecimate may ignore up to 6 (14 for
SSE2) pixels in the beginning or end of each row so that memory reads are
aligned. If this bothers you, then set "unaligned" to true, which will force
SmartDecimate to use the non-optimized code when reading frame data.

SmartDecimate will warn about any pixels ignored if the logging level is set
at level 0 or higher.

Compiling
---------

To compile SmartDecimate you will need to install Gcc and Nasm, and perhaps
GNU Make. I used `MinGW`_ (2.0.0-3) with MSYS 1.09, Gcc 3.3.1, and `Nasm`_
0.98.37. Except for Nasm, all of the required utilities can be found on the
`MinGW Download page`_. Other configurations should work, but you may need to
edit the Makefile. Once all the proper tools are installed and in the path,
simply type:
::

    make

from the MSYS shell and that is all that should be required.

Final Words
-----------

Feedback more than appreciated. Please send it to kevin.tel at
atkinson.dhs.org.

+---------------------------------------------------------------------------------------------------+
| ChangeLog                                                                                         |
+======+==============+=============================================================================+
| 0.23 | Sep 16, 2003 | - Bug Fixed.                                                                |
|      |              | - Slightly improved the field matching algorithm.                           |
+------+--------------+-----------------------------------------------------------------------------+
| 0.22 | Sep 12, 2003 | - Bug fixes.                                                                |
|      |              | - Fixed typos in documentation thanks to Tom Daniel.                        |
+------+--------------+-----------------------------------------------------------------------------+
| 0.21 | Sep 7, 2003  | - Greatly expanded the documentation to give users a better idea of how     |
|      |              |   SmartDecimate works and how to use it to its maxium potential.            |
|      |              | - The CPU type is now auto detected. SSE2 support is also provided.         |
|      |              | - Bug fixes, especially when working with unaligned data.                   |
|      |              | - No longer ignores the last couple of rows (in some cases as many as       |
|      |              |   11) of each frame                                                         |
|      |              | - Now use DoubleWeave() for rendering weaved frames rather than doing       |
|      |              |   it itself.                                                                |
|      |              | - When stepping through the video, one may now step backwards up to 60      |
|      |              |   frames or so without causing the pattern to be reset.                     |
|      |              | - Other minor changes.                                                      |
+------+--------------+-----------------------------------------------------------------------------+
| 0.20 | Aug 29, 2003 | - Expanded documentation with a short guide to using SmartDecimate.         |
|      |              | - Uses an improved field matching algorithm.                                |
|      |              | - Bug fix in the non-optimized version when dealing with yv12 input.        |
+------+--------------+-----------------------------------------------------------------------------+
| 0.12 | Aug 22, 2003 | - Can now control the output that went to the console on previous versions. |
|      |              | - Can now modify some of the internal paramaters.                           |
|      |              | - An SSE optimized version is now provided.                                 |
+------+--------------+-----------------------------------------------------------------------------+
| 0.11 | Aug 18, 2003 | - Greatly improved the classification algorithm                             |
+------+--------------+-----------------------------------------------------------------------------+
| 0.10 | Aug 16, 2003 | - Initial Release                                                           |
+------+--------------+-----------------------------------------------------------------------------+

$Date: 2004/08/17 20:31:19 $

.. _DgBob: dgbob.rst
.. _Decomb 5: http://neuron2.net/decomb/decombnew.html
.. _MultiDecimate: http://neuron2.net/multidecimate/multidecimate.html
.. _MinGW: http://www.mingw.org/
.. _Nasm: http://nasm.sourceforge.net/
.. _MinGW Download page: http://www.mingw.org/download.shtml
