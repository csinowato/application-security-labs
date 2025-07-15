# Out-of-Bounds Register

## Summary
This section covers an out-of-bounds register vulnerability in `giftcardreader.c`, a custom C application. The application parses `.gft` files, which contain gift card data stored in binary format. A separate tool (`giftcardwriter.c`) was used to craft custom `.gft` files for testing.

To exploit this vulnerability, I modified the gift card generator to create a `.gft` file that followed the expected format but caused the application to access an invalid register index, triggering a segmentation fault.

**Note:** The original source files were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerability exists in the `animate()` function, which is the interpreter that executes a custom instruction set. Within the function, the handler for opcode `0x01` does not validate the register index and writes directly to `regs[arg1]`.

## Vulnerability Details
Here, the register array is declared with a fixed size of 16, which means it can be indexed from 0 to 15.
```
unsigned char regs[16];
```
The code snippet below shows the vulnerable handler for opcode `0x01`, which writes to a register index without validating that `arg1` falls within range. Since `regs` has a fixed-size of 16, if the value of `arg1` exceeds the valid range, it results in an out-of-bounds write.

For clarity, only the relevant case is shown in the switch statement below, other cases have been omitted.
```
switch (*pc) {
    case 0x01:
        regs[arg1] = *mptr;
        break;
}
```

## Steps to Reproduce
In `giftcardwriter.c`, I added the following "program" to store the current character in an invalid index. The instruction stream includes the opcode `0x01`, followed by an invalid register index `0x87` which exceeds the valid range of 0-15. The program was then written to the `.gft` file.
```
unsigned char program[256] = {
    0x01, 0x87, 0x03,
};
fwrite(program, 256, 1, fd1);
```
The crafted gift card file was then loaded into the gift card reader application. Because there was no bounds check, the out-of-bounds register value led to an invalid memory write.

## Impact
Exploiting this vulnerability caused the application to crash with a segmentation fault. Because the interpreter didn't validate the register index, it led to an out-of-bounds write, which could be potentially be exploited further in a real-world context.

## Mitigation
To mitigate the issue, I added a bounds check to make sure that the register index (`arg1`) is within the range 0 to 15. If it is not in the valid range, then the instruction is skipped. This prevents out-of-bounds access to the `regs` array.
```
switch (*pc) {
    case 0x01:
        if ((arg1) < 0 || (arg1) > 15) break;
        regs[arg1] = *mptr;
        break;
}
```

## Tools and Artifacts Used
- `giftcardwriter.c` (used to generate crafted `.gft` files)
- `giftcardreader.c` (target application used for testing)
- VS Code debugger (to trace execution and observe the crash)
- Manual source code review
