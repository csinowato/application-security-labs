# Infinite Loop

## Summary
This section covers an infinite loop vulnerability in `giftcardreader.c`, a custom C application. The application parses `.gft` files, which contain gift card data stored in binary format. A separate tool (`giftcardwriter.c`) was used to craft custom `.gft` files for testing.

To exploit this vulnerability, I modified the gift card generator to create a `.gft` file that followed the expected format but caused the application to repeatedly decrement the iterator, resulting in an infinite loop.

**Note:** The original source files (`giftcardreader.c` and `giftcardwriter.c`) were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerability exists in the `animate()` function, which is the interpreter that executes a custom instruction set. Within the function, the handler for opcode `0x09` does not check that `arg1` is positive, so a negative value causes the iterator to keep getting decremented, leaving it stuck in an infinite loop.

## Vulnerability Details
The code snippet below shows the vulnerable handler for opcode `0x09` which updates the value of `pc` without checking that `arg1` is positive.

For clarity, only the relevant case is shown in the switch statement below, other cases have been omitted.
```
switch (*pc) {
    ...
    case 0x09:
        pc += (char)arg1;
        break;
}
```
At the end of the `animate()` function is the following loop termination condition:
```
if (pc > program+256) break;
```
Because a negative value causes `pc` to keep getting decremented, this condition is never satisfied, causing the application to hang in an infinite loop.

## Steps to Reproduce
In `giftcardwriter.c`, I added the following "program" to keep decrementing the iterator. The instruction stream includes the opcode `0x09`, followed by `0xee`, which is a hexadecimal value that becomes -18 when cast to a signed `char`. This program was then written to the `.gft` file.
```
unsigned char program[256] = {
    0x09, 0xee, 0x02,
};
fwrite(program, 256, 1, fd1);
```
The crafted gift card file was then loaded into the `giftcardreader` application. Because there was no check for `arg1` being positive, the negative value of `0xee` caused the pointer to move backwards repeatedly. This prevented the loop termination condition from being met, which caused the application to hang indefinitely.

## Impact
Exploiting this vulnerability caused the application to enter an infinite loop, rendering it unresponsive. In a real world context, this could be further exploited to perform a denial-of-service (DoS) attack.

## Mitigation
To mitigate the issue, I added a check to make sure that `arg1` only gets added to `pc` if it is a positive value. This prevents the iterator from continuously getting decremented if `arg1` is negative, thus preventing an infinite loop.
```
switch (*pc) {
    ...
    case 0x09:
        if ((char)arg1 > 0) {
            pc += (char)arg1;
        }
        break;
}
```

## Tools and Artifacts Used
- `giftcardwriter.c` (used to generate crafted `.gft`files)
- `giftcardreader.c` (target application used for testing)
- VS Code debugger (to trace execution and observe the crash)
- Manual source code review
