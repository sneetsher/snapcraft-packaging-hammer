# Snapcraft: Reference Pointers

> PLEASE, CHECK LAST TIME WHEN THIS WAS UPDATED. IF IT IS OLD OR YOU JUST STARTED LEARNING SNAP PACKAGING, GO DIRECTLY TO REFERENCES SECTION AND FOLLOW LINKS TO THE UPSTREAM EDGE DOCUMENTATION. NO NEED TO WASTE YOUR TIME AND MEMORY. THE PROJECT WAS STILL UNDER HEAVY DEVELOPMENT.

> USED SNAPCRAFT 3.11 / SNAPD 2.44

## Components [&#8227;][docs]

- `snap` CLI
- `snapd` service
- `snapcraft` CLI
- Snap store

## Commands

### Setup

- snapd [&#8227;][installing-snapd]
    
        sudo apt update
        sudo apt install snapd

- snapcraft

        sudo snap install snapcraft --classic
        sudo snap install multipass --classic
        
        #case KVM not available
        sudo snap install lxd
        lxd init

### Initial Template

    snapcraft init

### Build options [&#8227;][build-options]

    snapcraft clean
    
    snapcraft
    #eq: snapcraft snap
    snapcraft --destructive-mode
    #snapcraft cleanbuild #dep
    snapcraft --use-lxd
    snapcraft remote-build

### Publish & Use
    
    snapcraft login
    snapcraft register
    snapcraft push --release=<channel> <snap-name>.snap
    
              --release=<track>/<risk>/<branch>
              --release=latest|<version>/stable|candidate|beta|edge/...
              or –stable|–candidate|–beta|–edge
              
    sudo snap install <snap-name> 
    sudo snap install --dangerous <snap-name>.snap
    
    sudo snap connect <snap-name>:<plug-name>
    
    <snap-name>.<app>



## Parts lifecycle [&#8227;][parts-lifecycle]

### Steps

1. `pull`: *source*&rarr;`SNAPCRAFT_PART_SRC`
2. `build`: `SNAPCRAFT_PART_BUILD`&rarr;`SNAPCRAFT_PART_INSTALL`
    - *organize*: `SNAPCRAFT_PART_INSTALL`&#10226;
4. `stage`: `SNAPCRAFT_PART_INSTALL`&rarr;`SNAPCRAFT_STAGE`
5. `prime`: `SNAPCRAFT_PART_INSTALL`&rarr;`SNAPCRAFT_PRIME`
6. `snap`: `SNAPCRAFT_PRIME`&rarr;`SNAPCRAFT_PROJECT_DIR`/`<snap-name>.snap` (Compressed SquashFS)

### Building to a step

    snapcraft pull [<part-name>]
    snapcraft build [<part-name>]
    snapcraft stage [<part-name>]
    snapcraft prime [<part-name>]
    snapcraft snap | snapcraft

    #each cmd run all prev steps

### Iterating over a build [&#8227;][iterating-over-a-build]

    #build break down & debug shell
    snapcraft [<step>] --shell|--shell-after [<part-name>]
    snapcraft [<step>] --debug [<part-name>]

    snapcraft clean
    snapcraft build --shell
    snapcraft --debug

    #install without packaging
    snapcraft try
    sudo snap try prime
    sudo snap remove <snap-name>
    
    #package extract
    unsquashfs <snao-name>.snap
    mount <snap-name>.snap <mount-point> -t squashfs -o loop
    
    #strict mode
    snappy-debug.security scanlog
    #devmode
    journalctl -xe


## Project structure

```
. 
├── <source> (root,any-where,remote)
└── snap
    ├── hooks #opt
    │   ├── configure
    │   ├── install
    │   ├── pre-refresh
    │   ├── post-refresh
    │   ├── remove
    │   └── prepare-device #rel type:gadget
    ├── plugins #opt
    └── snapcraft.yaml
```

## snapcraft.yaml [&#8227;][snapcraft-yaml-reference]

### Personal Note

> **Its schema is still evolving, WATCHOUT for:**

> - Deprecations and old documentation all over the web.
> - Wrong naming, example `version:git` means version:source.
> - Waving/crispy model, may be for simplicity (shalow depth), example: each `plugin` has some specific set of parameter names spoiling the parent `part` object. `python`/`python-version`, `qmake`/`qt-version`, `nodejs`/`node-engine`.
> - Hardcoded/forced switches, example `<app>:type` & `<part>:plugin` defines which stanzas can be used in combination.
> - Docummantation listing are not ordered logically. So this paper to be easier to compare for updates.

> YAML format, more structured than Makefile and RPM Spec, flat compared to DEB debian/*.

> **Is worth it packaging as snap?**

> - No:

>     libraries, large frameworks, tools with long release-cycle or easy to build statically, portable apps/deamons GNU/Linux/BSD/MacOS/MinGW, tools which are current with thier language platform pip/npm/...


> - Yes:

>     tools with short release-cycle/rolling that have many end users in supported OS's or have too many dependencies and hard to build statically or affects its performance.


### Demo [&#8227;][snapcraft-format]

`snap/snapcraft.yaml`

```
name: hello
base: core18
version: '2.10'
summary: GNU Hello, the "hello world" snap
description: |
  GNU hello prints a friendly greeting.
grade: stable
confinement: strict

apps:
  hello:
    command: bin/hello

parts:
  gnu-hello:
    source: http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
    plugin: autotools
```

    snapcraft


### Creating snapcraft.yaml [&#8227;][creating-snapcraft-yaml]

*codes:*

```
#mon mondatory
#opt optional
#dep deprecated
#glo global
#rel related to
    
{} dict
e  enumeration
i  integer
[] list
|  multi-line string
o  object
s  string
t  type
    
c command
p path
t time/timer
u uri, url
v variant
```

```
name: s #mon
title: s #opt
base: e core18|core16/core|bare #opt
version: s|git #mon
sammary: s #mon
description: | #mon
type: e app|gadget|kernel|base #opt
confinement: e strict|classic|devmode #opt
icon: s/p #opt
license: s #opt
grade: e stable|devel #opt
adopt-info: s #opt
architectures: [o] #opt
  - build-on: [e] amd64,arm64,armhf,i386,ppc64el,s390x #opt
  - run-on: [e] amd64,arm64,armhf,i386,ppc64el,s390x,all #opt
assumes: [s] #opt
passthrough: {t:o} #opt
epoch: i #opt

apps: {s:o} #opt
  <app-name>: {}
    adapter: e full|none
    command: s/c
    command-chain: [s/c]
    common-id: s
    daemon: e simple|oneshot|forking|notify
    desktop: s/p
    environment: {s:s}
    extensions: [s] gnome-3-28|gnome-3-34|kde-neon
    plugs: [s] #interfaces-plugs
    slots: [s] #interface-slots
    stop-command: s/c
    post-stop-command: s/c
    stop-timeout: s/t
    timer: s/t
    restart-condition: e on-failure|on-success|on-abnormal|on-abort|always|never
    socket: {} #rel plug:network-bind
    socket-mode: i/octal
    listen-stream: s/p
    passthrough: {t:o}

plugs: {s:o} #opt #glo
  <plug-name>: {s:v}

slots: {s:o} #opt #glo
  <slot-name>: {s:v}   

parts: {s:o} #mon
  <part-name>: {}    
    plugin: s #plugin
    source: s/p,u
    source-type: e bzr|deb|git|hg|local|mercurial|rpm|subversion|svn|tar|zip|7z
    source-checksum: s/s md5|sha1|sha224|sha256|sha384|sha512|sha3_256|sha3_384|sha3_512/<digest>
    source-depth: i
    soourc-branch: s
    source-commit: s
    source-tag: s
    source-subdir: s/p
    after: [s] #other-parts
    autostart: s/p
    build-environment: [s]
    build-snaps: [s]
    build-packages: [s]
    stage-snaps: [s]
    stage-package: [s]
    organize: {s/p:s/p}
    fileset: {s:o}
      <set-name>: [s/p] #support * wildcard, - exclude
    stage: [s/p|$<set-name>]
    parse-info: s/p
    prime: [s/p|$<set-name>]
    override-pull: |/c #!/bin/sh; set -e
      snapcraftctl pull  #default
    override-build: |/c #!/bin/sh; set -e
      snapcraftctl build #default
    override-stage: |/c #!/bin/sh; set -e
      snapcraftctl stage #default
    override-prime: |/c #!/bin/sh; set -e
      snapcraftctl prime #default
    build-attributes: e

hooks: #opt
  <hook>: #see project structure
    command-chain: [s/c]
    plugs: [s]
```

- Advanced grammar: `on`,`to`,`try` [&#8227;][advanced-grammar]
- `override-*` & `snapcraftctl` [&#8227;][scriptlets]
- Supported interfaces [&#8227;][supported-interfaces]


### Deprecations [&#8227;][deprecation-notices]

```
01 snap -> prime
02 parts/plugins -> snap/plugins
03 setup/gui -> snap/gui
04 snapcraft history -> snapcraft list-revisions
05 aliases -> snap-store request or 'snap alias' by user
06 snapcraft snap <directory> -> snapcraft pack <directory>
07 prepare -> override-pull, override-build
08 build -> override-build
09 install -> override-build
10 version-script: s/c -> 'snapcraftctl set-version'

```

## Variables

### Build-time - Parts environment variables [&#8227;][parts-environment-variables]

    # Locating directories
    SNAPCRAFT_PROJECT_DIR
    SNAPCRAFT_PART_SRC        parts/<part-name>/src
    SNAPCRAFT_PART_BUILD      parts/<part-name>/build
    SNAPCRAFT_PART_INSTALL    parts/<part-name>/install
    SNAPCRAFT_STAGE           stage
    SNAPCRAFT_PRIME           prime
    
    # Snapcraft configuration
    SNAPCRAFT_ARCH_TRIPLET
    SNAPCRAFT_PARALLEL_BUILD_COUNT
    SNAPCRAFT_PROJECT_NAME
    SNAPCRAFT_PROJECT_VERSION
    SNAPCRAFT_PROJECT_GRADE
    
    # Build flags
    CFLAGS
    CPPFLAGS
    CXXFLAGS
    LDFLAGS
    PKG_CONFIG_PATH
    
    # Plugin variables
    parts:
      <part>:
        build-environment: [s]

### Run-time - Environment variables [&#8227;][environment-variables]

    $ snap run --shell <snap>.<command>
    $>$ env

    SNAP                  RO, install path
    SNAP_ARCH
    SNAP_COMMON
    SNAP_DATA             RW, in /var
    SNAP_INSTANCE_NAME
    SNAP_INSTANCE_KEY
    SNAP_LIBRARY_PATH
    SNAP_NAME
    SNAP_REVISION
    SNAP_USER_COMMON
    SNAP_USER_DATA        RW, in /home
    SNAP_VERSION

    HOME
    PATH
    XDG_RUNTIME_DIR


## Generic launch wrapper

    #!/bin/sh
    exec "$@" --executed-from="$(pwd)" --pid=$$ > /dev/null 2>&1 &
    

## Writing local plugins [&#8227;][writing-local-plugins]

As last resort, Plugin API [&#8227;][snapcraft-plugin-api]

`snap/plugins/<plugin_name>.py`

```     
import snapcraft

class MyPlugin(snapcraft.BasePlugin):

    @classmethod
    def schema(cls):
        schema = super().schema()

        # Add a new property called "my-property"
        schema['properties']['my-property'] = {
            'type': 'string',
        }

        # The "my-option" property is now required
        schema['required'].append('my-property')

        return schema

    def pull(self):
        super().pull()
        print('pulled! "my-property"= {}'.format(
            self.options.my_property))

    def build(self):
        super().build()
        print('built!')
```

`snap/snapcraft.yaml`

```
parts:
  my-part:
    plugin: my-plugin
    my-property: test value
```


## References

- [snapcraft - docs](https://snapcraft.io/docs/)
- [snapcraft - forum](https://forum.snapcraft.io/)
- [snapcraft - youtube](https://www.youtube.com/channel/UCcH6oAZ0FOSVUMUAojHtFBg)
- [ubuntu - blog](https://ubuntu.com/blog/)
- [snapd - github](https://github.com/snapcore/snapd)


## Examples

- [snapcraft-desktop-helpers - github](https://github.com/ubuntu/snapcraft-desktop-helpers)
- [snapcraters - github](https://github.com/snapcrafters)
- [github - snapcraft.yaml](https://github.com/search?q=%22snapcraft.yaml%22&type=Commits)
- [duckduckgo - snapcraft.yaml](https://duckduckgo.com/?q=%22snapcraft.yaml%22+filetype%3Ayaml&t=ffnt&ia=web)


## Authors

### Original

- Abdellah Chelli

### Updates/Contributers 

- You ... may be


[docs]:https://snapcraft.io/docs
[advanced-grammar]:https://snapcraft.io/docs/snapcraft-advanced-grammar
[build-options]:https://snapcraft.io/docs/build-options
[creating-snapcraft-yaml]:https://snapcraft.io/docs/creating-snapcraft-yaml
[deprecation-notices]:https://snapcraft.io/docs/deprecation-notices
[environment-variables]:https://snapcraft.io/docs/environment-variables
[installing-snapd]:https://snapcraft.io/docs/installing-snapd
[iterating-over-a-build]:https://snapcraft.io/docs/iterating-over-a-build
[parts-environment-variables]:https://snapcraft.io/docs/parts-environment-variables
[parts-lifecycle]:https://snapcraft.io/docs/parts-lifecycle
[scriptlets]:https://snapcraft.io/docs/scriptlets
[snapcraft-plugin-api]:https://snapcraft.io/docs/snapcraft-plugin-api
[supported-interfaces]:https://snapcraft.io/docs/supported-interfaces
[writing-local-plugins]:https://snapcraft.io/docs/writing-local-plugins
[snapcraft-format]:https://snapcraft.io/docs/snapcraft-format
[snapcraft-yaml-reference]:https://snapcraft.io/docs/snapcraft-yaml-reference
