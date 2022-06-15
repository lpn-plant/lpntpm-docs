# TPM communication over SPI

TPM may operate as SPI save which has been described in
[PC Client Platform TPM profile](https://www.trustedcomputinggroup.org/wp-content/uploads/PCClientPlatform-TPM-Profile-for-TPM-2-0-v1-03-20-161114_public-review.pdf)
section 6.4. This document summarises the most important aspects of SPI.

- TPM interrupts
  - TPM must implement PIRQ# pin. See section 6.4.3.
- Flow control
  - TPM may insert wait state to delay data transfer (from Host to TPM).
  - This is a non-standard behaviour not defined by SPI specification.
  - TPM **must** insert wait state when it's internal FIFO if full or accept the
    transaction.
    > The TPM, if it doesn’t insert a wait state at the designated point must
    > accept the transaction, as long as it doesn’t cross a register boundary.
    > If the transaction crosses a register boundary, the TPM may choose to
    > accept all of the data and discard the data that exceeds the size limit
    > for that register as long as doing so does not cause a change to the state
    > of any adjacent register.
  - Usually TPM is allowed to insert as many wait states (each wait state is 8
    clock cycles), except during read of a few registers where it is allowed to
    insert only one wait cycle. TPM cannot insert more than 1 wait cycle for the
    following registers: TPM_ACCESS_x TPM_STS_x TPM_INTF_CAPABILITY TPM_INT_ENABLE
    TPM_INT_STATUS TPM_INT_VECTOR TPM_DID TPM_VID.
  - TPM **must** insert wait state if Host attempts to write data without
    checking TPM status register to see if TPM is ready.
    > For writes, the TPM is required to insert wait states if software
    > attempts to write data without waiting for the TPM to transition to the
    > Ready state.
- Data transmission (see section 6.4.6 for details)
  - TPM uses SPI mode 0 (CPHA=0, CPOL=0), no other modes are supported.
  - SPI packet is outlined in Table 46
  - Data transfer starts from the most significant bit (MSB) and ends on the
    least significant bit (LSB)
  ![](images/tpm2_spi_protocol.png)
  - The first bit transferred determines whether transaction is read or write.
  - This is followed by 6 bit length from 1 byte to 64 bytes (TBD: is it length
    of entire packet or the data payload?)
  - Length is followed by 24-bit register address.
  - TPM commands are sent writing to a specific register, the command itself is
    passed in data payload, this is what should be passed to the
    [ExecuteCommand](https://github.com/microsoft/ms-tpm-20-ref/blob/99503012bca923f540b8b307aad43c942223b15c/TPMCmd/tpm/src/main/ExecCommand.c#L87)
    function in Microsoft reference TPM implementation.
