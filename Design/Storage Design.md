# Storage Design


```
+----------------+     +----------+     +------------------+
| Input Resource | --> | Metadata | --> | Metadata Storage |
+----------------+     +----------+     +------------------+
                            |    +-----------------+     +--------------+
                            +--> | Blob (FileLike) | --> | Blob Storage |
                                 +-----------------+     +--------------+
```


## Save Resource Flow

```mermaid
graph TD
  SaveResource--Resource-->SaveFlow
  SaveFlow--Without Blob-->GenerateMetadata--Timestamp/Exif/etc-->SaveMetadata-->Done
  SaveFlow--With Blob-->SaveBlob--RelativePath/Hash/Bucket/etc-->GenerateMetadata
```

## Load Resource Flow

```mermaid
graph TD
  LoadResource--ID/Hash-->LookupResource-->Metadata
  LoadFile--RelativePath-->LoadBlob-->File_Stream
```