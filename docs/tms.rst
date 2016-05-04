===============
Rendering tiles
===============
After creating and parsing a :class:`RaterLayer`, the tiles for that layer can
be accessed through the tiles url. The raster urls have to be added to the
application's url patterns. Here we assume that the `/raster/` base url is used
as proposed in the :doc:`installation` section.

The tiles url is structured as follows,

::

    /raster/tiles/layer_id/{z}/{x}/{y}.png

where the ``layer_id`` is the primary key of a raster layer. This structure can be
used direclty in online mapping software such as `OpenLayers`__ or `Leaflet`__. An
example request could look like this: ``/raster/tiles/23/8/536/143.png``,
returning a tile in png format of the layer with ID ``pk=23`` at zoom level
``z=8`` and indexes ``x=536`` and ``y=143``.

__ http://openlayers.org/
__ http://leafletjs.com/

By default, the tiles are rendered using simple grayscale. To apply a custom
colormap, a  :class:`Legend` needs to be assigned to the layer. Raster layers
have an optional foreign key to a Legend object, which can be set through the
admin interface.

Caching
-------
All views of django-raster are cached for 24 hours by default. To change the
timeout of the cache use the ``RASTER_TILE_CACHE_TIMEOUT`` setting. To disable
caching, set this timeout to 0.

Legends
-------
Legends are objects that are used to interpret raster data. This includes
the cartographic information (colors), but also the semantics of the data
(such as names). Legends be created through the admin interface.

A legend is stored as in the :class:`Legend` model, which is essentially a
collection of :class:`LegendEntry` that each have an expression for
classifying the data and a semantic meaning of the expression. The semantics
of the expression are stored in the :class:`LegendSemantics` model. Here is
an example for a legend representing two temperatures::

    >>> from raster.models import Legend, LegendEntry, LegendSemantics
    >>> hot_semantics = LegendSemantics.objects.create(name='Hot')
    >>> cold_semantics = LegendSemantics.objects.create(name='Cold')
    >>> hot_entry = LegendEntry.objects.create(semantics=cold, expression='0', color='#0000FF')
    >>> cold_entry = LegendEntry.objects.create(semantics=hot, expression='1', color='#FF0000')
    >>> legend = Legend.objects.create(title='Temperatures')
    >>> legend.entries.add(entry)
    >>> legend.json
    ... '[{"color": "#FFFFFF", "expression": "1", "name": "Earth"}]'


Legend Entries
^^^^^^^^^^^^^^
:class:`LegendEntry` entries combine semantics and a color value with a range
of pixel values. One entry has a foreign key to a :class:`LegendSemantics`
object, a color in hex format and an expression.

The expression is a classification of pixels. It describes a range of pixel
values in the data. It is either an exact number for discrete rasters, or a
formula for continuous rasters::

    expression = "3"  # Matches all pixels with an exact value of 3

For more complicated expressions, a logical expression can be specified through
a formula. The variable ``x`` represents the pixel value in the formula. Here
are some examples of valid formula expressions::

    # Match pixel values bigger than -3 and smaller or equal than 1
    expression = "(-3.0 < x) & (x <= 1)"
    # Match all pixels with values smaller or equal to one
    expression = "x <= 1"

Formula expressions are currenlty not validated on input. Wrongly specified
formulas might lead to errors when rendering raster tiles. Check your formulas
if unexpected errors happen on the TMS endpoints.
