
# Replace substring

```bash
# -e for 'execute'
# -i for in place
# s/ for substitute
# g for 'global', or whole line
sed -e -i 's/old/new/g' file.json
```

# Delete lines in a file containing a string

```bash
sed -i '/pattern/d' file.json
# More complex escaping example
# checks for http://hello-world.com
sed -i '/http:\/\/hello-world.com/d' file.json
```