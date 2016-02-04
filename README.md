# matplotlib serializer
Plot Caching Orthogonal to MEP25 for the Impatient (=me)


## About

This is a quick hack project to serialize the most important data of a
[matplotlib figure](http://matplotlib.org/faq/usage_faq.html#usage) object:

- [ ] loop through all **Axes**
  - [ ] save the **Axes** (position, size)
  - [ ] save the **Axis** (label, title, range, ticks)
  - [ ] save the **data** as `tiff` (`plot()`'s `lines.Line2D` and `imshow()`'s `image.AxesImage`)
  - [ ] save some more meta such as **title**

- [ ] maybe implement a `load` routine, too


### How to Use

```python
# imports
import matplotlib.pyplot as plt
import numpy as np
from matplotlib-serializer import serializer

# create empty mpl figure
f = plt.figure()

# create class
mpls = serializer()

# restore
if mpls.load(f, "my_analysis_t0") :
    # manipulate f ...
    # ... or its children (axes, axis, cb's, labels, fonts, etc.)
    # or just plot it *interactively* and play with the zoom:
    f.show()
else:
    # dang, I have to crunch data!

    # data
    x = np.arange(100)
    y1 = np.random.uniform(size=100)
    y2 = np.random.uniform(size=100)
    d = np.random.uniform(size=(100,100))

    # figure and axes preparation
    f = plt.figure()
    ax1 = f.add_axes([0.1,0.1,0.4,0.8])
    ax2 = f.add_axes([0.5,0.1,0.4,0.8])

    # plot
    ax1.plot(x, y1)
    ax1.plot(x, y2)
    ax2.plot(d)

    # save
    #   should create a directory alongside the "picture" on is usually drawing
    #   inside the dir we save the data we want to "cache"/serialize
    mpls.save(f, "my_analysis_t0")
    mpls.save_and_plot(f, "my_analysis_t0", "png")
```


### Why?

No time to wait for [MEP25](http://matplotlib.org/devdocs/devel/MEP/MEP25.html)
(introducing a good MVC concept would be *amazing* and solve a lot!).

Annoyed of abstracting even more parts of my scientific post-processing scripts
into a clean layer that explicitly
[caches and annotates each data set](https://stackoverflow.com/questions/21479886/how-to-conveniently-modify-matplotlib-plots-in-the-aftermath-persistence-serial)
before plotting (I am already doing that *while* plotting).

Annoyed of not being able to zoom into already plotted data. Especially if it
takes a long time to generate the data that is then plotted in no-time (caching).

Afraid of using [pickle](https://docs.python.org/3.4/library/pickle.html) because:
  - it's only useful for python users
  - pickle binary blobs changes over time (python2/3)
  - mpl objects themselves change even more over time rendering the pickle blob useless

**Update:** oh sweet, `mpld3`'s `save_json()` routine
[could be half of the solution!!](https://mpld3.github.io/quickstart.html) :)
We just need to export the `"data"` objects or get them out of the json! :)

tiff images are quite nice to hack on (e.g., via ImageJ). float32 support
should (hopefully) be sufficient for most applications.

Most of my images do not need fancy stuff but just of `plot` or `imshow` data,
labels, limits, ticks/ranges, title and color(bar).

Basically this is just a data and meta-data saving mechanism that immitates a "replay"
of the public (!) matplotlib API calls that created the figure and reads the data from
a cache instead of needing the source data again.
