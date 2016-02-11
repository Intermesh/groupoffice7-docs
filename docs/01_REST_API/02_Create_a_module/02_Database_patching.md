Each module can install and upgrade database tables. The GroupOffice framework 
will automatically apply patch files.

Database files must be put in this path:

/Path/To/Module/Database/Install

You can put in php or sql files and the name must be in this format:

YYYYMMDD-HHMM.sql or YYYYMMDD-HHMM.php

For example:

20141223-1033.sql

When the upgrade process starts it will gather all module patch files and they 
will be applied in chronological order. This is done so because a patch file might
rely on other module updates when they depend on each other.

The number of applied patches per module are stored in the modules_module.version
column.

Installing and upgrading is basically the same process. The first patch file is
the initial database.


# The initial database


> ### Naming convention
> The tables must be named in lowercase and underscores and it should start with 
> the module name. The tables should be in the singular form.
> The Models will automatically use this name in most cases because it will use
> All the parts of the namespaces except for Model and GO\Modules.
> eg. GO\Modules\Bands\Model\Band will automatically use the table name "bands_band".
>
> **Wrong:** bands_bands
>
> **Right**: bands_band

This is the diagram of the database:

![Database diagram](/img/bands-db.png "Database diagram")

Put the following SQL code in Install/Database/20150115-1423.sql:

````````````````````````````````````````````````````````````````````````````````
CREATE TABLE `bands_band` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `ownedBy` int(11) NOT NULL,
  `createdAt` datetime NOT NULL,
  `modifiedAt` datetime NOT NULL,
  `deleted` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `ownedBy` (`ownedBy`),
  KEY `deleted` (`deleted`),
  CONSTRAINT `bands_band_ibfk_1` FOREIGN KEY (`ownedBy`) REFERENCES `auth_user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;


CREATE TABLE `bands_album` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `bandId` int(11) NOT NULL,
  `name` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `ownedBy` int(11) NOT NULL,
  `createdAt` datetime NOT NULL,
  `modifiedAt` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `bandId` (`bandId`),
  KEY `ownedBy` (`ownedBy`),
  CONSTRAINT `bands_album_ibfk_1` FOREIGN KEY (`bandId`) REFERENCES `bands_band` (`id`) ON DELETE CASCADE,
  CONSTRAINT `bands_album_ibfk_2` FOREIGN KEY (`ownedBy`) REFERENCES `auth_user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
````````````````````````````````````````````````````````````````````````````````


Now do a get request to the /system/upgrade route and it will execute this new sql
file.

The response should be:

``````````````````
{
  "success": true
}
``````````````````