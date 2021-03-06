# Commodore Peripheral Bus: Part 0: Overview

The well-known Serial Bus (aka Serial "IEC" Bus) of the Commodore 64 that connects to disk drives such as the 1541 is just one variant of a whole family of busses and protocols used by the line of 8 bit Commodore machines from the PET to the C65. This is the first article of a **multi-part series** on the **Commodore Peripheral Bus family**.

The following figure compares the different protocol stacks. There are three different connectors with their unique byte transfer protocols: IEEE-488, Serial, and TCBM. Fast Serial and JiffyDOS are optimized, but backwards-compatible protocols for existing cables, and CBDOS integrates the drive directly into the computer. The higher level protocols are the same across all Commodore 8 bit machines, including the "KERNAL" operating system APIs.

![](docs/cbmbus/cbmbus.png =601x241)

These are the properties and tradeoffs of the different variants of the protocol stack[^1]:

|                         | IEEE-488 | Serial        | Fast Serial   | JiffyDOS    | TCBM                   | CBDOS       |
|-------------------------|----------|---------------|---------------|-------------|------------------------|-------------|
| Data Wires              | 13       | 3             | 4             | 3           | 12                     | -           |
| Speed (KB/sec)          | 2.1      | 0.4           | 2.1           | 2.1         | 2.4                    | &infin;     |
| Controller Code (bytes) | 334      | 434           | 708           | 739         | 262                    | 0           |
| Comments                | compatible with industry standard | very slow | requires decidated bit shifting hardware | | point-to-point |drive integrated into computer |

All variants are based on the IEEE-488 standard and therefore share (mostly) the same basic architecture:
* All participants are **daisy-chained**.
* **One dedicated controller** (the computer) does bus arbitration of **up to 31 devices**.
* **One-to-many**: Any participant can send data to any set of participants.
* A device has **multiple channels** for different functions.
* Data transmission is **byte stream** based.

The different variants and layers will be described in multiple articles.

<hr/>

> **_NOTE:_**  I am releasing one part every once in a while, at which time links will be added to the bullet points below. The articles will also be announced on the Twitter account <a href="https://twitter.com/pagetable">@pagetable</a> and the Mastodon account <a href="https://mastodon.social/@pagetable">@pagetable&#64;mastodon.social</a>.

<hr/>

* **Part 0: Overview and Introduction**
*That's this part.*
* **[Part 1: IEEE-488](https://www.pagetable.com/?p=1023)** [PET/CBM Series; 1977]
This part covers layers 1 (electrical) and 2 (byte transfer) of IEEE-488, an 8-bit parallel bus with three handshake lines, an ATN line for bus arbitration and very relaxed timing requirements. 
* **[Part 2: The TALK/LISTEN Layer](https://www.pagetable.com/?p=1031)**
This part talks about layer 3 (TALK/LISTEN), which is shared between all bus variants.
* **[Part 3: The Commodore DOS Layer](https://www.pagetable.com/?p=1038)**
This part describes layer 4 (Commodore DOS), which is shared between all bus variants.
* **[Part 4: Standard Serial (IEC)](https://www.pagetable.com/?p=1135)** [VIC-20, C64; 1981]
The VIC-20 introduced a serial version of layers 1 and 2 with one clock and one data line for serial data transmission, and an ATN line for bus arbitration. It has some strict timing requirements. This bus is supported by all members of the home computer line: VIC-20, C64, Plus/4 Series, C128 and C65.
* **[Part 5: TCBM](https://www.pagetable.com/?p=1324)** [C16, C116, Plus/4; 1984]
The Plus/4 Series introduced a 1-to-1 bus between the computer and one drive, with 8 bit parallel data, two handshake lines, and two status lines from the drive to the computer. It was the short-lived planned successor of the Standard Serial bus, but was then replaced by Fast Serial.
* **Part 6: Fast Serial** [C128; 1985] *(coming soon)*
The C128 introduced Fast Serial, which replaces layer 2 byte transmission of Standard Serial by using a previously unused wire in the Serial connector as a third line for data transmission. Bus arbitration is unchanged. The controller detects a device's Fast Serial support and can fall back to the Standard Serial protocol.
* **[Part 7: JiffyDOS](https://www.pagetable.com/?p=1387)** [1986]
JiffyDOS, a 3rd party ROM patch for computers and drives, replaces layer 2 byte transmission of Standard Serial by using the clock and data lines in a more efficient way. Bus arbitration is unchanged. The controller detects a device's JiffyDOS support and can fall back to the Standard Serial protocol.
* **Part 8: CBDOS** [C65; 1991] *(coming soon)*
The unreleased C65 added CBDOS ("computer-based DOS") by integrating one or more drive controllers into the computer. There are no layers 1 and 2, and layer 3 sits directly on top of function calls that call into the DOS code running on the same CPU.

> This article series is an Open Source project. Corrections, clarifications and additions are **highly** appreciated. I will regularly update the articles from the repository at [https://github.com/mist64/cbmbus_doc](https://github.com/mist64/cbmbus_doc).

[^1]: The speeds have been measured by repeatedly reading the status channel of a disk drive. IEEE-488, Serial and JiffyDOS were measured on a 1 MHz C64 and Fast Serial on a C128, which executes all (Fast) Serial code in 1 MHz mode. TCBM was measured on a 1.77 MHz Plus/4 with the screen on, which makes the effective CPU speed similar to the C64. This is the code: a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 9d 00 04 e8 d0 f7 60. Both Fast Serial and JiffyDOS can reach higher speeds in the special case of loading files using custom protocols. Controller code size was measured on CBM2 for IEEE-488, on C64 for Serial and JiffyDOS, and on C128 for Fast Serial. Code sizes are approximate and do not include the LOAD and SAVE code.

<!---

f0ed-f0f4 (7)
f8ea-f911 (39)
fb97-fc9a (259)
----
+305




* size
	* controller!
	* IEEE: CBM2
	* Serial: C64
	* Fast: C128
	* TCBM: EC8B-ED18, EDD4-EDEA, EDFA-EE5D
		* = 99 + 22 + 141 = 262

>2000 a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff ca d0 fa 60
break 2000
break 2012
break 2015
g
sys8192
g

# inc $d020
>2000 a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff ee 20 d0 ca d0 f7 60

# jsr $ffd2
>2000 a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 20 d2 ff ca d0 f7 60

# sta $0400,x (C64)
>2000 a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 9d 00 04 e8 d0 f7 60

# sta $0400,x (C128; 16 bytes only)
>2000 a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 f0 20 a5 ff 9d 00 04 e8 d0 f7 60

# sta $0c00,x (Plus/4)
>2000 a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 9d 00 0c e8 d0 f7 60

# sta $1e00,x (VIC-20)
>1000 a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 9d 00 1e e8 d0 f7 60
break 1000
break 1026

# two loops (C64)
>2000 a9 00 20 bd ff a9 01 a2 08 a0 0f 20 ba ff 20 c0 ff a9 08 20 b4 ff a9 6f 20 96 ff a2 00 20 a5 ff 9d 00 04 e8 d0 f7 20 a5 ff 9d 00 04 e8 d0 f7 60
break 2027
break 2031

# IEEE cart
>2000 a9 08 20 d7 cb a9 6f 20 27 cc a2 00 20 b4 cc ca d0 fa 60

# IEEE cart; sta $0400,x

>2000 a9 08 20 d7 cb a9 6f 20 27 cc a2 00 20 b4 cc 9d 00 04 e8 d0 f7 60

* VIC-20 Serial
	* VIC-20/1540     2614 -> 0.38
* C64 Serial
	* VIC-20/1541     2941 -> 0.34
	* VIC-20/1581     2767 -> 0.36
	* C64/1541:       2616 -> 0.38
	* C64/1581:       2426 -> 0.41
	* C64/J1541:      2274 -> 0.44
	* Plus/4, 1541:   5163 -> 0.34
* JiffyDOS
	* C64J/1541:       472 -> 2.1
* Fast Serial
	* C128/1571:       487 -> 2.1
* TCBM
	* Plus/4, 1551:    746 -> 2.4(1.3)
* IEEE-488
	* IEEE-488 cart:   479 -> 2.1


12567 cycles
199 lines
$ec -> $7b
236 -> 123
235 -> 312 -> 123

# Misc

* interesting historical detail:
	* they started with a complex industry standard
	* didn't support all use features, but allowed users to do so
	* newer variants tried to pull over some of the unused features as well
		* Serial supports one-to-many
		* Serial supports device-to-device
	* they were only slowly removed
		* Fast Serial breaks SRQ
		* Fast Serial breaks device-to-device for different protocol versions
		* TCBM amnd CBDOS break device-to-device and one-to-many

-->
