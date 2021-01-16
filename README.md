# AlbianWarp - Game Modifications

This repository contains the Creatures 3/Docking Station Game modifications needed for AlbianWarp to work.

## Installation

The Bootstrap directory must be copied into the Docking Station installation directory. The Bootstrap directory will then integrate into the original Bootstrap directory.

Your install directory may vary. If you downloaded and installed Creatures Exodus from gog.com with the default settings, the standard installation directory might be `C:\GOG Games\Creatures Exodus\Docking Station`. If you installed the game another way, or into your Documents, this might be something like `C:\Users\USERNAME\Documents\Creatures\Docking Station`.

The bootstrap should only overwrite one file at most-- the file called 'reinject_updated_agents.cos' in the 'dsbuild 195' directory. A backup of this file is provided in the same directory should you wish to restore it later. Note that restoring the file will disable the functionality of Albian Warp.

The My Agents folder can also be copied into your Docking Station installation directory, but it is not required. It contains only an agent you can use to update already existing worlds to the version of Albian Warp currently in your bootstrap. Please back up worlds that are important to you before injecting it, as it has not been well tested. Newly created worlds will not need this agent.

Unless you are developing, you can ignore the Misc. folder.

## Usage

The Game Modification by itself are not very useful, as they are only used by the [AlbianWarpClient](https://github.com/AlbianWarp/AlbianWarpClient).

## License

This project is licensed under the GPL3 License - see the [LICENSE.md](/LICENSE.md) file for details
