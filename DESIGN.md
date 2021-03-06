# Design Guide for Simple ROM Manager

## Overview

The Simple ROM Manager (SRM) is designed to help verify, rename, prune and re-pack sets or ROM files. There is at least one public group that provides authoritative DAT files that will be used as the foundation of deciding what a correct ROM set should contain. At first, SRM will target the No-Intro DAT sets and expand to other groups if there is sufficient demand.

## Project Goals

To be a useful project SRM will try and accomplish the following goals through a simple command-line interface.

* Locate, download and update published No-Intro ROM set DAT files.
  * Plan to support ROM DATs from other release groups.
* Read ROMs from ZIP or 7z archives and allow expansion to other formats.
  * Support single and multiple cloned ROMs in a single archive.
  * Nesting archives will have limited support, or not allowed if possible.
* Perform CRC, MD5, and SHA1 checks on ROM files.
* Identify missing ROMs from a set.
* Identify and remove invalid ROMs from a set.
* Identify and remove duplicate ROMS from a set.
* Rename ROMs and archives to canonical name.
* Re-pack ROMs and clones into a single archive.
* Un-pack ROMs and clones into multiple archives.
* Search DAT files for ROMs by name, hashes, size, region.
* Print reports of ROM sets or groups of ROM sets.


## Interface

The goal of the interface is to be convenient and concise but flexible. This will be accomplished by using sub-commands, command flags, and on-disk configuration defaults. Default command options should provide the most common configuration to reduce the number of necessary options.

### Command Examples

#### Romset

The `romset` command tells SRM what ROM set is contained in a specific directory.

    srm romset [--nointro] <rom_set> [<directory>]
    srm romset --add [--nointro] <rom_set> [<directory>]
    srm romset --delete [--nointro] <rom_set> [<directory>]

In this example, `--nointro` is an optional parameter to select the ROM DAT release group. `<rom_set>` is either a string or TAG name that selects the ROM set. SRM will have a default list of know tags, but also attempt to support ROM set names it does not previously know if a release group provides a list of ROM DATs to chose from. `<directory>` is an option parameter to set the root directory of the ROM files. The default value is the current directory.

When using the `romset` command, configuration values will be written to disk to inform SRM the default ROM set to use for any commands that require a ROM set define. When a user has a tree of different ROM sets, this allows the user to easily switch between directories and issue commands or apply a single command to all sub-directories at once.

The `romset` command syntax was designed to allow multiple ROM sets to be set on the same directory using the `--add` and `--delete` switches. Support for this feature is not a priority at this time.

#### Update

The `update` command lets SRM check and update the current list of ROM DATs it is tracking.

    srm update [--recursive] [<directory>]

The optional `<directory>` option limits the update to group release set(s) reference in the specific directory. If not specified the current directory is used. The optional `--recursive` flag can but used to update all ROM DATs in all sub-dirs of `<directory>`.

#### Status

The `status` command display current settings and configuration. The information displayed will include a directory listing with the configured ROM sets and summary of any data saved or generated by SRM.

    srm status [--recursive] [<directory>]

The `--recurvie` and `<directory>` parameters behave just like the same parameters from `update`.

The status command does not perform long-running operations, therefore all of the values must be cached be easy to calculate. For example, no ROM checksum values are calculated, but certain statistics will be return like "last scan date", "tracked group release DATs", "ROM DAT version", "ROM DAT update time", "total files in ROM DAT set", "total files", "total ROM sizes from DAT", "total ROM size on disk",  "total valid ROMs", "total invalid ROMS", "total duplicate ROMS".

    srm config

    srm check

    # list ROMs from the ROM set
    srm ls

    srm report

    srm find

    srm join

## Configuration

There will be two levels of configuration; "global" and "local" (in the ROM directory). The config files will be split into sections. A subset of the sections will only be valid in the "local" location. The other sections will be valid in both levels. Configs from the "local" location will always circumvent "global" ones.

The common "global" and "local" configs can be changed with the `config` and `config --local` command respectively. "local" configurations can be changed only through the specific SRM commands that set the local configuration.

Users may manually edit the config files for additional control.

## Storage

The global configuration file is stored in `~/.local/srm.conf`. Local configuration files are stored at `<dir>/.srm/config`.

ROM set DAT files will be stored in `<dir>/.src/<group>-<romset>-<version>.[dat|xml]`.

Cached directory and/or runtime data will be stored in `<dir>/srm/cache`. The format of this data has not been determined. The logical choice should be easily readable and portable.

## User Stories

### Single Group Release Collecting

A user has a preferred ROM release group and they want to maintain a correct set over time. This include collecting new ROMs, removing bad ones, and renaming ROMs as needed. Updated ROM release DATs will need to be retrieved from online sources either automatically or manually.

The user may want to add or remove sets of ROMs in the future but keep the same workflow.

### Multiple Sets/Release Collecting

Similar to the previous story, but the user wants to build a collection from multiple group release DATs or combine multiple systems into a single set. The goals is to have the union of all group release sets. The behavior should be like above but the user can attach multiple group release DATs to the same directory.

### Testing Third-party Collections

The user has temporary directory of ROMs obtained from removable media or other means. Their goal is to generate a report of the current set. The user wants to run a single command to report on the set and is not interested in storing any configuration data with the set.
