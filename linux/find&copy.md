To find all .jpg files (case-insensitive) and copy them into a single destination folder:

### Copy All .jpg Files (Flattened, No Folder Structure)

```bash
find /source/dir -type f -iname '*.jpg' -exec cp '{}' /destination/dir/ \;
```

*    -type f: Only files.

 *   -iname '*.jpg': Matches .jpg and .JPG,         case-insensitively.

 *   cp '{}' /destination/dir/: Copies to one flat folder (no structure preserved).



### Preserve Folder Structure While Copying .jpg Files

* If you want to keep the original folder hierarchy:

```bash
find . -type f -iname '*.jpg' -exec cp --parents '{}' /destination/dir/ \;
```