# Modules (LMOD)
Software modules are a convenient way for administrators to provide pre-installed software for users using [lmod](https://lmod.readthedocs.io/en/latest/index.html) and [EasyBuild](https://docs.easybuild.io/). Software modules are loaded by manipulating the user's environment where the software is made available by replacing default paths to where binaries are found. This makes it possible to isolate software and all its dependencies from the operating system to ensure compatibility on most platforms. Modules are designed and installed using declarative build scripts, most of which are available from [Easybuilders](https://github.com/easybuilders/easybuild-easyconfigs/tree/develop/easybuild/easyconfigs).

Currently installing new modules is not supported, but modules installed previously are still available through the `module` command. It's recommended to install software yourself using either [conda environments](conda.md) or [containers](containers.md) instead.

## List available modules
```
$ module avail
```

## Search modules
```
$ module spider
```

## Load a module
When loading modules it's important not to load more than one at a time since conflicts between dependencies of the individual tools may arise. If you need more than one, you need to unload any modules loaded previously using `module purge`.
```
$ module load minimap2
```

## Unload a module
```
$ module unload minimap2
```
