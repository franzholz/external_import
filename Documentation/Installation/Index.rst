﻿.. include:: ../Includes.txt


.. _installation:

Installation
------------

Installing this extension does nothing in and of itself. You still
need to extend the TCA definition of some tables with the appropriate
syntax and create specific connectors for the application you want to
connect to.

Automating the imports requires system extension "scheduler" or setting up
cron jobs calling the command-line controller.

TYPO3 CMS 7 or above is required. The command-line controller is only
available as of TYPO3 CMS 8.


.. _installation-compatibility:

Compatibility issues
^^^^^^^^^^^^^^^^^^^^


.. _installation-upgrade-400:

Upgrade to 4.0.0
""""""""""""""""

.. _installation-upgrade-400-importer-api:

Importer API changes
~~~~~~~~~~~~~~~~~~~~

The External Import configuration is now fully centralized in a :class:`\\Cobweb\\ExternalImport\\Domain\\Model\\Configuration`
object. Every time you need some aspect of the configuration, you should get it via the instance
of this class rather than through any other mean. The most current use case was getting the
name of the current table and index from the :class:`\\Cobweb\\ExternalImport\\Importer` class,
using :code:`Importer::getTableName()` and :code:`Importer::getIndex()`. Such methods
were deprecated and should not be used anymore. Use instead:

  .. code-block:: php

		$table = $importer->getExternalConfiguration()->getTable();
		$index = $importer->getExternalConfiguration()->getIndex();


The :code:`Importer::synchronizeData()` method was renamed to :code:`Importer::synchronize()` and
the :code:`Importer::importData()` method was renamed to :code:`Importer::import()`. The old methods
were kept, but are deprecated.

The :code:`Importer::synchronizeAllTables()` method should not be used anymore as it does not allow
for a satisfying reporting. Instead a loop should be done on all configurations and
:code:`Importer::synchronize()` called inside the loop. See for example
:code:`\Cobweb\ExternalImport\Command\ImportCommand::execute()`.

Other deprecated methods are :code:`Importer::getColumnIndex()` and :code:`Importer::getExternalConfig()`.

The :code:`Importer::getExistingUids()` method was moved to a new class called
:class:`\\Cobweb\\ExternalImport\\Domain\\Repository\\UidRepository` (which is a Singleton).


.. _installation-upgrade-400-transformation-properties:

Transformation properties
~~~~~~~~~~~~~~~~~~~~~~~~~

All column properties that are related to the "Transform data" scope have been grouped into a new
property called :ref:`transformations <administration-columns-properties-transformations>`.
This is an ordered array, which makes it possible to use transformation properties several times
on the same field (e.g. calling several user functions) and to do that in a precise order.
As an example, usage of such properties should be changed from:

  .. code-block:: php

		$GLOBALS['TCA']['fe_users']['columns']['starttime']['external'] = array(
				0 => array(
						'field' => 'start_date',
						'trim' => true
						'userFunc' => array(
								'class' => \Cobweb\ExternalImport\Task\DateTimeTransformation::class,
								'method' => 'parseDate'
						)
				)
		);


to:

  .. code-block:: php

		$GLOBALS['TCA']['fe_users']['columns']['starttime']['external'] = array(
				0 => array(
						'field' => 'start_date',
						'transformations => array(
								10 => array(
										'trim' => true
								),
								20 => array(
										'userFunc' => array(
												'class' => \Cobweb\ExternalImport\Task\DateTimeTransformation::class,
												'method' => 'parseDate'
										)
								)
						)
				)
		);


If you want to preserve "old-style" order, the transformation properties were called in the
following order up to version 3.0.x: "trim", "mapping", "value", "rteEnabled" and "userFunc".
Also note that "value" was ignored if "mapping" was also defined. Now both will be taken into
account if both exist (although that sounds rather like a configuration mistake).

A compatibility layer ensures that old-style transformation properties are preserved, but
this is a temporary convenience, which will be removed in the next version. So please upgrade
your configurations.

.. note::

   The upgrade wizard from version 3.0.0 has been removed. If you are upgrading from TYPO3
   6.2 to TYPO3 8.7, you must go through TYPO3 7.6 first and use the upgrade wizard from
   External Import 3.0.x before moving on to TYPO3 8.7.


.. _installation-upgrade-400-renamed-properties:

Renamed prooperties
~~~~~~~~~~~~~~~~~~~

To continue the move to unified naming conventions for properties started in version 3.0,
the mapping and MM properties which had underscores in their names were moved to
lowerCamelCase name.

The old properties are interpreted for backwards-compatibility, but this will be dropped
in the next major version. The backend module will show you the deprecated properties.


.. _installation-upgrade-400-breaking-changes:

Breaking changes
~~~~~~~~~~~~~~~~

While all hooks were preserved as is, in the sense that they still receive a back-reference
to the :class:`\\Cobweb\\ExternalImport\\Importer` object, the :code:`processParameters`
hook was modified due to its particular usage (it is called in the backend module,
so that processed parameters can be viewed when checking the configuration).
It now receives a reference to the :class:`\\Cobweb\\ExternalImport\\Domain\\Model\\Configuration`
object and not to the :class:`\\Cobweb\\ExternalImport\\Importer` object anymore.
Please update your hooks accordingly.


.. _installation-upgrade-300:

Upgrade to 3.0.0
""""""""""""""""

The "excludedOperations" column configuration, which was deprecated since
version 2.0.0, was entirely removed. The same goes for the "mappings.uid_foreign"
configuration.

More importantly the Scheduler task was renamed from :class:`tx_externalimport_autosync_scheduler_Task`
to :class:`\\Cobweb\\ExternalImport\\Task\\AutomatedSyncTask`. As such, existing
Scheduler tasks need to be updated. An upgrade wizard is provided in the
Install Tool. It will automatically migrate existing old tasks.

.. figure:: ../Images/UpdateWizard.png
	:alt: The update wizard shows that there are tasks to update

If there are no tasks to migrate, the External Import wizard will simply not show up.
Otherwise just click on the "Execute" button and follow the instructions.

Several general TCA configuration properties were renamed, to respect a global
lowerCamelCase naming convention. This is the list of properties and how they
were renamed:

- additional\_fields => additionalFields
- reference\_uid => referenceUid
- where\_clause => whereClause


.. _installation-upgrade-200:

Upgrade to 2.0.0
""""""""""""""""

The column configuration "excludedOperations" has been renamed to
"disabledOperations", for consistency with the table configuration
option. The "excludedOperations" is preserved for now and will log an
entry into the deprecation log. You are advised to change the naming
of this configuration if you use it, support will be dropped at some
point in the future.


Other requirements
^^^^^^^^^^^^^^^^^^

As was mentioned in the introduction, this extension makes heavy use
of an extended syntax for the TCA. If you are not familiar with the
TCA, you are strongly advised to read up on it in the
:ref:`TCA Reference manual <t3tca:start>`.
