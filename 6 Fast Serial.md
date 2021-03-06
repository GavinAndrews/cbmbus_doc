# Commodore Peripheral Bus: Part 7: Fast Serial

* "synchronous serial", "burst"
* "Commodore Fast Serial Interface Protocol"
* history
	* Commodore 1985
	* supported by C128, C65, 1570, 1571, 1581
	* supported by classic 3rd party devices, like CMD hard disks
* requires hardware device for CLK/DATA, like CIA 6526
* requires a fourth wire, reuses "SRQ" wire
	* clock is used to ack every byte (cbdos.spec), probably to allow for slow receivers like C64 with badlines

* new byte send protocol
	* uses SRQ as clock, DATA for data
		* names are wrong now, should we use saner names?
		* SRQ -> CLK
		* CLK -> ACK
	* MSB first
	* 10 µs per bit
		* 6526 docs says min clock is Phi0/4, so 8 µs per bit
	* TODO same byte handshaking using DATA

* timing properties
	* talker and listener need a TODO ~80 µs window 
	* makes quite some assumptions
	* but they are true on the C64, which is pretty much the worst case

* backwards compatibility
	* idea
		* controller must tell device that it supports *and* requests fast serial
		* when device receives, it must request fast serial
		* when device sends, it can send fast serial if receiver supports it
	* controller
	    * before TALK/LISTEN
		* sends HRF, waits 100 µs
			* TODO why
		* every fast device knows the controller supports fast serial
	    * listener
		* check bit 3 ICR (for a fast byte) and the CLK line (for the start of the slow protocol)
		* if a fast byte arrived, set the fast serial flags
	    * talker
	    	* if DRF arrives before sending data, set the fast serial flags
		* if fast serial flag set, send fast byte
	    * UNTALK/UNLISTEN/error
		* clear fast serial bits
	* device (if it knows the controller supports fast serial)
	    * listener
		* before receiving the first byte, before releasing DATA
		* send DRF
	    * talker
		* send fast byte

* TODO can two devices talk fast serial?
	* does device-to-device talking work at all any more?
	* any LISTEN/TALK from the controller will signal devices should speak fast serial
	* slow/slow should still work
	* fast-capable/slow probably breaks
	* does fast-fast work?
	* is it possible to tell a fast serial device that it shouldn't do fast serial any more to set it up for a session with a slow device? yes, UNTALK/UNLISTEN.

	* in theory, it's a two-way communication
		* for every TALK/LISTEN, controller requests fast serial
		* request gets canceled on UNTALK/UNLISTEN
		* so it's a request for a single session
		* but device does not immediately answer whether it supports it
			* when device listens, it doesn't ack until talker starts sending first byte (can this be canceled?)
			* when device talks, it doesn't ack until it has sent the first byte (which is in fast serial protocol!)
	* in practice, 1571/1581 turn on fast serial as soon as they see HRF, no matter whether the following command is for them (there doesn't even have to be a command, HRF can be sent at any time)
		* so the HRF is valid for the session as a whole, for both the talker and the listener
		* i.e. HRF TALK8 SEC0 LISTEN9 SEC0 -> 8 and 9 speak fast serial
		* but the only way to detect whether a device supports fast serial is to transfer data, e.g. by reading the error channel
		* -> that's communication on a higher layer :(

* bust mode
	* "When fast serial communications are available, files are loaded by sectors (254-byte chunks of data) using a special feature of the 1571 drive known as burst mode. However, fast mode SAVEs are still done byte by byte." (Mapping the 128)
	* TODO

* Notes
	* Fast: setzen devices beim Unlisten das fast Flag zurück?
	* Fast macht SRQ kaputt :(
		* PET kann SRQ in Hardware, aber Software hat keinen Support
		* SRQ als Feature hat parallel nach seriell überlebt, VIC-20 und C64 können SRQ 
		* wenn auch 1541 kein SRQ in Hardware kann
		* beim 264 ist SRQ Hardware Support weggefallen
		* C128 benutzt SRQ-Leitung für Fast
		* -> ansich war SRQ noch mit erweiterter Software möglich, wenn auch ungenutzt aber C128 macht das kaputt

	* cbdos.spec:
		Standard serial: Actual transfer rate = 4800-6800 baud.
		    Fast serial: Actual transfer rate = 21000-23000 baud.
		   Burst serial: Actual transfer rate = 78400-80000 baud.

* Burst
	* TODO


Part 6 of the series of articles on the Commodore Peripheral Bus family will cover the JiffyDOS protocol on layer 2, which shipped as a third party ROM patch for computers and drives, replacing the byte transmission protocol of Standard Serial by using the clock and data lines in a more efficient way.


http://the-cbm-files.tripod.com/diskdrive/h1571.htm
