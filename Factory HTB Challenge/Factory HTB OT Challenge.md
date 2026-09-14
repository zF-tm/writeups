# Analysis

Lets look at the connection first via nmap:

![Alt text](Pasted image 20260915002931.png)

the challenge gives us 2 files, an image and a pdf, 
![Alt text](Pasted image 20260915001114.png)

lets look at this picture. our task is to close the in_valve and open the out_valve


![Alt text](Pasted image 20260915011908.png)

this picture is explaining how auto_mode opens on start, but manual_mode is open only when auto_mode is closed

![Alt text](Pasted image 20260915001945.png)
in this photo, were gonna ignore the automatic mode which is at the top, since we're gonna be controlling everything manually. 

so lets look at the bottom 

in order out_value to be 1
force_start_out has to be 1
and manual mode has to be 1 
and stop_out has to be 0 

this is how we turn on out_valve


now for in_valve, we need to turn it off:
![Alt text](Pasted image 20260915003802.png)


to turn in_valve off, we need the following values:

force_start_in = 0
manual_mode = 1 
stop_in = 1 


to get those values we need to look up 
![Alt text](Pasted image 20260915003953.png)
to turn stop_in on we need these values:

cutoff_in = 1  
manual_mode = 1


# Turning out_valve on plan
lets look at our original values and simulate what we are going to do before running any command

![Alt text](Pasted image 20260915002931.png)

- auto_mode = 1
- manual_mode = 0
- stop_out = 0
- stop_in = 0
- low_sensor = 0
- high_sensor = 0
- in_valve = 1
- out_valve = 0

looking back on how to turn out_valve, we need 
force_start_out has to be 1
and manual mode has to be 1 
and stop_out has to be 0 

so we are going to turn manual_mode to true
making the new values:
- auto_mode = 0
- manual_mode = 1
- stop_out = 0
- stop_in = 0
- low_sensor = 0
- high_sensor = 0
- in_valve = 1
- out_valve = 0

stop_out is already 0
so now we need to turn on force_start_out = 1 

#### Steps:
1- turn on manual mode
2- turn on force_start_out


# Turning in_valve off plan

after performing the last steps, the values are:
- auto_mode = 0
- manual_mode = 1
- stop_out = 0
- stop_in = 0
- low_sensor = 0
- high_sensor = 0
- in_valve = 1
- out_valve = 1

lets look back at how we are gonna turn in_valve off:
force_start_in = 0
manual_mode = 1 
stop_in = 1 

manual mode is already on
and by default force_start_in and out are 0 on start

so we need to turn stop_in to true

we do that through:

cutoff_in = 1  
manual_mode = 1

#### final step:
turn cutoff_in to true.

the final values:
 - auto_mode = 0
- manual_mode = 1
- stop_out = 0
- stop_in = 0
- low_sensor = 0
- high_sensor = 0
- in_valve = 0
- out_valve = 1


# Actually Sending the commands

In this challenge, we need to send a modbus packet structured command

| FC      | Command                       | Request structure                   |
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

here we need write single coil 

AA represents the slave ID in hex
CCCC represents the PLC address in hex
and DDDD represents the value in hex aswell 

, in our challenge we had the photo:

![Alt text](Pasted image 20260915010336.png)

Slave ID is 82 in decimal which is 52
Second Part is 05 for writing a single coil 
PLC address to hex (we're gonna change that later)
last part: 
FF00 means true/open
0000 means false/close

lets limit the structure for this specific challenge
5205CCCCDDDD

#### Turning Manual Mode on

5205CCCCDDDD

CCCC: Turn 9947 to hex: 26DB
DDDD: FF00

Command:

520526DBFF00
![Alt text](Pasted image 20260915011051.png)


#### Turning on out_valve
the step:
turn on force_start_out

5205CCCCDDDD
Address: 52 in hex: 0034
True: FF00

52050034FF00
![Alt text](Pasted image 20260915011340.png)



#### Turning off in_valve

turn cutoff_in to true.
5205CCCCDDDD
address cutoff_in in hex: 001A
on: FF00

final command:
5205001AFF00


![Alt text](Pasted image 20260915011541.png)

we get the flag
