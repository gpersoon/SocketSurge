### Description

The functions `setSourceGasPrice()` and `setRelativeGasPrice()` can be updated by all transmitters.

If different transmitters would have a different view on the world, they might set slightly different values.
Also they could compete on setting values.
If transmitters would be premissionless they might accidentally or purposely set wrong values, which would mess with the fees 
and possibly result in griefing.

### Steps to reproduce

### Expected behavior
Perhaps have a seperate role for gas price updates.
Or maybe have a selection which transmitter can update the gas prices.

### Actual behavior

Values from the next transmitter overwrite the values from the previous transmitter.

### Screenshots


### Additional information

