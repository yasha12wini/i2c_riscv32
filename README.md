# i2c_riscv32


---

````markdown
# I²C Communication in RISC-V Linux VM (with i2c-stub)

This project demonstrates how to simulate and interact with an I²C device (at address `0x50`) in a **RISC-V 32-bit Linux VM on Windows**, using the `i2c-stub` kernel module and `i2c-tools`.

>  No real hardware is required. This uses a virtual I²C device emulated in software.

---

##  Prerequisites

- A working **RISC-V Linux VM** (running on Windows)
- `i2c-tools` installed
- `gcc` compiler available
- Root access (`sudo`) in your VM

---

##  Steps

### 1. Boot into your RISC-V Linux VM

Make sure you are logged in to the Linux shell.

Check your architecture:

```bash
uname -m
````

Expected output:

```
riscv32
```

---

### 2. Install I²C Tools (if not already installed)

```bash
sudo apt update
sudo apt install i2c-tools
```

Check the version:

```bash
i2cdetect -V
```

---

### 3. Check I²C Buses

```bash
ls /dev/i2c-*
```

> If **no output**, proceed to step 4 to create a virtual device.

---

### 4. Simulate a Virtual I²C Device

Load the `i2c-stub` kernel module with a fake device at address `0x50`:

```bash
sudo modprobe i2c-stub chip_addr=0x50
```

Verify the I²C bus was created:

```bash
ls /sys/bus/i2c/devices/
```

You should see something like `i2c-0`, `i2c-1`, etc.

---

### 5. Scan the I²C Bus

```bash
sudo i2cdetect -y 0
```

Expected output (partial):

```
     0 1 2 3 4 5 6 7 8 9 a b c d e f
...
50: 50 -- -- -- -- -- -- -- 
...
```

This confirms that a device at address `0x50` is visible on `/dev/i2c-0`.

---

### 6. Limitations of `i2c-stub` for Read/Write

* `i2cget` and `i2cset` **require a real driver behind the emulated device** to properly store and respond to read/write operations.
* With only `i2c-stub` loaded, **writes will not persist**, and reads will return **static or dummy values (usually 0x00)**.

```bash
# Will appear to succeed but does nothing real
sudo i2cset -y 0 0x50 0x00 0xAB

# Will likely return 0x00 or dummy data
sudo i2cget -y 0 0x50 0x00
```

These commands demonstrate communication with the I²C bus, **but not actual memory behavior** without a full device emulation driver.

---

### 7. I²C C Program (Optional)

#### Create `i2c_read.c`

```c
#include <stdio.h>
#include <fcntl.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <unistd.h>

int main() {
    int file;
    char *bus = "/dev/i2c-0"; // Adjust if your bus is different

    if ((file = open(bus, O_RDWR)) < 0) {
        perror("Failed to open the bus.\n");
        return 1;
    }

    ioctl(file, I2C_SLAVE, 0x50);

    // Write 0xAB to register 0x00
    char writeBuffer[2] = {0x00, 0xAB};
    write(file, writeBuffer, 2);

    // Set register pointer to 0x00
    char reg[1] = {0x00};
    write(file, reg, 1);

    // Read from register 0x00
    char data[1];
    read(file, data, 1);

    printf("Data read: 0x%X\n", data[0]);

    close(file);
    return 0;
}
```

#### Compile and Run

```bash
gcc i2c_read.c -o i2c_read
sudo ./i2c_read
```

Again, due to `i2c-stub` limitations, the output will likely be:

```
Data read: 0x0
```
---


```

---


```
