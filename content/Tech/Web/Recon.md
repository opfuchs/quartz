See also Recon and OSINT - [[Web]]
# Directories

Get rootdirs from a list of links one-liner:

```
cat links.txt | grep -oP '^https?://(?:[^/]*/){2}' | sort -u | tee root-dirs.txt
```
