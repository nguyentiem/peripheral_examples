eusart_spi_main_dma

This project demonstrates DMA-driven operation of the EUSART in
synchronous main (formerly master) mode.  EUSART1 is configured for
SPI-compatible operation at 1 Mbps.  Compatibility with the Synchronous
Peripheral Interface standard implies a word size of 8 data bits
transmitted and received MSB-first.

In systems where the secondary (formerly slave) device is a processor,
some delay is generally required to detect the chip select falling edge
and to prepare to receive the incoming data.  It is incumbent on the main
processor to insert this delay through some means, e.g. via a software
delay or otheriwse (alternatively, a pin-based handshaking mechanism can
be used).  Because this example must start the receive and transmit LDMA
channels, the pair of calls to LDMA_StartTransfer() do this, effectively
inserting about 42 microseconds of chip select setup time at 19 MHz.

Buffers inbuf[] and outbuf[] are, respectively, zeroed out and populated
with 10 bytes of data (which can be changed by modifying BUFLEN).

Previously, the LDMA has been configured so that (a) one channel moves
each byte in outbuf[] to the EUSART1_TXDATA register when there is space
in the transmit FIFO (EUSART1_STATUS_TXFL is asserted), and (b) another
channel moves the matching received byte from EUSART1_RXDATA to inbuf[]
(upon assertion of EUSART1_STATUS_RXFL).  Chip select is asserted, the
transfers are started for the LDMA channels, and the device enters the
EM1 low-energy mode.

While the CPU is halted, the LDMA writes each byte from outbuf[] to
the transmit FIFO as space is available.  EUSART1 drives the CLK pin
so that each pulse causes the byte written to TXDATA to be shifted out
while an incoming byte is shifted into RXDATA one bit at a time.  Once
all 8 bits of the incoming byte have been received, the LDMA writes each
byte from the receive FIFO to inbuf[].

This process occurs autonomously, and only the LDMA interrupt request
(assertion of one or more of the LDMA_IF_DONEx flags or LDMA_IF_ERROR in
the event of some kind of problem) causes the device to exit EM1.  The
LDMA_IRQHandler() clears each channel done interrupt flag and sets the
rx_done state variable to indicate transfer completion when all bytes
have been received.  The chip select is de-asserted once the last byte is
received.  Set a breakpoint in the main loop where the rx_done flag is
cleared in order to inspect the contents of inbuf[] before and after
each set of transfers.

Note: This example uses inclusive lexicon wherever possible. For more
information, visit https://www.silabs.com/about-us/inclusive-lexicon-project

================================================================================

Peripherals Used:

GPIO
LDMA
EUSART1

The CMU is used indirectly via the EUSART_SpiInit() function to calculate
the divisor necessary to derive the desired bit rate.

================================================================================

Test Procedure:

1. This example requires two Starter Kits (STKs) for any two Series 2 EFM32
   or EFR32 devices.  Build the spi_secondary_dma example and download it to
   the first board.

2. Build this project and download it to the second board.

3. Connect the Expansion Header pins from the main to the secondary board
   as follows:

      Main WSTK         Secondary WSTK
    -------------       --------------
    Pin 1  (GND)  <---> Pin 1  (GND)
    Pin 4  (MOSI) <---> Pin 4  (MOSI)
    Pin 6  (MISO) <---> Pin 6  (MISO)
    Pin 8  (SCLK) <---> Pin 8  (SCLK)
    Pin 10 (CSn)  <---> Pin 10 (CSn)

   NOTE: Some radio boards do not follow this convention for SPI
   pin-mapping.  See the supported board list below for specific
   assignments.

4. Before running the example, set a breakpoint on the rx_done = false
   statement in the main loop.  Examine the inbuf[] array to see the
   received data from the secondary.  This example transfers data
   continuously but will not start until PB0 is pressed.

================================================================================

Listed below are the port and pin mappings for working with this example.

Board:  Silicon Labs EFR32ZG23 868-915 MHz 14 dBm Radio Board (BRD4204D)
        + Wireless Starter Kit Mainboard (BRD4001A)
Device: EFR32ZG23B010F512IM48 (this code will run unchanged on any
        EFR32ZG23 variant so long as the pins used as present).
PC1 - EUSART1_TX (MOSI)  - Expansion Header pin 4
PC2 - EUSART1_RX (MISO)  - Expansion Header pin 6
PC3 - EUSART1_CLK (SCLK) - Expansion Header pin 8
PC0 - EUSART1_CS (CSn)   - Expansion Header pin 10

Board:  Silicon Labs EFR32xG24 2.4 GHz 10 dBm Radio Board (BRD4186C)
        + Wireless Starter Kit Mainboard (BRD4001A)
Device: EFR32MG24B210F1536IM48 (this code will run unchanged on any
        EFR32MG24 variant so long as the pins used as present).
PC1 - EUSART1_TX (MOSI)  - Expansion Header pin 4
PC2 - EUSART1_RX (MISO)  - Expansion Header pin 6
PC3 - EUSART1_CLK (SCLK) - Expansion Header pin 8
PC0 - EUSART1_CS (CSn)   - Expansion Header pin 10

Board:  Silicon Labs EFR32FG25 902-928 MHz 14 dBm Radio Board (BRD4270B)
        + Wireless Starter Kit Mainboard (BRD4001A)
Device: EFR32FG25B222F1920IM56 (this code will run unchanged on any
        EFR32FG25 variant so long as the pins used as present).
PC0 - EUSART1_TX (MOSI)  - Expansion Header pin 4
PC1 - EUSART1_RX (MISO)  - Expansion Header pin 6
PC2 - EUSART1_CLK (SCLK) - Expansion Header pin 8
PB4 - EUSART1_CS (CSn)   - Expansion Header pin 10