
# Git

## Git Credential Manager

NOTE: As of late March 2024, there is an arithmetic overflow in the AUR package due to an upstream bug in how pacman handles .NET self-contained applications. 

Temporary fix is to set:

```
options=("!strip" "!debug")
```

in the PKGBUILD. Be sure to completely clean before rebuilding (an AUR helper cleanbuild is insufficient) if you have already had a failed install from the bug.

