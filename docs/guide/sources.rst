.. _data-sources:

Data sources
===============

Getting data from a source
----------------------------

We can get data from a given source by using :func:`from_source`:

.. py:function:: from_source(name, *args, **kwargs)

  Return a :ref:`data object <data-object>` from the source specified by ``name`` .

  :param str name: the source (see below)
  :param tuple *args: specifies the data location and additional parameters to access the data
  :param dict **kwargs: provides **additional functionalities** including caching, filtering, sorting and indexing

  **earthkit-data** has the following built-in sources:

  .. list-table:: Data sources
    :widths: 30 70
    :header-rows: 1

    * - Name
      - Description
    * - :ref:`data-sources-file`
      - read data from a file/files
    * - :ref:`data-sources-file-pattern`
      - read data from a list of files  created from a pattern
    * - :ref:`data-sources-url`
      - read data from a URL
    * - :ref:`data-sources-url-pattern`
      - read data from a list of URLs created from a pattern
    * - :ref:`data-sources-sample`
      - read example data
    * - :ref:`data-sources-stream`
      - read data from a stream
    * - :ref:`data-sources-memory`
      - read data from a memory buffer
    * - :ref:`data-sources-multi`
      - read data from multiple sources
    * - :ref:`data-sources-ads`
      - retrieve data from the `Copernicus Atmosphere Data Store <https://ads.atmosphere.copernicus.eu/>`_ (ADS)
    * - :ref:`data-sources-cds`
      - retrieve data from the `Copernicus Climate Data Store <https://cds.climate.copernicus.eu/>`_ (CDS)
    * - :ref:`data-sources-eod`
      - retrieve `ECMWF open data <https://www.ecmwf.int/en/forecasts/datasets/open-data>`_
    * - :ref:`data-sources-fdb`
      - retrieve data from the `Fields DataBase <https://fields-database.readthedocs.io/en/latest/>`_ (FDB)
    * - :ref:`data-sources-mars`
      - retrieve data from the ECMWF `MARS archive <https://confluence.ecmwf.int/display/UDOC/MARS+user+documentation>`_
    * - :ref:`data-sources-opendap`
      - retrieve NetCDF data from `OPEnDAP <https://en.wikipedia.org/wiki/OPeNDAP>`_ services
    * - :ref:`data-sources-polytope`
      - retrieve data from the `Polytope services <https://polytope-client.readthedocs.io/en/latest/>`_
    * - :ref:`data-sources-wekeo`
      - retrieve data from `WEkEO`_ using the WEkEO grammar
    * - :ref:`data-sources-wekeocds`
      - retrieve `CDS <https://cds.climate.copernicus.eu/>`_ data stored on `WEkEO`_ using the `cdsapi`_ grammar

----------------------------------

.. _data-sources-file:

file
----

.. py:function:: from_source("file", path, expand_user=True, expand_vars=False, unix_glob=True, recursive_glob=True, parts=None)
  :noindex:

  The simplest source is ``file``, which can access a local file/list of files.

  :param path: input path(s). Each path can contain the :ref:`parts <parts>` defining the byte ranges to read.
  :type path: str, list, tuple
  :param bool expand_user: replace the leading ~ or ~user in ``path`` by that user's home directory. See ``os.path.expanduser``
  :param bool expand_vars:  expand shell environment variables in ``path``. See ``os.path.expandpath``
  :param bool unix_glob: allow UNIX globbing in ``path``
  :param bool recursive_glob: allow recursive scanning of directories. Only used when ``uxix_glob`` is True
  :param parts: the :ref:`parts <parts>` to read from the file(s) specified by ``path``. Cannot be used when ``path`` already defines the :ref:`parts <parts>`.
  :type parts: pair, list or tuple of pairs, None

  *earthkit-data* will inspect the content of the files to check for any of the
  supported :ref:`data formats <data-format>`.

  When the input is an archive format such as ``.zip``, ``.tar``, ``.tar.gz``, etc,
  *earthkit-data* will attempt to open it and extract any usable files, which are then stored in the :ref:`cache <caching>`.

  The ``path`` can be used in a flexible way:

  .. code:: python

      import earthkit.data

      # UNIX globbing is allowed by default
      ds = earthkit.data.from_source("file", "path/to/t_*.grib")

      # list of files can be specified
      ds = earthkit.data.from_source("file", ["path/to/f1.grib", "path/to/f2.grib"])

      # a path can be a directory, in this case it is recursively scanned for supported files
      ds = earthkit.data.from_source("file", "path/to/dir")


  The following examples using parts:

  .. code:: python

      import earthkit.data

      # reading only certain parts (byte ranges) from a single file
      ds = earthkit.data.from_source("file", "my.grib", parts=[(0, 150), (400, 160)])

      # reading only certain parts (byte ranges) from multiple files
      ds = earthkit.data.from_source(
          "file",
          [
              ("a.grib", (0, 150)),
              ("b.grib", (240, 120)),
              ("c.grib", None),
              ("d.grib", [(240, 120), (720, 120)]),
          ],
      )



  Further examples:

    - :ref:`/examples/files.ipynb`
    - :ref:`/examples/multi_files.ipynb`
    - :ref:`/examples/file_parts.ipynb`
    - :ref:`/examples/tar_files.ipynb`
    - :ref:`/examples/grib_overview.ipynb`
    - :ref:`/examples/bufr_temp.ipynb`
    - :ref:`/examples/netcdf.ipynb`
    - :ref:`/examples/odb.ipynb`

.. _data-sources-file-pattern:

file-pattern
--------------

.. py:function:: from_source("file-pattern", pattern, *args, **kwargs)
  :noindex:

  The ``file-pattern`` source will build paths from the pattern specified,
  using the other arguments to fill the pattern. Each argument can be a list
  to iterate and create the cartesian product of all lists.
  Then each file is read in the same ways as with :ref:`file source <data-sources-file>`.

  .. code-block:: python

      import datetime
      import earthkit.data

      ds = earthkit.data.from_source(
          "file-pattern",
          "path/to/data-{my_date:date(%Y-%m-%d)}-{run_time}-{param}.grib",
          {
              "my_date": datetime.datetime(2020, 5, 2),
              "run_time": [12, 18],
              "param": ["t2", "msl"],
          },
      )


  The code above will read the following files::

    path/to/data-2020-05-02-12-t2.grib
    path/to/data-2020-05-02-12-msl.grib
    path/to/data-2020-05-02-18-t2.grib
    path/to/data-2020-05-02-18-msl.grib

Further examples:

    - :ref:`/examples/files.ipynb`


.. _data-sources-url:

url
---

.. py:function:: from_source("url", url, unpack=True, parts=None, stream=False, read_all=False)
  :noindex:

  The ``url`` source will download the data from the address specified and store it in the :ref:`cache <caching>`. The supported data formats are the same as for the :ref:`file <data-sources-file>` data source above.

  :param url: the URL(s) to download. Each URL can contain the :ref:`parts <parts>` defining the byte ranges to read.
  :type url: str
  :param bool unpack: for archive formats such as ``.zip``, ``.tar``, ``.tar.gz``, etc, *earthkit-data* will attempt to open it and extract any usable file. To keep the downloaded file as is use ``unpack=False``
  :param parts: the :ref:`parts <parts>` to read from the resource(s) specified by ``url``. Cannot be used when ``url`` already defines the :ref:`parts <parts>`.
  :type parts: pair, list or tuple of pairs, None
  :param bool stream: when it is ``True`` the data is read as a :ref:`stream <streams>`. Otherwise the data is retrieved into a file and stored in the :ref:`cache <caching>`. This option only works for GRIB data. No archive formats supported (``unpack`` is ignored). ``stream`` only works for ``http`` and ``https`` URLs. See details about streams :ref:`here <streams>`.
  :param bool read_all: when it is ``True`` all the data is read straight to memory from a :ref:`stream <streams>`. Used when ``stream=True``. *New in version 0.8.0*
  :type group_by: str, list of str
  :param dict **kwargs: other keyword arguments specifying the request

  .. code-block:: python

      >>> import earthkit.data
      >>> ds = earthkit.data.from_source(
      ...     "url",
      ...     "https://get.ecmwf.int/repository/test-data/earthkit-data/examples/test4.grib",
      ... )
      >>> ds.ls()
        centre shortName    typeOfLevel  level  dataDate  dataTime stepRange dataType  number    gridType
      0   ecmf         t  isobaricInhPa    500  20070101      1200         0       an       0  regular_ll
      1   ecmf         z  isobaricInhPa    500  20070101      1200         0       an       0  regular_ll
      2   ecmf         t  isobaricInhPa    850  20070101      1200         0       an       0  regular_ll
      3   ecmf         z  isobaricInhPa    850  20070101      1200         0       an       0  regular_ll

  .. code-block:: python

      >>> import earthkit.data
      >>> ds = earthkit.data.from_source(
      ...     "url",
      ...     "https://get.ecmwf.int/repository/test-data/earthkit-data/examples/test4.grib",
      ...     parts=[(0, 130428), (260856, 130428)],
      ... )
      >>> ds.ls()
        centre shortName    typeOfLevel  level  dataDate  dataTime stepRange dataType  number    gridType
      0   ecmf         t  isobaricInhPa    500  20070101      1200         0       an       0  regular_ll
      1   ecmf         t  isobaricInhPa    850  20070101      1200         0       an       0  regular_ll

  Further examples:

    - :ref:`/examples/url.ipynb`
    - :ref:`/examples/url_parts.ipynb`
    - :ref:`/examples/url_stream.ipynb`


.. _data-sources-url-pattern:


url-pattern
-----------

.. py:function:: from_source("url-pattern", url, unpack=True)
  :noindex:

  The ``url-pattern`` source will build urls from the pattern specified,
  using the other arguments to fill the pattern. Each argument can be a list
  to iterate and create the cartesian product of all lists.
  Then each url is downloaded and stored in the :ref:`cache <caching>`. The
  supported download the data from the address data formats are the same as
  for the *file* and *url* data sources above.

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "url-pattern",
          "https://www.example.com/data-{foo}-{bar}-{qux}.csv",
          foo=[1, 2, 3],
          bar=["a", "b"],
          qux="unique",
      )

  The code above will download and process the data from the six following urls::

    https://www.example.com/data-1-a-unique.csv
    https://www.example.com/data-2-a-unique.csv
    https://www.example.com/data-3-a-unique.csv
    https://www.example.com/data-1-b-unique.csv
    https://www.example.com/data-2-b-unique.csv
    https://www.example.com/data-3-b-unique.csv

  If the urls are pointing to archive format, the data will be unpacked by
  ``url-pattern`` according to the **unpack** argument, similarly to what
  the source ``url`` does (see above the :ref:`data-sources-url` source).



.. _data-sources-sample:

sample
------

.. py:function:: from_source("sample", name_or_path)
  :noindex:

  The ``sample`` source will download example data prepared for earthkit and store it in the :ref:`cache <caching>`. The supported data formats are the same as for the :ref:`file <data-sources-file>` data source above.

  :param name_or_path: input file name(s) or relative path(s) to the root of the remote storage folder.
  :type name_or_path: str, list, tuple

  .. code-block:: python

    >>> import earthkit.data
    >>> ds = earthkit.data.from_source("sample", "storm_ophelia_wind_850.grib")
    >>> ds.ls()
      centre shortName    typeOfLevel  level  dataDate  dataTime stepRange dataType  number    gridType
    0   ecmf         u  isobaricInhPa    850  20171016         0         0       an       0  regular_ll
    1   ecmf         v  isobaricInhPa    850  20171016         0         0       an       0  regular_ll



.. _data-sources-stream:

stream
--------------

.. py:function:: from_source("stream", stream, read_all=False)
  :noindex:

  The ``stream`` source will read data from a stream (or streams), which can be an FDB stream, a standard Python IO stream or any object implementing the necessary stream methods. At the moment it only works for :ref:`grib` and CoverageJson data. For more details see :ref:`here <streams>`.

  :param stream: the stream(s)
  :type stream: stream, list, tuple
  :param bool read_all: when it is ``True`` all the data is read into memory from a stream. Used when ``stream=True``. *New in version 0.8.0*

  In the examples below, for simplicity, we create a file stream from a :ref:`grib` file. By default :ref:`from_source() <data-sources-stream>` returns an object that can only be used as an iterator.

  .. code-block:: python

      >>> import earthkit.data
      >>> stream = open("docs/examples/test4.grib", "rb")
      >>> ds = earthkit.data.from_source("stream", stream)

      # f is a GribField
      >>> for f in ds:
      ...     print(f)
      ...
      GribField(t,500,20070101,1200,0,0)
      GribField(z,500,20070101,1200,0,0)
      GribField(t,850,20070101,1200,0,0)
      GribField(z,850,20070101,1200,0,0)

  We can also iterate through the stream in batches of fixed size using ``batched()``:

    .. code-block:: python

      >>> import earthkit.data
      >>> stream = open("docs/examples/test4.grib", "rb")
      >>> ds = earthkit.data.from_source("stream", stream, batch_size=2)

       # f is a FieldList
      >>> for f in ds.batched(2):
      ...     print(f"len={len(f)} {f.metadata(('param', 'level'))}")
      ...
      len=2 [('t', 500), ('z', 500)]
      len=2 [('t', 850), ('z', 850)]


  When using ``group_by()`` we can iterate through the stream in groups defined by metadata keys. In this case each iteration step yields a :obj:`FieldList <data.readers.grib.index.FieldList>`.

    .. code-block:: python

      >>> import earthkit.data
      >>> stream = open("docs/examples/test4.grib", "rb")
      >>> ds = earthkit.data.from_source("stream", stream)

      # f is a FieldList
      >>> for f in ds.group_by("level"):
      ...     print(f"len={len(f)} {f.metadata(('param', 'level'))}")
      ...
      len=2 [('t', 500), ('z', 500)]
      len=2 [('t', 850), ('z', 850)]

  We can consume the whole stream and load all the data into memory by using ``read_all=True`` in :ref:`from_source() <data-sources-stream>`. **Use this option carefully!**

    .. code-block:: python

      >>> import earthkit.data
      >>> stream = open("docs/examples/test4.grib", "rb")
      >>> ds = earthkit.data.from_source("stream", stream, read_all=True)

      # ds is empty at this point, but calling any method on it will
      # consume the whole stream
      >>> len(ds)
      4

      # now ds stores all the messages in memory

  See the following notebook examples for further details:

    - :ref:`/examples/data_from_stream.ipynb`
    - :ref:`/examples/fdb.ipynb`
    - :ref:`/examples/url_stream.ipynb`


.. _data-sources-memory:

memory
--------------

.. py:function:: from_source("memory", buffer)
  :noindex:

  The ``memory`` source will read data from a memory buffer. Currently it only works for a ``buffer`` storing a single :ref:`grib` message or CoverageJson data.

  Please note that a buffer can always be read as a :ref:`stream source <data-sources-stream>` using ``io.BytesIO``.

  .. code-block:: python

      import io
      import earthkit.data

      # buffer stores GRIB messages
      buffer = ...
      stream = io.BytesIO(buffer)

      ds = earthkit.data.from_source("stream", stream)
      for f in ds:
          print(f.metadata("param"))


.. _data-sources-multi:

multi
--------------

.. py:function:: from_source("multi", *sources, merger=None, **kwargs)
  :noindex:

  The ``multi`` source reads multiple sources.

  :param tuple *sources: the sources
  :param merger: if it is None an attempt is made to merge/concatenate the sources by their classes (using the nearest common class). Otherwise the sources are merged/concatenated using the merger in a lazy way. The merger can one of the following:

    - class/object implementing  the :func:`to_xarray` or :func:`to_pandas` methods
    - callable
    - str, describing a call either to "concat" or "merge". E.g.: "concat(concat_dim=time)"
    - tuple with 2 elements. The fist element is a str, either "concat" or "merge", and the second element is a dict with the keyword arguments for the call. E.g.: ("concat", {"concat_dim": "time"})
  :param dict **kwargs: other keyword arguments



.. _data-sources-ads:

ads
---

.. py:function:: from_source("ads", dataset, *args, **kwargs)
  :noindex:

  The ``ads`` source accesses the `Copernicus Atmosphere Data Store`_ (ADS), using the cdsapi_ package. In addition to data retrieval, ``request`` also has post-processing options such as ``grid`` and ``area`` for re-gridding and sub-area extraction respectively.

  :param str dataset: the name of the ADS dataset
  :param tuple *args: specify the request as a dict
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves CAMS global reanalysis GRIB data for 2 parameters:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "ads",
          "cams-global-reanalysis-eac4",
          variable=["particulate_matter_10um", "particulate_matter_1um"],
          area=[50, -50, 20, 50],  # N,W,S,E
          date="2012-12-12",
          time="12:00",
      )

  Data downloaded from the ADS is stored in the the :ref:`cache <caching>`.

  To access data from the ADS, you will need to register and retrieve an access token. The process is described `here <https://ads.atmosphere.copernicus.eu/api-how-to>`__. For more information, see the `ADS_knowledge base`_.

  Further examples:

      - :ref:`/examples/ads.ipynb`


.. _data-sources-cds:

cds
---

.. py:function:: from_source("cds", dataset, *args, prompt=True, **kwargs)
  :noindex:

  The ``cds`` source accesses the `Copernicus Climate Data Store`_ (CDS), using the cdsapi_ package. In addition to data retrieval, the request has post-processing options such as ``grid`` and ``area`` for regridding and sub-area extraction respectively. It can
  also contain the earthkit-data specific :ref:`split_on <split_on>` parameter.

  :param str dataset: the name of the CDS dataset
  :param tuple *args: specify the request as dict. A sequence of dicts can be used to specify multiple requests.
  :param bool prompt: when True it can offer a prompt to specify the credentials for cdsapi_ and write them into the default RC file ``~/.cdsapirc``. The prompt only appears when:

    - no cdsapi_ RC file exists at the default location ``~/.cdsapirc``
    - no cdsapi_ RC file exists at the location specified via the ``CDSAPI_RC`` environment variable
    - no credentials specified via the ``CDSAPI_URL`` and ``CDSAPI_KEY`` environment variables
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves ERA5 reanalysis GRIB data for a subarea for 2 surface parameters. The request is specified using ``kwargs``:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "cds",
          "reanalysis-era5-single-levels",
          variable=["2t", "msl"],
          product_type="reanalysis",
          area=[50, -10, 40, 10],  # N,W,S,E
          grid=[2, 2],
          date="2012-05-10",
      )

  The same retrieval can be defined by passing the request as a positional argument:

  .. code-block:: python

      import earthkit.data

      req = dict(
          variable=["2t", "msl"],
          product_type="reanalysis",
          area=[50, -10, 40, 10],  # N,W,S,E
          grid=[2, 2],
          date="2012-05-10",
      )

      ds = earthkit.data.from_source(
          "cds",
          "reanalysis-era5-single-levels",
          req,
      )


  Data downloaded from the CDS is stored in the the :ref:`cache <caching>`.

  To access data from the CDS, you will need to register and retrieve an access token. The process is described `here <https://cds.climate.copernicus.eu/api-how-to>`__. For more information, see the `CDS_knowledge base`_.

  Further examples:

      - :ref:`/examples/cds.ipynb`


.. _data-sources-eod:

ecmwf-open-data
-------------------

.. py:function:: from_source("ecmwf-open-data", *args, source="ecmwf", model="ifs", **kwargs)
  :noindex:

  The ``ecmwf-open-data`` source provides access to the `ECMWF open data`_, which is a subset of ECMWF real-time forecast data made available to the public free of charge.  It uses the `ecmwf-opendata <https://github.com/ecmwf/ecmwf-opendata>`_ package.

  :param tuple *args: specify the request as a dict
  :param str source: either the name of the server to contact or a fully qualified URL. Possible values are "ecmwf" to access ECMWF's servers, or "azure" to access data hosted on Microsoft's Azure. Default is "ecmwf".
  :param str model: name of the model that produced the data. Use "ifs" for the physics-driven model and "aifs" for the data-driven model. Please note that "aifs" is currently experimental and only produces a small subset of fields. Default is "ifs".
  :param dict **kwargs: other keyword arguments specifying the request

  Details about the request format can be found `here <https://github.com/ecmwf/ecmwf-opendata>`__.

  The following example retrieves forecast for 2 surface parameters from the latest forecast:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "ecmwf-open-data", param=["2t", "msl"], levtype="sfc", step=[0, 6, 12]
      )


  The resulting GRIB data files are stored in the :ref:`cache <caching>`.

  Further examples:

      - :ref:`/examples/ecmwf_open_data.ipynb`


.. _data-sources-fdb:

fdb
---

.. py:function:: from_source("fdb", *args, stream=True, read_all=False, **kwargs)
  :noindex:

  The ``fdb`` source accesses the `FDB (Fields DataBase) <https://fields-database.readthedocs.io/en/latest/>`_, which is a domain-specific object store developed at ECMWF for storing, indexing and retrieving GRIB data. earthkit-data uses the `pyfdb <https://pyfdb.readthedocs.io/en/latest>`_ package to retrieve data from FDB.

  :param tuple *args: positional arguments specifying the request as a dict
  :param bool stream: when it is ``True`` the data is read as a :ref:`stream <streams>`. Otherwise it is retrieved into a file and stored in the :ref:`cache <caching>`. Stream-based access only works for :ref:`grib` and CoverageJson data. See details about streams :ref:`here <streams>`.
  :param bool read_all: when it is ``True`` all the data is read into memory from a :ref:`stream <streams>`. Used when ``stream=True``. *New in version 0.8.0*
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves analysis :ref:`grib` data for 3 surface parameters as stream.
  By default we will consume one message at a time and ``ds`` can only be used as an iterator:

  .. code-block:: python

      >>> import earthkit.data
      >>> request = {
      ...     "class": "od",
      ...     "expver": "0001",
      ...     "stream": "oper",
      ...     "date": "20240421",
      ...     "time": [0, 12],
      ...     "domain": "g",
      ...     "type": "an",
      ...     "levtype": "sfc",
      ...     "step": 0,
      ...     "param": [151, 167, 168],
      ... }
      >>>
      >>> ds = earthkit.data.from_source("fdb", request)
      >>> for f in ds:
      ...     print(f)
      ...
      GribField(msl,None,20240421,0,0,0)
      GribField(2t,None,20240421,0,0,0)
      GribField(2d,None,20240421,0,0,0)
      GribField(msl,None,20240421,1200,0,0)
      GribField(2t,None,20240421,1200,0,0)
      GribField(2d,None,20240421,1200,0,0)

  We can also iterate through the stream in batches of fixed size using ``batched``:

  .. code-block:: python

      >>> ds = earthkit.data.from_source("fdb", request)
      >>> for f in ds.batched(2):
      ...     print(f"len={len(f)} {f.metadata(('param', 'level'))}")
      ...
      len=2 [('msl', 0), ('2t', 0)]
      len=2 [('2d', 0), ('msl', 0)]
      len=2 [('2t', 0), ('2d', 0)]


  When using ``group_by()`` we can iterate through the stream in groups defined by metadata keys. In this case each iteration step yields a :obj:`FieldList <data.readers.grib.index.FieldList>`.

  .. code-block:: python

      >>> ds = earthkit.data.from_source("fdb", request)
      >>> for f in ds.group_by("time"):
      ...     print(f"len={len(f)} {f.metadata(('param', 'level'))}")
      ...
      len=3 [('msl', 0), ('2t', 0), ('2d', 0)]
      len=3 [('msl', 0), ('2t', 0), ('2d', 0)]

  We can consume the whole stream and load all the data into memory by using ``read_all=True`` in :ref:`from_source() <data-sources-stream>`. **Use this option carefully!**

  .. code-block:: python

      >>> import earthkit.data
      >>> ds = earthkit.data.from_source("fdb", request, read_all=True)

      # ds is empty at this point, but calling any method on it will
      # consume the whole stream
      >>> len(ds)
      3

      # now ds stores all the messages in memory

  Further examples:

      - :ref:`/examples/fdb.ipynb`
      - :ref:`/examples/grib_fdb_write.ipynb`


.. _data-sources-mars:

mars
--------------

.. py:function:: from_source("mars", *args, prompt=True, log="default", **kwargs)
  :noindex:

  The ``mars`` source will retrieve data from the ECMWF MARS (Meteorological Archival and Retrieval System) archive. In addition
  to data retrieval, the request specified as ``*args`` and/or ``**kwargs`` also has GRIB post-processing options such as ``grid`` and ``area`` for regridding and
  sub-area extraction, respectively.

  To figure out which data you need, or discover relevant data available in MARS, see the publicly accessible `MARS catalog`_ (or this `access restricted catalog <https://apps.ecmwf.int/mars-catalogue/>`_).

  The MARS access is direct when the MARS client is installed (as at ECMWF) and the ``use-standalone-mars-client-when-available`` :ref:`settings <settings>` is True (this is the default), otherwise it will use the `web API`_. In order to use the `web API`_ you will need to register and retrieve an access token. For a more extensive documentation about MARS, please refer to the `MARS user documentation`_.

  :param tuple *args: positional arguments specifying the request as a dict
  :param bool prompt: when True it can offer a prompt to specify the credentials for `web API`_ and write them into the default RC file ``~/.ecmwfapirc``. The prompt only appears when:

    - no `web API`_ RC file exists at the default location ``~/.ecmwfapirc``
    - no `web API`_ RC file exists at the location specified via the ``ECMWF_API_RC_FILE`` environment variable
    - no credentials specified via the ``ECMWF_API_URL`` and ``ECMWF_API_KEY``  environment variables
  :param log: control the logging of the retrieval. The behaviour depends on the underlying MARS client used:

    - `web API`_ based access:

      - "default": the built-in logging of `web API`_ is used (the log is written to stdout)
      - None: turn off logging
      - callable: the log is written to the specified callable. The callable should accept a single argument, a string with the log message.

      .. code-block:: python

          import earthkit.data


          def my_logging_function(msg):
              print("message=", msg)


          request = {...}
          ds = earthkit.data.from_source("mars", request, log=my_logging_function)

    - direct MARS access:

      - "default": log is written to stdout
      - None: turn off logging
      - dict specifying the "stdout" or/and the "stderr" kwargs for Pythons's ``subrocess.run()`` method

  :type log: str, None, callable, dict
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves analysis GRIB data for a subarea for 2 surface parameters:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "mars",
          {
              "param": ["2t", "msl"],
              "levtype": "sfc",
              "area": [50, -50, 20, 50],
              "grid": [2, 2],
              "date": "2023-05-10",
          },
      )

  Data downloaded from MARS is stored in the :ref:`cache <caching>`.

  Further examples:

      - :ref:`/examples/mars.ipynb`


.. _data-sources-opendap:

opendap
--------

.. py:function:: from_source("opendap", url)
  :noindex:

  The ``opendap`` source accesses NetCDF data from `OPeNDAP <https://en.wikipedia.org/wiki/OPeNDAP>`_ services. OPenDAP is an acronym for "Open-source Project for a Network Data Access Protocol".

  :param str url: the url of the remote NetCDF file

  Examples:

      - :ref:`/examples/netcdf_opendap.ipynb`


.. _data-sources-polytope:

polytope
--------

.. py:function:: from_source("polytope", collection, *args, address=None, user_email=None, user_key=None, stream=True, read_all=False, **kwargs)
  :noindex:

  The ``polytope`` source accesses the `Polytope web services <https://polytope-client.readthedocs.io/en/latest/>`_ , using the polytope-client_ package.

  :param str collection: the name of the polytope collection
  :param tuple *args: specify the request as a dict
  :param str address: specify the address of the polytope service
  :param str user_email: specify the user email credential. Must be used together with ``user_key``. This is an alternative to using the ``POLYTOPE_USER_EMAIL`` environment variable. *New in version 0.7.0*
  :param str user_key: specify the user key credential. Must be used together with ``user_email``. This is an alternative to using the ``POLYTOPE_USER_KEY`` environment variable. *New in version 0.7.0*
  :param bool stream: when ``True`` the data is read as a :ref:`stream <streams>`. Otherwise it is retrieved into a file and stored in the :ref:`cache <caching>`. Stream-based access only works for :ref:`grib` and CoverageJson data. See details about streams :ref:`here <streams>`.
  :param bool read_all: when ``True`` all the data is read into memory from a :ref:`stream <streams>`. Used when ``stream=True``. *New in version 0.8.0*
  :param dict **kwargs: other keyword arguments, these can include options passed to the polytope-client_


  The following example retrieves GRIB data from the "ecmwf-mars" polytope collection:

  .. code-block:: python

      import earthkit.data

      request = {
          "stream": "oper",
          "levtype": "pl",
          "levellist": "1",
          "param": "130.128",
          "step": "0/12",
          "time": "00:00:00",
          "date": "20200915",
          "type": "fc",
          "class": "rd",
          "expver": "hsvs",
          "domain": "g",
      }

      ds = earthkit.data.from_source("polytope", "ecmwf-mars", request, stream=False)

  Data downloaded from the polytope service is stored in the the :ref:`cache <caching>`. However,
  please note that, in the current version, each call to  :func:`from_source` will download the data again.

  To access data from polytope, you will need to register and retrieve an access token.

  Further examples:

      - :ref:`/examples/polytope.ipynb`


.. _data-sources-wekeo:

wekeo
-----

.. py:function:: from_source("wekeo", dataset, *args, prompt=True, **kwargs)
  :noindex:

  `WEkEO`_ is the Copernicus DIAS reference service for environmental data and virtual processing environments. The ``wekeo`` source provides access to `WEkEO`_ using the WEkEO grammar. The retrieval is based on the hda_ Python API.

  :param str dataset: the name of the WEkEO dataset
  :param tuple *args: specify the request as a dict
  :param bool prompt: when True it can offer a prompt to specify the credentials for hda_ and write them into the default RC file ``~/.hdarc``. The prompt only appears when:

    - no hda_ RC file exists at the default location ``~/.hdarc``
    - no hda_ RC file exists at the location specified via the ``HDA_RC`` environment variable
    - no credentials specified via the ``HDA_URL``, ``HDA_USER`` and ``HDA_PASSWORD`` environment variables
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves Normalized Difference Vegetation Index data derived from EO satellite imagery in NetCDF format:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "wekeo",
          "EO:CLMS:DAT:CGLS_GLOBAL_NDVI300_V1_333M",
          request={
              "datasetId": "EO:CLMS:DAT:CGLS_GLOBAL_NDVI300_V1_333M",
              "dateRangeSelectValues": [
                  {
                      "name": "dtrange",
                      "start": "2014-01-01T00:00:00.000Z",
                      "end": "2014-01-01T23:59:59.999Z",
                  }
              ],
          },
      )


  Data downloaded from WEkEO is stored in the the :ref:`cache <caching>`.

  To access data from WEkEO, you will need to register and set up the Harmonized Data Access (HDA) API client. The process is described `here <https://help.wekeo.eu/en/articles/6751608-what-is-the-hda-api-python-client-and-how-to-use-it>`_.

  Further examples:

      - :ref:`/examples/wekeo.ipynb`


.. _data-sources-wekeocds:

wekeocds
--------

.. py:function:: from_source("wekeocds", dataset, *args, prompt=True, **kwargs)
  :noindex:

  `WEkEO`_ is the Copernicus DIAS reference service for environmental data and virtual processing environments. The ``wekeocds`` source provides access to `Copernicus Climate Data Store`_ (CDS) datasets served on `WEkEO`_ using the `cdsapi`_ grammar. The retrieval is based on the hda_ Python API.

  :param str dataset: the name of the WEkEO dataset
  :param tuple *args: specify the request as a dict
  :param bool prompt: when True it can offer a prompt to specify the credentials for hda_ and write them into the default RC file ``~/.hdarc``. The prompt only appears when:

    - no hda_ RC file exists at the default location ``~/.hdarc``
    - no hda_ RC file exists at the location specified via the ``HDA_RC`` environment variable
    - no credentials specified via the ``HDA_URL``, ``HDA_USER`` and ``HDA_PASSWORD`` environment variables
  :param dict **kwargs: other keyword arguments specifying the request

  The following example retrieves ERA5 surface data for multiple days in GRIB format:

  .. code-block:: python

      import earthkit.data

      ds = earthkit.data.from_source(
          "wekeocds",
          "EO:ECMWF:DAT:REANALYSIS_ERA5_SINGLE_LEVELS",
          variable=["2m_temperature", "mean_sea_level_pressure"],
          product_type=["reanalysis"],
          year=["2012"],
          month=["12"],
          day=["12", "13", "14", "15"],
          time=["11:00"],
          format="grib",
      )

  Data downloaded from WEkEO is stored in the the :ref:`cache <caching>`.

  To access data from WEkEO, you will need to register and set up the Harmonized Data Access (HDA) API client. The process is described `here <https://help.wekeo.eu/en/articles/6751608-what-is-the-hda-api-python-client-and-how-to-use-it>`_.

  Further examples:

      - :ref:`/examples/wekeo.ipynb`


.. _MARS catalog: https://apps.ecmwf.int/archive-catalogue/
.. _MARS user documentation: https://confluence.ecmwf.int/display/UDOC/MARS+user+documentation
.. _web API: https://www.ecmwf.int/en/forecasts/access-forecasts/ecmwf-web-api

.. _Copernicus Climate Data Store: https://cds.climate.copernicus.eu/
.. _Copernicus Atmosphere Data Store: https://ads.atmosphere.copernicus.eu/
.. _cdsapi: https://pypi.org/project/cdsapi/
.. _CDS_knowledge base: https://confluence.ecmwf.int/pages/viewpage.action?pageId=151530614
.. _ADS_knowledge base: https://confluence.ecmwf.int/pages/viewpage.action?pageId=151530675

.. _ECMWF open data: https://www.ecmwf.int/en/forecasts/datasets/open-data

.. _WEkEO: https://www.wekeo.eu/
.. _hda: https://pypi.org/project/hda

.. _polytope-client: https://pypi.org/project/polytope-client
