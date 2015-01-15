Each module can install and upgrade database tables. The GroupOffice framework 
will automatically apply patch files.

Database files must be put in this path:

/{Namespace}/{ModuleName}/Database/Install

You can put in php or sql files and the name must be in this format:

YYYYMMDD-HHMM.sql or YYYYMMDD-HHMM.php

For example:

20141223-1033.sql

When the upgrade process starts it will gather all module patch files and they 
will be applied in chronological order. This is done so because a patch file might
rely on other module updates when they depend on eachother.

The number of applied patches per module are stored in the modulesModule.version
 column.

Installing and upgrading is basically the same process. The first patch file is
the initial database.


# The initial database


> ### Naming convention
> The tables must be named lowerCamelCase and it should start with the module name. The tables should be in the singular form.
>
> **Wrong:** BandsBands
>
> **Right**: bandsBand

This is the diagram of the database:

![Database diagram](/groupoffice-docs/img/bands-db.png "Database diagram")

Put the following SQL code in Install/Database/20150115-1423.sql:

````````````````````````````````````````````````````````````````````````````````
CREATE TABLE IF NOT EXISTS `bandsAlbum` (
`id` int(11) NOT NULL,
  `bandId` int(11) NOT NULL,
  `name` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `ownerUserId` int(11) NOT NULL,
  `createdAt` datetime NOT NULL,
  `modifiedAt` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci AUTO_INCREMENT=1 ;


CREATE TABLE IF NOT EXISTS `bandsBand` (
`id` int(11) NOT NULL,
  `name` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `ownerUserId` int(11) NOT NULL,
  `createdAt` datetime NOT NULL,
  `modifiedAt` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci AUTO_INCREMENT=1 ;

ALTER TABLE `bandsAlbum`
 ADD PRIMARY KEY (`id`), ADD KEY `bandId` (`bandId`), ADD KEY `ownerUserId` (`ownerUserId`);

ALTER TABLE `bandsBand`
 ADD PRIMARY KEY (`id`), ADD KEY `ownerUserId` (`ownerUserId`);


ALTER TABLE `bandsAlbum`
MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;

ALTER TABLE `bandsBand`
MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;

ALTER TABLE `bandsAlbum`
ADD CONSTRAINT `bandsAlbum_ibfk_2` FOREIGN KEY (`ownerUserId`) REFERENCES `authUser` (`id`),
ADD CONSTRAINT `bandsAlbum_ibfk_1` FOREIGN KEY (`bandId`) REFERENCES `bandsBand` (`id`) ON DELETE CASCADE;

ALTER TABLE `bandsBand`
ADD CONSTRAINT `bandsBand_ibfk_1` FOREIGN KEY (`ownerUserId`) REFERENCES `authUser` (`id`);
````````````````````````````````````````````````````````````````````````````````


Since we already installed the module before we need to increase the module 
version manually now otherwise it will attempt to run this SQL file on the next update action.
Alternatively you can remove the bands tables and run update to add them again.