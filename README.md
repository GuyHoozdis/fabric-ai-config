# Fabric AI Config
A collection of resources I use with [danielmiessler/Fabric](fabric-ai-repo).

## Installation

See the main Fabric repository for [installation instructions](https://github.com/danielmiessler/Fabric?tab=readme-ov-file#from-source).

Fabric installs its library of patterns to `~/.fabric/patterns/` by default. 

> [!NOTE]
> Installing or updating Fabric _might_ overwrite the `patterns` directory.  Any custom patterns
> should be added into that directory after installation or update.

> [!NOTE]
> Symbolic links didn't work the last time I tried them, so the files have to be copied for now.


```bash
$ tar -czf - custom-patterns/ \
    | tar -xzf - -C ~/.fabric/patterns/
```


[fabric-ai-repo]: https://github.com/danielmiessler/Fabric
