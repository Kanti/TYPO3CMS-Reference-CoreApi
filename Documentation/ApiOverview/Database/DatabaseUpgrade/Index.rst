.. include:: ../../../Includes.txt


.. _database-upgrade:

Upgrade table and field definitions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each extension in TYPO3 CMS can bring the file :file:`ext_tables.sql` that
defines which tables and fields the extension needs. Gathering all
:file:`ext_tables.sql` thus defines the full set of tables, fields and
indexes of a TYPO3 instance to unfold its full feature set. Some functionality
in the install tool can compare the defined set with the current active
database schema and shows options to align those two by adding fields,
removing fields and so on.

When you upgrade to newer versions of TYPO3 CMS or upgrade an extension,
the data definition of tables and fields might have changed.
The TYPO3 CMS Install Tool will detect such changes.

When you install a new extension, any change to the database
is automatically performed. When you upgrade to a new major
version of TYPO3 CMS, you should normally go through the Upgrade
Wizard, whose first step is to perform all necessary database
changes:

.. figure:: ../../../Images/DatabaseUpgradeWizard.png
   :alt: The Upgrade Wizard indicating that the database needs updates

   The Upgrade Wizard indicating that the database needs updates


When performing smaller updates, after updating extensions or - in
general - if you want to check the sanity of your system,
you can go to **Admin tools > Maintenance > Analyze Database Structure**:

.. figure:: ../../../Images/DatabaseDatabaseAnalyzer.png
   :alt: Analyze Database Structure of the Install Tool

   The Database analyzer is part of the Maintenance area


What this tool does is collating the information from all
:file:`ext_tables.sql` files of active extensions and compare it with
the current database structure. It then proposes to perform the necessary changes,
grouped by type: creating new tables, adding new fields to existing
tables, altering existing fields, dropping unused tables and fields.

You can choose which updates you want to perform. You can even
decide not to create new fields and tables, although that will
very likely break your installation.

More information about the process of upgrading TYPO3 CMS can be found in
the :ref:`Installation and Upgrade Guide <t3install:upgrade>`.


.. _database-exttables-sql:

The ext\_tables.sql files
"""""""""""""""""""""""""

As mentioned before, all data definition statements are stored in
files called :file:`ext_tables.sql` which may be present in any
extension.

The peculiarity is that these files may not always contain
a complete and valid SQL data definition. For example,
system extension "rsaauth" defines a new table for storing
RSA keys:

.. code-block:: sql

   CREATE TABLE tx_rsaauth_keys (
      uid int(11) NOT NULL auto_increment,
      pid int(11) DEFAULT '0' NOT NULL,
      crdate int(11) DEFAULT '0' NOT NULL,
      key_value text,

      PRIMARY KEY (uid),
      KEY crdate (crdate)
   );

This is a complete and valid SQL data definition. However
system extension "css\_styled\_content" extends the "tt_content"
table with additional fields. It also provides these changes
in the form of a SQL :code:`CREATE TABLE` statement:

.. code-block:: sql

   CREATE TABLE tt_content (
      header_position varchar(6) DEFAULT '' NOT NULL,
      image_compression tinyint(3) unsigned DEFAULT '0' NOT NULL,
      image_effects tinyint(3) unsigned DEFAULT '0' NOT NULL,
      image_noRows tinyint(3) unsigned DEFAULT '0' NOT NULL,
      section_frame int(11) unsigned DEFAULT '0' NOT NULL,
      spaceAfter smallint(5) unsigned DEFAULT '0' NOT NULL,
      spaceBefore smallint(5) unsigned DEFAULT '0' NOT NULL,
      table_bgColor int(11) unsigned DEFAULT '0' NOT NULL,
      table_border tinyint(3) unsigned DEFAULT '0' NOT NULL,
      table_cellpadding tinyint(3) unsigned DEFAULT '0' NOT NULL,
      table_cellspacing tinyint(3) unsigned DEFAULT '0' NOT NULL
   );

The classes which take care of assembling the complete SQL data
definition will compile all the :code:`CREATE TABLE` statements
for a given table and turn it into a single :code:`CREATE TABLE`
statement. If the table already exists, missing fields are isolated
and :code:`ALTER TABLE` statements are proposed instead.

What this means is that - as an extension developer - you should always
have only :code:`CREATE TABLE` statements in your :file:`ext_tables.sql`
files, the system will handle them as needed.
