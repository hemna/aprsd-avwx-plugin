AVWX Api based APRSD plugins
============================

|PyPI| |Status| |Python Version| |License|

|Read the Docs| |Tests| |Codecov|

|pre-commit|

.. |PyPI| image:: https://img.shields.io/pypi/v/aprsd-avwx-plugin.svg
   :target: https://pypi.org/project/aprsd-avwx-plugin/
   :alt: PyPI
.. |Status| image:: https://img.shields.io/pypi/status/aprsd-avwx-plugin.svg
   :target: https://pypi.org/project/aprsd-avwx-plugin/
   :alt: Status
.. |Python Version| image:: https://img.shields.io/pypi/pyversions/aprsd-avwx-plugin
   :target: https://pypi.org/project/aprsd-avwx-plugin
   :alt: Python Version
.. |License| image:: https://img.shields.io/pypi/l/aprsd-avwx-plugin
   :target: https://opensource.org/licenses/MIT
   :alt: License
.. |Read the Docs| image:: https://img.shields.io/readthedocs/aprsd-avwx-plugin/latest.svg?label=Read%20the%20Docs
   :target: https://aprsd-avwx-plugin.readthedocs.io/
   :alt: Read the documentation at https://aprsd-avwx-plugin.readthedocs.io/
.. |Tests| image:: https://github.com/hemna/aprsd-avwx-plugin/workflows/Tests/badge.svg
   :target: https://github.com/hemna/aprsd-avwx-plugin/actions?workflow=Tests
   :alt: Tests
.. |Codecov| image:: https://codecov.io/gh/hemna/aprsd-avwx-plugin/branch/main/graph/badge.svg
   :target: https://codecov.io/gh/hemna/aprsd-avwx-plugin
   :alt: Codecov
.. |pre-commit| image:: https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white
   :target: https://github.com/pre-commit/pre-commit
   :alt: pre-commit


Features
--------

* Fetch METAR weather reports for APRS callsigns
* Automatically finds the nearest weather station to a callsign's location
* Uses aprs.fi to get GPS coordinates for callsigns
* Returns raw METAR data for aviation weather information


Requirements
------------

* Python 3.11 or higher
* APRSD server (version 4.2.0 or higher)
* AVWX API access (either via subscription or self-hosted)
* aprs.fi API key (for location lookups)


Installation
------------

You can install *AVWX Api based APRSD plugins* via pip_ from PyPI_:

.. code:: console

   $ pip install aprsd-avwx-plugin


AVWX API Setup
--------------

This plugin requires access to an AVWX API instance. You have two options:

Official AVWX REST API (Subscription Required)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The official AVWX REST API is available at https://info.avwx.rest/_. This service provides:

* METAR - Current surface conditions
* TAF - 24-hour forecasts
* PIREP - Inflight observations
* AIRMET/SIGMET - Weather advisories
* NOTAM - Notices to airmen
* Station information and search

The official service requires a subscription (hobby, pro, or enterprise tiers). Basic METAR and TAF parsing services are free, but other features require paid plans.

For more information, visit the `AVWX REST API`_ website.

.. _AVWX REST API: https://info.avwx.rest/

Self-Hosted AVWX Engine
~~~~~~~~~~~~~~~~~~~~~~~~

Alternatively, you can self-host your own AVWX API using the open-source `avwx-engine`_ project. This allows you to run your own aviation weather parsing service without subscription fees.

To self-host:

1. Clone the avwx-engine repository:

   .. code:: console

      $ git clone https://github.com/avwx-rest/avwx-engine
      $ cd avwx-engine

2. Follow the installation and setup instructions in the `avwx-engine`_ README

3. Configure the plugin to point to your self-hosted instance (see Configuration below)

The avwx-engine is a Python library that can be run as a service. You'll need to set up an API server wrapper around it to provide REST API endpoints that this plugin can consume.


Configuration
-------------

The plugin requires two configuration options in your APRSD configuration file:

* ``avwx_plugin.apiKey`` - Your AVWX API key (required)
* ``avwx_plugin.base_url`` - The base URL for the AVWX API (default: ``https://avwx.rest``)

Example configuration:

.. code:: ini

   [avwx_plugin]
   apiKey = your-api-key-here
   base_url = https://avwx.rest

If you're self-hosting, set ``base_url`` to your self-hosted instance URL:

.. code:: ini

   [avwx_plugin]
   apiKey = your-api-key-here
   base_url = http://localhost:8000

You also need an aprs.fi API key for location lookups:

.. code:: ini

   [aprs_fi]
   apiKey = your-aprs-fi-api-key-here


Usage
-----

Once installed and configured, the plugin can be triggered via APRS messages.

Command Format
~~~~~~~~~~~~~~

The plugin responds to messages starting with ``m`` or ``metar``:

* ``m`` - Get METAR for your own callsign's location
* ``metar`` - Get METAR for your own callsign's location
* ``m <CALLSIGN>`` - Get METAR for the specified callsign's location
* ``metar <CALLSIGN>`` - Get METAR for the specified callsign's location

How It Works
~~~~~~~~~~~~

1. When a user sends a ``metar`` command (with or without a callsign), the plugin:

   a. Determines the target callsign (either the sender or the specified callsign)

   b. Queries aprs.fi to get the GPS coordinates for that callsign

   c. Uses the AVWX API to find the nearest weather station to those coordinates

   d. Fetches the METAR report for that station

   e. Returns the raw METAR data to the user

2. The plugin requires the callsign to have recent GPS data in aprs.fi for location lookup.

Example Interactions
~~~~~~~~~~~~~~~~~~~

The following examples show typical interactions with the plugin via APRS messages:

Example 1: Get METAR for Your Own Location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You send a message to the APRSD server asking for weather at your current location:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m

The plugin looks up your callsign (N0CALL-1) on aprs.fi, finds your GPS coordinates,
locates the nearest weather station, and responds:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: KJFK 251851Z 36010KT 10SM FEW250 12/03 A3012 RMK AO2 SLP201 T01220028

Example 2: Get METAR Using Full Command
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also use the full "metar" command:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: metar

Response:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: KORD 251855Z 28015G22KT 10SM OVC045 08/M02 A2998 RMK AO2 SLP145 T00831017

Example 3: Get METAR for Another Callsign
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can request weather for a different callsign by including it in your message:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m W1AW

The plugin looks up W1AW's location and responds:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: KBDL 251900Z 27012KT 15SM CLR 10/02 A3001 RMK AO2 SLP168 T01000017

Example 4: Using Short Command with Callsign
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The short "m" command also works with a callsign:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m KB1ABC

Response:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: KBOS 251905Z 35008KT 10SM FEW020 SCT250 11/04 A3005 RMK AO2 SLP178 T01110039

Example 5: Error - Callsign Not Found
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the callsign doesn't have location data in aprs.fi:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m NOCALL

Response:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: Failed to fetch location

Example 6: Error - No Recent GPS Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the callsign exists but has no recent GPS beacons:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m OLDSTATION

Response:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: Failed to fetch location

Example 7: Error - AVWX API Unavailable
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the AVWX API is down or unreachable:

.. code:: text

   From: N0CALL-1
   To: APRSD
   Message: m

Response:

.. code:: text

   From: APRSD
   To: N0CALL-1
   Message: Failed to get the weather

Understanding METAR Reports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The plugin returns raw METAR (Meteorological Aerodrome Report) data. Here's what the elements mean:

* **Station ID** (e.g., ``KJFK``) - 4-letter ICAO airport identifier
* **Date/Time** (e.g., ``251851Z``) - Day 25th, 18:51 UTC
* **Wind** (e.g., ``36010KT``) - Wind from 360° at 10 knots
* **Visibility** (e.g., ``10SM``) - 10 statute miles
* **Clouds** (e.g., ``FEW250``) - Few clouds at 25,000 feet
* **Temperature/Dewpoint** (e.g., ``12/03``) - 12°C / 3°C
* **Altimeter** (e.g., ``A3012``) - 30.12 inches of mercury

For more information on reading METAR reports, see `METAR format documentation`_.

.. _METAR format documentation: https://en.wikipedia.org/wiki/METAR


Contributing
------------

Contributions are very welcome.
To learn more, see the `Contributor Guide`_.


License
-------

Distributed under the terms of the `MIT license`_,
*AVWX Api based APRSD plugins* is free and open source software.


Issues
------

If you encounter any problems,
please `file an issue`_ along with a detailed description.


Credits
-------

This project was generated from `@hemna`_'s `APRSD Plugin Python Cookiecutter`_ template.

.. _@hemna: https://github.com/hemna
.. _Cookiecutter: https://github.com/audreyr/cookiecutter
.. _MIT license: https://opensource.org/licenses/MIT
.. _PyPI: https://pypi.org/
.. _APRSD Plugin Python Cookiecutter: https://github.com/hemna/cookiecutter-aprsd-plugin
.. _file an issue: https://github.com/hemna/aprsd-avwx-plugin/issues
.. _pip: https://pip.pypa.io/
.. _avwx-engine: https://github.com/avwx-rest/avwx-engine
.. _https://info.avwx.rest/: https://info.avwx.rest/
.. github-only
.. _Contributor Guide: CONTRIBUTING.rst
.. _Usage: https://aprsd-avwx-plugin.readthedocs.io/en/latest/usage.html
