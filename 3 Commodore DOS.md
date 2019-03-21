# Commodore Peripheral Bus: Part 3: Commodore DOS

* disk drives
	* channel = 0 is reserved for a reading a PRG file.
	* channel = 1 is reserved for a writing a PRG file.
	* channel = 2-14 need the filetype and the read/write flag in the filename as ",P,W" for example.
	* channel = 15 for DOS commands or device status info.

* Printers
	* printers use the secondary address to pre-select a character set

0 graphic
7 business

	ftp://www.zimmers.net/pub/cbm/manuals/printers/MPS-801_Printer_Users_Manual.pdf
Sa= 0: Print data exactly as received
Sa= 6: Setting spacing between lines
Sa= 7: Select business mode
Sa= S: Select graphic mode
Sa=10: Reset the printer

	https://www.mocagh.org/forsale/mps1000-manual.pdf
0 Print data exactly as received in Uppercase/Graphics mode
1 Print data according to a previously-defined format
2 Store the formatting data
3 Set the number of lines per page to be printed
4 Enable the printer format diagnostic messages
5 Define a programmable character
6 Set spacing between lines
7 Print data excactly as received in Upper/lowercase
9 Suppress diagnostic message printing
10 Reset printer

* practice:
	* listen 4, secondary 7, "Hello", unlisten
	* TODO: bus turnaround
