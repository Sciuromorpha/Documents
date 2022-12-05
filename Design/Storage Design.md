# Storage Design

Basic idea: every micro service will share a root volume/mount point.

* `storage root path` will be a config section;
* `relative path` will be storaged in metadata;
* `service path` and `service managed path` will be extra keys in `document` metadata section;

```
|                                full/absolute path                               |
|       storage path     |                  meta relative path                    |
|                service path                 |        service managed path       |
|       storage path     |  service relative  |  service managed path | files...  |
/data/save/volume/or/path/service/meta/context/service/managed/subpath/filename.ext
```

For grabber workers, it will ask Storage service and get their "service working path" folder, and manage subfolder themselfs, report downloaded files' metadata to meta service.

For parse workers, it need to ask Storage service for the metadata "real path", and try to read the file depended and update the metadata.

To import files already downloaded, the scanner worker will try to read every file/folder and generate metadata for indexing.

    IMPORTANT: If these microservers running in different containers, need ensure that the "data storage volume" are shred between containers and mounted to the same mountpoint. Otherwise these workers cannot read/write these files.

## Save Resource Flow

```mermaid
graph TD
  GetServiceFolder--Service Context-->GenerateServiceFolder-->MakeServiceFolder-->Return
  Download-->GenerateSubfolderByMeta-->MakeSubfolder-->WriteResourceToFile-->WriteMetadata
  Scan/Import-->GenerateMetadata-->WriteMetadata
```

## Load Resource Flow

```mermaid
graph TD
  GetFullPath--Metadata-->GenerateFullpath-->ReadFile
```