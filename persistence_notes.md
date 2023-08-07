# Persistence


## I/O Devices
- Memory bus
- General I/O bus (e.g. PCI)
- Peripheral I/O Bus (e.g. SCSI, SATA, USB)
- Interface - allows the system software to control its operation
- Internal structure - implementation specific and responsible for implementing the abstraction the device presents to the system


#### The Canonical Protocol
- status register
- command register
- data register
1. OS waits until the device is ready to receive a command by repeatedly reading the status register; polling the device
2. OS sends some data down to data register
3. OS writes a command to command register; lets device know that both the data is present and that it should begin working on the command

- Crux: how to avoid the costs of polling
- Lowering CPU overhead with interrupts
- OS can issue a request, put the calling process to sleep, and context switch to another task
- when device is finally finished with operation, it will raise a hardware interrupt, causing the CPU to jump into the OS at a predetermined interrupt service routine (ISR) or interrupt handler
- handler is a piece of OS code that will finish the request
- Interrupts allow for overlap of computation and I/O
- switching is expensive so if a device is fast, it may be best to poll
- if speed of device is not known or varies, it may be best to use a hybrid approach that polls for a little while, then uses interrupts


#### Direct Memory Access
- DMA engine is a very specific device within a systme that can orchestrate transfers between devices and main memory wihtout much CPU intervention
- OS would program DMA engine by telling it where the data lives in memory, how much dat to copy and which device to send it to
- when DMA is complete, the controller raises an interrupt, and the OS knows the transfer is complete
