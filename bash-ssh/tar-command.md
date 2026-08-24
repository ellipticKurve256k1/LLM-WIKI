## Intro
Unlike zip in window that does both archive files into single file and compress, linux has two separated process.

1. archive files into one single file ending with `.tar`
	1.1 tar: tape archive
2. compress the file size, ending with `.gz`

## tar command

### tar
#### options:
-c: create a new archive
-z: compress with gzip
-f: specify the archive file name
-t: list the contents of an archive
-v: show details while runnin
-x: extract zip

Example
```sh
tar -czf ../$dir.tar.gz \
--exclude='./.*' \  
--exclude="$dir.tar.gz" \
. # arhive and zip every files in this current directory excluding above
```








