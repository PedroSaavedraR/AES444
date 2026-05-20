# Small Scale AES Hardware Implementation

## Overview
This project implements a small scale variant of the Advanced Encryption Standard (AES). Specifically, it is an implementation of the $SR^{*}(10,4,4,4)$ variant, which consists of 10 rounds, a 4x4 state matrix, and 4-bit words/SBoxes. The design is a substitution permutation network (SPN) block cipher written in Verilog and targeted for a Spartan 7 FPGA using Xilinx Vivado. 

This project was developed as part of the Hardware-Oriented Security course at the University of Stuttgart.

---

## Project Requirements
* **Development Environment:** Xilinx Vivado
* **Hardware:** Arty S7-50 development board
* **Target FPGA:** Spartan 7 (xc7s50csga324-2)
* **Serial Communication:** Putty (or similar terminal emulator)

---

## Project Structure and Modules
The small scale AES modifies a state matrix through several operations. The implementation is broken down into the following Verilog modules:

### Core Operations
* **SubBytes:** Substitutes all words in the state matrix using a 4-bit Substitution Box (SBox) implemented as a lookup table.
* **MixColumns:** Multiplies each column by a Maximum Distance Separable (MDS) matrix over $GF(2^{4})$. The multiplication uses the polynomial $X^{4}+X+1$.
* **ShiftRows:** Performs a left rotation of the words in the state matrix according to their row index (0 to 3).
* **AddRoundKey:** Executes a simple XOR operation between the state matrix and the generated round key.

### Key Generation & Round Logic
* **Key Schedule:** Generates 10 round-dependent keys. It computes round constants (`rcon`) over $GF(2^{4})$ and uses the SubBytes module to substitute the last column of the previous key.
* **Round Module:** Integrates SubBytes, ShiftRows, MixColumns, and AddRoundKey into a single round structure. The MixColumns operation is omitted in the final (10th) round.
* **Top Level Entity:** The main AES module can be implemented as either a one clock cycle (unrolled) design or a "one round per cycle" design utilizing a state machine.

---

## Testing and Simulation
Testbenches must be created for each individual module to verify correct output via Behavioral Simulation in Vivado. 

**Key Test Vectors:**
* **MixColumns:** Input `A7E9` results in Output `9CA5`.
* **Round 5 Verification:** Input `1234 5678 9ABC DEF0` with Key `EDCC 1233 1233 7455` results in Output `8915 6871 5AE2 313F`.
* **Full Encryption:** Plaintext `0000 0000 0000 0000` with Master Key `FEDC BA98 7654 3210` results in Ciphertext `609B 6227 BA39 3803`.

---

## Hardware Deployment
The FPGA implementation requires integrating the Verilog code with an additional VHDL controller file (`AES_Controller.vhd`) provided by the course. To program the FPGA:

1. Instantiate your top-level Verilog AES module as a component inside the provided `AES_Controller.vhd` file.
2. Run Synthesis, Implementation, and Generate the Bitstream in the Vivado project manager panel.
3. Open the Hardware Manager, select Auto-Connect to detect the board, and program the `xc7s50` device.

### FPGA Interaction
Use Putty configured for serial communication (9600 baud rate) to interface with the board. The board is controlled via its physical buttons:
* **Button 0:** Sets the plaintext to all zeroes.
* **Button 1:** Sets the plaintext to a random value.
* **Button 2:** Starts the encryption process.
* **Button 3:** Prints the resulting ciphertext to the Putty terminal.
