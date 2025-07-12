# Signed Integer Overflow

## Summary
This section covers a signed integer overflow vulnerability in `giftcardreader.c`, a custom C application. The application parses `.gft` files, which contain gift card data stored in binary format. A separate tool (`giftcardwriter.c`) was used to craft custom `.gft` files for testing.

To exploit this vulnerability, I modified the gift card generator to create a `.gft` file that followed the expected format but triggered a flaw in the gift card reader's size handling logic, leading to memory corruption and a crash.

**Note:** The original source files (`giftcardreader.c` and `giftcardwriter.c`) were provided as part of the course and are not included in this repository.

## Affected Component
The vulnerability exists in `giftcardreader.c`, where the program reads a 32-bit signed integer from the `.gft` file and uses it to allocate memory without checking if the value is valid.

## Vulnerability Details
In C, a signed 32-bit integer can only hold values between -2,147,483,648 and 2,147,483,647. Although this exploit used a large positive out-of-range value, even negative numbers within the range can lead to incorrect memory allocation. This happens because when a signed integer is passed to `malloc()`, it gets implicitly cast to an unsigned value (e.g. -1 becomes 4,294,967,295). So only values between 0 and 2,147,483,647 can be safely passed.

As shown in the code snippet below, the application reads an 4-byte value from the `.gft` file and uses it directly in a memory operation:
```
fread(&ret_val->num_bytes, 4,1, input_fd);
ptr = malloc(ret_val->num_bytes);
```
Here, because `num_bytes` is not validated, an out-of-range value can cause `malloc()` to allocate the wrong amount of memory, leading to a buffer overflow.

## Steps to Reproduce
In `giftcardwriter.c`, I modified the code to set the value of the `num_bytes` field in the gift card structure to 2,147,483,648.
```
examplegc.num_bytes = 2147483648;
```
This value exceeds the maximum value for a signed 32-bit integer. It was then written to the `.gft` file using:
```
fwrite(&examplegc.num_bytes, 4, 1, fd1);
```
The crafted gift card file was then loaded into the `giftcardreader` application. Because `num_bytes` was not validated, the out-of-range value led to incorrect memory allocation and a segmentation fault.

## Impact
Exploiting this vulnerability caused the application to crash with a segmentation fault. Since the input wasn't validated, it led to memory corruption, which could be potentially be exploited further in a real-world context.

## Mitigation
To mitigate the issue, I added a validation check before allocating memory:
```
fread(&ret_val->num_bytes, 4,1, input_fd);
if (ret_val->num_bytes > 2147483647 || ret_val->num_bytes < 0) exit(0);
ptr = malloc(ret_val->num_bytes);
```
This check ensures that `num_bytes` has a value between 0 and 2,147,483,647 before calling `malloc()`.

## Tools and Artifacts Used
- `giftcardwriter.c` (used to generate crafted `.gft`files)
- `giftcardreader.c` (target application used for testing)
- VS Code debugger (to trace execution and observe the crash)
- Manual source code review
