# Salvaged M17 Reverse-Engineered Design Notes

These documents from June 2022 were created mainly by me (Paul Williamson KB5MU) in the process of developing an initial understanding of the `m17-cxx-demod` code base [m17-cxx-demod repo](https://github.com/mobilinkd/m17-cxx-demod). Nothing here should be taken as authoritative in any way.

Shortly after these initial drafts of documents were created, the main emphasis of the project shifted from understanding the M17 code base to converting it to implement Opulent Voice instead of M17. The work to create detailed documentation of the M17 code fizzled out at that point.

The files here were either never intended for publication at all, or never finished. They are provided here in the hope that they might still be useful.

## Summary of Salvaged Files

* [m17-demod design notes.md](m17-demod%20design%20notes.md) was to be the beginning of an actual readable set of design notes for the M17 code. Unfortunately, all that was completed was an overview and a list of files with short descriptions.

* [m17-demod_notes.txt](m17-demod_notes.txt) is a narrative read-through of the entire receive code base, in some sort of logical execution order. These represent my initial understanding of the code as I was studying it, not for the first time, but in detail with the intent to really comprehend most details. There were too many details to keep in my head, so I resorted to writing them down as I went along. As a result, these notes are very informal and probably not very accurate, and were never written for any audience other than myself.

* [m17-demod-initial-notes.txt](m17-initial-notes.txt) are the beginning of an even rougher set of read-through notes describing the receive code. I snapshotted this version before going back and morphing it into something more like what you see in `m17-demod_notes.txt`.

* [M17 syncword re notes.txt](M17%20syncword%20re%20notes.txt) is a similar set of rougher notes for just the syncword processing in M17.

* [M17 State Diagram.drawio.svg](M17%20State%20Diagram.drawio.svg) is a drawing of the top-level state machine in the M17 receiver, as reverse engineered. It was drawn with draw.io (probably the app but maybe with the online webapp) and output in SVG format for publication.

* [M17 State Diagram.drawio](M17%20State%20Diagram.drawio) is the draw.io source file for the above SVG file.