# Islandora Backup Discussion Document

Preservation within the repository context utilizes a variety of strategies to ensure the successful longterm stewardship of assets.  

These activities<sup>1</sup> typically include:

     * archival formats
     * normalization
     * format migration
     * _bitstream copying_
     * fixity checking
     * documentation of file formats

This proposal is an attempt to provide a discussion starter about an offsite/offline bitstream copying strategy related to digital objects within the Islandora context and isn't intended to replace a standard/full system backup plan. 

## Backup Plan

At UPEI our Backup Plan includes a backup to both the network filesystem (spinning disk) and LTO5 tape on the following schedule:

     * daily incremental
     * weekly differential
     * monthly full backups

We use [Bacula](http://www.bacula.org/) to manage our backups.

### What gets backed up?
This section is intend to provide an overview of our existing backup plan.

In addition to our production servers, we backup our database servers, and data created during our digitization workflow (our working data prior to loading into the repo) which resides on the internal network. 

## Implementing a Generic Backup Strategy for Islandora Objects

One of the key features we are missing in our current backup plan is remote storage of assets. We've explored various options and Paul Pound developed the [Vault module](https://github.com/ppound/cirrostratus_assimilate) that utilizes [CloudSync](https://wiki.duraspace.org/display/CLOUDSYNC11/Fedora+CloudSync+1.1) to store Fedora content to a [DuraCloud](https://wiki.duraspace.org/display/DURACLOUDDOC/Release+Notes) instance.

### Proposed Use Cases

**Backup Use Cases**
* an islandora admin/repository manager is able to back up their Islandora site (Drupal filesystem, Drupal database, Fedora digital objects)
* a islandora admin/repository manager is able to back up selected portions of their Islandora site (eg. just the Drupal filesystem, and/or the database, and/or the Fedora objects)
* an islandora admin/repository manager can schedule backups
* an islandora admin/repository manager can manually initiate a backup
* aislandora admin/repository manager is able to direct their backup to a multiple destinations. For example:
 * the same filesystem
 * a local network filesystem
 * a cloud service
* the application can leverage multiple methods for transferring backups. For example:
 * FTP
 * SFTP
 * RSync
 * BitTorrent
 * Swift
 * other
* an authorized user is able to construct various backup scenarios. For example:
 * create a backup of the Drupal filesystem and database and download it to their local machine
 * create a backup of Islandora objects and select Amazon S3 as a destination.
* For Islandora Objects, a user may want to select the full FOXML object or only selected datastreams
* A backup of an Islandora object should record a [replication event] (http://id.loc.gov/vocabulary/preservation/eventType/rep.html) to the AUDIT datastream
 * Additional preservation metadata should be recorded as part of the backup process (eg. a checksum of files at time of backup should be recorded and compared)

**Restore Use Cases**

* an authorized user is able to restore from a previous backup
* an authorized user is able to selectively restore an object (eg. restore a single Islandora object)

** Report Use Cases**

* an authorized user can review the list of backups performed.
* an authorized user can compare the checksum of a backed up item with the checksum originally recorded.

### Implementation

![Backup Flow Diagram](https://drive.google.com/a/upei.ca/file/d/0B9XaOKp03SzmeHU3MmdlQTMxNWM/view)

**Backup and Migrate Module** 

Drupal has a well supported module - [Backup and Migrate](https://www.drupal.org/project/backup_migrate) - that provides a flexible backup framework for Drupal sites.  Using an existing Drupal module that provides a fair amount of functionality and support for many of the use cases 'out-of-the-box' may be more sustainable.  Adopting an exist module also allows potential contributions back to the module, and by managing this function in the Drupal layer may make the solution more resilent in the future with Fedora4 integration.

The [Backup and Migrate](https://www.drupal.org/project/backup_migrate) module has default support for:

* Backup/Restore multiple MySQL databases and code
* Backup of files directory is built into this version
* Add a note to backup files
* Smart delete options make it easier to manage  backup files
* Backup to FTP/S3/Email or NodeSquirrel.com
* Drush integration
* Multiple backup schedules
* AES encryption for backups

Given the plugin architecture of the [Backup and Migrate](https://www.drupal.org/project/backup_migrate) module, the Drupal community has contributed additional addon modules including:

* Backup and Migrate SFTP - Backup to SFTP (tested this and it doesn't work with the current version of B&M)
* Backup and Migrate Dropbox - Backup to Dropbox (likewise tested this and it doesn't work with the current version of B&M)
* Backup and Migrate Rackspace Cloudfiles - Backup to Rackspace Cloudfiles
* HPCloud - Backup to HPCloud

I had mixed experiences when trying to implement some of these modules, so further investigation is required.

**Islandora BagIt Module**

When you combine the functionality of the [Backup and Migrate](https://www.drupal.org/project/backup_migrate) module with Mark Jordan's [Islandora Bagit](https://github.com/islandora/islandora_bagit) module a number of our Islandora object use cases become possible. The [Islandora Bagit](https://github.com/islandora/islandora_bagit) module packages up Islandora (Fedora) digital objects and their metadata into a format that can be shared between applications/systems. The module provides a plugin architecture for creating bags and includes:

* plugin_object_archivematica_transfers
* plugin_object_ds_add_file
* plugin_object_ds_basic
* plugin_object_foxml
* plugin_object_premis

When utilizing [Islandora Bagit](https://github.com/islandora/islandora_bagit) users can configure where their bags are stored (eg. sites/default/files/bags) which makes them accessible to the [Backup and Migrate](https://www.drupal.org/project/backup_migrate) module.

To get a proof of concept pulled together I think there are two use cases we can investigate:

* individual object packaged as a Bag backed up to a service like Amazon S3 (in theory the current set of modules/functionality should support this)
* serialized objects (eg. an Islandora collection) stored in a single Bag backed up to Amazon Glacier.

**FOXML BagIt Approach**

As a starting point we may want to consider limiting our BagIt configuration to store the FOXML of the object (plugin_object_foxml). This bag would contain the default BagIt files and for the Fedora Object both Inline datastreams and Managed datastreams - all Managed Content datastreams will have the datastream content Base64-encoded within the digital object export file), eliminating the need to develop restore methods based on content models. 

* potential issues
  * the checksums used in the Bag are SHA1, while the Islandora default is MD5 (though SHA1 is an option).
  * the checksum in the Bag is applied to the FOXML file itself, not the individual datastreams
  * there currently isn't a restore for FOXML files, though [Tuque supports the creation of objects based on FOXML](https://github.com/Islandora/tuque/blob/1.x/FedoraApi.php#L1057-L1141).
  * What happens on restore? I think this is potentially snarly.

**Individual Datastreams Approach**

This approach will require additional thought.  The potential benefits of this approach include:

* the user can be selective (eg. only Bag the OBJ, MODS, POLICY and generated PREMIS datastreams).
* each datastream will be stored individually within the Bag and will have its own checksum.
* complicates the restore process. Would we need to embed the associated Islandora CModel as part of the bag? Would we bag up objects based on their content models? eg. use a directory structure to separate those out? Additional layers of admin screens would be needed.
* the B&M module provides a note field for the backup ... would we insert cmodel details there?

**Issues and Further Discussion**
* what is reasonable re: size of backup? Would it need to be chunked / optomized in some way ... how do we accommodate a backup that may take days?
* we don't accommodate backup of the application stack in the proposal
 * eg the fedora app and it's configs, solr configs and index, etc 
* security
 * while in Fedora, object XACML policies are respected. When in a Bag, the original applied security is lost.
* what is the frequency of backups?
* what are the cost factors for an S3 / Glacier implementation?
* the restore piece is thin and needs further thought



1. [Scholar's Portal Preservation Implementation Plan](https://spotdocs.scholarsportal.info/display/OAIS/Preservation+Implementation+Plan)
