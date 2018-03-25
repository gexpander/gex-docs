# GEX's config files

GEX's configuration is stored in the internal flash memory in a packed binary form.
To make its editing easier for the user, the configuration is accessible as INI files
annotated by numerous comments explaining the individual config options.

## Filesystem access

There are two config files: `UNITS.INI` and `SYSTEM.INI`.

To access those files, the user presses the LOCK button (or removes the LOCK jumper),
which enables an emulated FAT16 storage that will appear on the host computer as a USB disk.
Some hardware platforms require that the jumper be removed before plugging in the USB cable,
as it can't trigger re-enumeration without external circuitry, which is missing on those
evaluation boards.

It's recommended to copy the INI files to a real disk before editing, as some editors create
temporary back-up files that can confuse the filesystem emulator. However, in-place editing
is also possible with some editors.

*This method is the only way of accessing the `SYSTEM.INI` file, which can reconfigure
the main communication port.*

After writing the changed files back to the virtual disk, it will briefly disappear
and an updated version of the files will become available for review.
This is when you can check for error messages.

To confirm the config modifications (they take effect imemdiately, but are not written to flash yet),
push the LOCK button again (or replace the LOCK jumper).

## API access

The `UNITS.INI` file may be accessed through the communication port.

The file is read using Bulk Read (see [FRAME_FORMAT.md](FRAME_FORMAT.md)) after sending a `INI_READ` request.

The modified file is written back using Bulk Write after sending a `INI_WRITE` request.

After writing the changed configuration, it may be persisted to flash using a `PERSIST_CFG` command.
This is not required if the new configuration will be used only temporarily; the original settings
will be restored after a restart.

## UNITS.INI file structure

The `UNITS.INI` file is interactive. The general layout is as follows:

```ini
## UNITS.INI
## GEX v0.0.1 on STM32F072-HUB
## built Mar 17 2018 at 17:53:15

# Overwrite this file to change settings.
# Press the LOCK button to save them to Flash.

[UNITS]
# Create units by adding their names next to a type (e.g. DO=A,B),
# remove the same way. Reload to update the unit sections below.

# Digital output
DO=
# Digital input with triggers
DI=btn,btn2
# (... more unit types here)

[DO:btn@1]
# Port name
port=B
# Pins (comma separated, supports ranges)
pins=2
# Initially high pins
initial=2
# Open-drain pins
open-drain=
```

- The keys in the `[UNITS]` section are lists of instances for each unit type.
  After adding a name to the list and saving the file, a new unit section will appear below, offering to configure
  the new unit as needed.
- Each unit section starts with `[type:name@callsign]`.The name must match the name used in the unit lists above.
  The callsign can be freely changed, as long as there is no conflict. Callsign 0 is forbidden and will be changed
  to a new unique callsign on save.
- If an error occurs during unit initialization, an error message will appear under the unit's section header.

## GEX reconfiguration workflow

1. Enable the mass storage using the LOCK button or jumper (or use the API for reading and writing the file)
2. Open the `UNITS.INI` file in a text editor
3. If adding or removing unit instances, add their names to the right type (eg. `DO=LED`), save and reload the file.
   On some systems, it's necessary to close the editor right after saving and then reopen it again after the disk
   refreshes.
4. Configure the units as needed; adjust callsigns if desired. Save and reload the file.
5. Check for error messages under unit headers. Correct any problems. Save and reload to apply any changes.
6. Push the LOCK button or replace the jumper to persist all changes to flash. If using the API, issue
   the `PERSIST_CFG` command for the same effect.

This is much easier when done through the API, perhaps using the PyQt GEX configuration utility included as an
example in the Python client repository.
