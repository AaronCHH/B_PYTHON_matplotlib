
# Chapter 5: Working with a File Output

<!-- toc orderedList:0 depthFrom:1 depthTo:6 -->

- [Chapter 5: Working with a File Output](#chapter-5-working-with-a-file-output)
	- [Introduction](#introduction)
	- [Generating a PNG picture file](#generating-a-png-picture-file)
	- [Handling transparency](#handling-transparency)
	- [Controlling the output resolution](#controlling-the-output-resolution)
	- [Generating PDF or SVG documents](#generating-pdf-or-svg-documents)
	- [Handling multiple-page PDF documents](#handling-multiple-page-pdf-documents)

<!-- tocstop -->

## Introduction

## Generating a PNG picture file


```python
# %load Chapter5/01.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

plot.plot(X, Y, c = 'k')
plot.savefig('sinc1.png')

```

## Handling transparency


```python
# %load Chapter5/02.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

plot.plot(X, Y, c = 'k')
plot.savefig('sinc2.png', transparent = True)

```


```python
# %load Chapter5/03.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

plot.plot(X, Y)
plot.savefig('sinc3.png', bbox_inches = 'tight')

```


```python
# %load Chapter5/04.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

plot.plot(X, Y) # 72, 96, 150, 300
plot.savefig('sinc4.png', dpi = 100)

```


```python
# %load Chapter5/05.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

fig = plot.figure(figsize = (6.40, 4.80))
plot.plot(X, Y)
plot.savefig('sinc5.png', dpi = 72)

```


```python
# %load Chapter5/06.py
import numpy
from matplotlib import pyplot as plot

X = numpy.linspace(-10, 10, 1024)
Y = numpy.sinc(X)

fig = plot.figure(figsize = (33.11, 46.81))
plot.plot(X, Y)
plot.savefig('sinc6.pdf')

```


```python
# %load Chapter5/07.py
import random
import matplotlib.pyplot as plot

name_list = ('Omar', 'Serguey', 'Max', 'Zhou', 'Abidin')
value_list = [random.randint(0, 99) for name in name_list]
pos_list = range(len(name_list))

plot.bar(pos_list, value_list, alpha = .5, color = '.25', align = 'center')

plot.xticks(pos_list, name_list)

plot.savefig('bar.png', transparent = True)

```

## Controlling the output resolution


```python
# %load Chapter5/08a.py
import numpy
import matplotlib.pyplot as plot

theta = numpy.linspace(0, 2 * numpy.pi, 8)
points = list(zip(numpy.cos(theta), numpy.sin(theta)))

plot.figure(figsize=(4., 4.))
plot.gca().add_patch(plot.Polygon(points, color = '.75'))

plot.grid(True)
plot.axis('scaled')

plot.savefig('polygon-a.png', dpi = 128)

```


```python
# %load Chapter5/08b.py
import numpy
import matplotlib.pyplot as plot

theta = numpy.linspace(0, 2 * numpy.pi, 8)
points = list(zip(numpy.cos(theta), numpy.sin(theta)))

plot.figure(figsize=(8., 8.))
plot.gca().add_patch(plot.Polygon(points, color = '.75'))

plot.grid(True)
plot.axis('scaled')

plot.savefig('polygon-b.png', dpi = 64)

```


```python
# %load Chapter5/08a.py
import numpy
import matplotlib.pyplot as plot

theta = numpy.linspace(0, 2 * numpy.pi, 8)
points = list(zip(numpy.cos(theta), numpy.sin(theta)))

plot.figure(figsize=(4., 4.))
plot.gca().add_patch(plot.Polygon(points, color = '.75'))

plot.grid(True)
plot.axis('scaled')

plot.savefig('polygon-c.png', dpi = 300)

```

## Generating PDF or SVG documents


```python
import numpy as np
from matplotlib import pyplot as plt

X = np.linspace(-10, 10, 1024)
Y = np.sinc(X)

plt.plot(X, Y)
plt.savefig('sinc.pdf')
```

## Handling multiple-page PDF documents


```python
# %load Chapter5/09.py
import numpy

from matplotlib import pyplot as plot
from matplotlib.backends.backend_pdf import PdfPages

# Generate the data
data = numpy.random.randn(15, 1024)

# The PDF document
pdf_pages = PdfPages('histograms.pdf')

# Generate the pages
nb_plots = data.shape[0]
nb_plots_per_page = 5
nb_pages = int(numpy.ceil(nb_plots / float(nb_plots_per_page)))
grid_size = (nb_plots_per_page, 1)

for i, samples in enumerate(data):
  # Create a figure instance (ie. a new page) if needed
  if i % nb_plots_per_page == 0:
  	fig = plot.figure(figsize=(8.27, 11.69), dpi=100)

  # Plot stuffs !
  plot.subplot2grid(grid_size, (i % nb_plots_per_page, 0))
  plot.hist(samples, 32, normed=1, facecolor='#808080', alpha=0.75)

  # Close the page if needed
  if (i + 1) % nb_plots_per_page == 0 or (i + 1) == nb_plots:
    plot.tight_layout()
    pdf_pages.savefig(fig)

# Write the PDF document to the disk
pdf_pages.close()

```


```python

```
