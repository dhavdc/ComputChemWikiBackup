---
title: Figure making / Plotting
description: 
published: true
date: 2020-11-13T16:58:31.628Z
tags: 
editor: markdown
dateCreated: 2020-08-25T16:35:18.492Z
---

# How to make publication quality figures

## Get yourself ready... 
If this is your first time making publication quaility figures get ready to spend a good amount of time to make the figure and be open minded to many iterations of figure modifications. Remember once the figure is published it is out there forever and no one wants to look back on their work and think "I wish I would have..."


## Software 
As of right now the lab uses MatPlotLib (In jupyter notebooks), Inkscape, and Pymol to generate publication qualitiy figures. This has not always been the case so feel free to use new tools, but the rest of this tutorial will focus on making a figure with these three tools. 


## Making the Figure
Take a look at this example of a pretty basic figure from one of our publications. I will go through different aspected of how this figure was created and things to keep in mind when you are making your own figure. 

![screen_shot_2020-11-05_at_2.01.47_pm.png](/screen_shot_2020-11-05_at_2.01.47_pm.png)

### MatPlotLib

Plotting the data is achieved by creating a jupyter notebook to read in your data and plot it. 

__Libraries to use in your Jupyter Notebook__
```
import matplotlib.pyplot as plt
from matplotlib.ticker import AutoMinorLocator
```
Here are the basic libraries that you will want to import to generate plots for your data. Naturally there will be other libraries that you will want to import for loading data and getting it ready to be plotted, but you can decide how you prefer to do that. 

__Setting up Global Plot Settings__

```
plt.rcParams['axes.linewidth'] = 1
plt.rcParams['xtick.major.size'] = 4
plt.rcParams['xtick.major.width'] = 1
plt.rcParams['xtick.minor.size'] = 2
plt.rcParams['xtick.minor.width'] = 1
plt.rcParams['ytick.major.size'] = 4
plt.rcParams['ytick.major.width'] = 1
plt.rcParams['ytick.minor.size'] = 2
plt.rcParams['ytick.minor.width'] = 1
plt.rcParams['savefig.pad_inches'] = 0
```

These global plot setting are what I have used to generate every plot. The will set the linewidths to 1pt and and set up the size the the X and Y tick marks.

__Choosing Colors for your Plots__ 

This, although seems like a minor task, is very important for creating easy to read and drawing your readers eye to the data you want to highlight. 

Here is the website I use to get ideas and pick colors:
https://htmlcolorcodes.com/color-picker/

![screen_shot_2020-11-05_at_11.22.26_am.png](/screen_shot_2020-11-05_at_11.22.26_am.png)

This website will allow you to explore different colors and find color harmonies so that when you find a color you like you can determine the best colors for the rest of the lines/points on the plot. Once I have chosen the colors I want to use I make a list of them in my jupyter notebook MatPlotLib will take the hex code directly. 

__Making the Plot__

The plot you output will be without labels and needs to be the proper size. Keep in mind a single column figure is 3.3 in and a double column figure is 6.5 in, thus a figure without labels will likely be 1.4 - 2.0 in wide and 0.9 - 1.4 in tall. Overcourse the individual plot size can vary depending on what you are trying to plot. 

For the above plot these are the sizes that were used.
```
ink_x = 1.5 # in.
ink_y = 0.9 # in.

ink_markersize  = 2 # pts.
ink_linewidth = 0.5 # pts.
```

Here is an example on how to make a basic labeless plot.

```
plt.close('all')
fig, axes = plt.subplots(figsize=(ink_x, ink_y))
fig.set_facecolor('white')

# plot points
for c, i in enumerate([2, 3, 4]):
    axes.plot([x[0] for x in data_2FE8], [y[i] for y in data_2FE8], 'o-', color=Colors[c], markersize=ink_markersize, linewidth=ink_linewidth, clip_on=False)

# plot details
axes.spines['top'].set_visible(False)
axes.spines['right'].set_visible(False)

# Y-Axis 
axes.set_ylim(0, 1)
axes.set_yticks([0, 0.5, 1.0])
axes.yaxis.set_minor_locator(AutoMinorLocator(2))
axes.tick_params(labelleft=False)

# X-Axis
axes.set_xlim(4, 9)
axes.set_xticks([4.5, 6.5, 8.5])
axes.xaxis.set_minor_locator(AutoMinorLocator(2))
axes.tick_params(labelbottom=False)

fig.tight_layout()
plt.savefig("plots/2FE8_Triad_HBs_wo_Labels.png", dpi=360, transparent=True)
plt.show()
```

Notice that the resulting plot has a transparent background and made with minor ticks. At this point you have a plot ready to be uploaded by inkscape.

### Pymol 

Making pymol figures is a slow and tedious task, but with a little creativity can produce textbook quality images. Remember to save frequently when figures get very complicated or you are new to pymol its not difficult to crash. This tutorial assumes you, for the most part know, a little about pymol and can load in PDBs and change how things look like showing the protein in cartoon, residues in sticks, and changing colors.

__Tried and True Standard Settings__
Enter the following commands into pymol. 

```
bg_color white
set cartoon_oval_length, 1.2
set cartoon_oval_width, 0.15
set cartoon_rect_width, 1.4
set cartoon_rect_width, 0.3
set ray_trace_mode, 1 
set ray_opaque_background, off
set ray_trace_gain, 0.15
```

These setting I have found to be consistly liked and have used on most publications. Overcourse there are always expection but these settings make a good starting point. 

__Select vs. Create__

Pymol is object oriented and using 'select' and 'create' are critical for creating nice images. The 'create' command allows you to make a new object in pymol and is what you will primarily use when you want to show something. The 'select' command is only a selection of an object so anything that happens to the object will also happen to the selection, thus be carful with selections.

__Should I show hydrogens?__

Given the nature of our simulation work it is important to know when to show hydrogens and when not to. For the most part you will not want to show any hydrogens on residues unless you need to show a specific protonation state or a hydrogen involved in a hydrogen bond. Naturally the residues you will want to consider most often are Asp, Glu, and His (HSP in our case). Some classic residues selection are as follows.


Aspartic acid deprotonated example (backbone not shown):
```
create asp_example, [object] and resid [resid #] and resname ASP and not name CA+C+N+O+H*
```
Aspartic acid protonated example (backbone not shown):
```
create asp_example, ([object] and resid [resid #] and resid ASP and not name CA+C+N+O+H*) or ([object] and resid [resid #] and resid ASP and name HG1)
```

__Showing Hydrogen Bonds__

Besides showing the hydrogen when showing a hydrogen bond you will want to place a dotted line between the hydrogen and heavy atoms of a hydrogen bonding pair with the following commands.

```
distance [name], [selection1], [selection2]
hide labels
set dash_color, black
set dash_gap, 0.4
set dash_radius, 0.12
```

__Rendering and Saving your Pymol Image__

After you render the image using 'ray' save your pymol image with the following command. If the rendering is too slow, or just very slow, save your pymol session and render the image on one of the GPU machines.

```
ray
png [name].png, dpi=3600
```

If you entered the basic settings above your pymol image will be saved with a transparent background. When I make a publication quality graphic I always save with a very high dpi, such as 3600, to make resolution images. 

__Tips__

As you figure things out and find new tips add them here.

Highlight residues: A good way to highlight residues is to set the cartoon transparency to 0.95 then when you render the image the ray_trace_gain will only be applied to the residue sidechains. This will result in the selected residues have a subtle outline with the cartoon represented protein having no outline.

Floating residues: Pymol has a way smoothing the cartoon representation which can make it look like the sidechains of residues are not attached to the cartoon, thus it looks like they are floating. You can get rid of this using the following command.

```
set cartoon_flat_sheets, 0
```

### Inkscape

Now that you have made your labeless figures and created pymol images you will put everything together in Inkscape. To start, open inkscape and set up the size of your image in document properties (File -> Document Properties). Change units to inches and set the width of the image to 3.3 in (single column) or 6.5 in (double column).

![screen_shot_2020-11-05_at_1.54.03_pm.png](/screen_shot_2020-11-05_at_1.54.03_pm.png){.align-center}

Now import (file -> import) all of your labelless figures and pymol images and place them on your figure in inkscape. 

![screen_shot_2020-11-05_at_2.11.49_pm.png](/screen_shot_2020-11-05_at_2.11.49_pm.png)

Now add all the labels. The standard font used is Helvetica and the standard font size is 9 for figure labels (A, B, C, ect...) is 12.

__Cropping Pymol Images__ 

In order to crop pymol images use clip (Objects->Clip->Set). First, create a rectangle/square (see tools on left hand side) and place that over the image. Size the rectangle over the portion of the image that you want to keep. Finally use clip and this will crop the image.

__Careful Alignments__

When adding the lables, placeing figures, and pymol images make use of the position tools on the top tool bar. This will allow you to make sure your labels, figures, and images are aligned.

![screen_shot_2020-11-05_at_2.20.15_pm.png](/screen_shot_2020-11-05_at_2.20.15_pm.png)


__Adjusting Text and Shape Colors, Fill, and Outline__

When coloring shapes and text you can use fill and stroke (Objects->Fill and Stroke). To select special colors you can use the color drop tool or just eneter the same hex codes that you used earlier. Fill will be the Fill Color (as you can imagine), Stroke will modify the outline color and Stroke Style will let you cahnge features about the stroke. 

![screen_shot_2020-11-05_at_2.28.44_pm.png](/screen_shot_2020-11-05_at_2.28.44_pm.png)

__Inkscape hates arrows...__

This may surprise you, but the reality of it is painfully true... If you want an arrow I suggested you make an arrow in powerpoint right click on it, export it as a PNG or PDF, and import it into inkscape. 








