# Fabric AI Config
A collection of resources I use with [danielmiessler/Fabric](fabric-ai-repo).

## Installation

See the main Fabric repository for [installation instructions](https://github.com/danielmiessler/Fabric?tab=readme-ov-file#from-source).

Fabric installs its library of patterns to `~/.config/fabric/patterns/` by default. 

> [!NOTE]
> After updating the CLI, use `fabric --setup` to use a tool that will update the patterns directory.

> [!NOTE]
> Custom patterns no longer need to be copied into the main patterns directory.  There is a tool in
> the `fabric --setup` menu that can be used to point Fabric to a custom patterns directory.

> [!NOTE]
> Save the `~/.config/fabric/.env` file before deleting/re-installing the `~/.config/fabric/`
> directory to preserve your API keys and other settings.



[fabric-ai-repo]: https://github.com/danielmiessler/Fabric
