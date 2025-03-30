### Introduction

The Windows API provides developers with a way for their applications to interact with the Windows operating system. For example, if the application needs to display something on the screen, modify a file or query the registry all of these actions can be done via the Windows API. The Windows API is very well documented by Microsoft and can be viewed [here](https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-api-list).

Windows API has specific data types that are essentially aliases for standard data types. They are used to improve readability, maintain consistency, and ensure compatibility across platforms. Here's a quick explanation of common Windows data types with examples:

---

### **1. `BOOL`**

- Represents a boolean value (`TRUE` or `FALSE`).
- **Size:** 4 bytes.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    BOOL isRunning = TRUE;

    if (isRunning) {
        printf("The program is running!\n");
    } else {
        printf("The program is not running!\n");
    }

    return 0;
}
```

**Output:**

```
The program is running!
```

---

### **2. `HANDLE`**

- Represents a handle to an object (e.g., a file, process, or thread).

```c
#include <windows.h>
#include <stdio.h>

int main() {
    HANDLE hFile = CreateFile(
        "example.txt", GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL
    );

    if (hFile == INVALID_HANDLE_VALUE) {
        printf("Failed to create file!\n");
    } else {
        printf("File handle created successfully.\n");
        CloseHandle(hFile);
    }

    return 0;
}
```

**Output:**

```
File handle created successfully.
```

---

### **3. `DWORD`**

- Unsigned 32-bit integer.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    DWORD value = 12345;
    printf("DWORD value: %lu\n", value);
    return 0;
}
```

**Output:**

```
DWORD value: 12345
```

---

### **4. `LPSTR` / `LPCSTR`**

- `LPSTR`: Pointer to a null-terminated string (modifiable).
- `LPCSTR`: Pointer to a constant null-terminated string.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    LPCSTR message = "Hello, Windows API!";
    printf("%s\n", message);
    return 0;
}
```

**Output:**

```
Hello, Windows API!
```

---

### **5. `LPVOID`**

- A pointer to any type of data.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    int data = 42;
    LPVOID ptr = &data;

    printf("Value stored in LPVOID pointer: %d\n", *(int*)ptr);
    return 0;
}
```

**Output:**

```
Value stored in LPVOID pointer: 42
```

---

### **6. `CHAR` / `WCHAR`**

- `CHAR`: 1-byte character.
- `WCHAR`: 2-byte wide character (Unicode).

```c
#include <windows.h>
#include <stdio.h>

int main() {
    CHAR charValue = 'A';
    WCHAR wcharValue = L'B';

    printf("CHAR value: %c\n", charValue);
    wprintf(L"WCHAR value: %lc\n", wcharValue);
    return 0;
}
```

**Output:**

```
CHAR value: A
WCHAR value: B
```

---

### **7. `ULONG_PTR`**

- An unsigned long integer used for pointers and handles in 64-bit or 32-bit platforms.

```c
#include <windows.h>
#include <stdio.h>

int main() {
    ULONG_PTR pointerValue = (ULONG_PTR)0xDEADBEEF;

    printf("ULONG_PTR value: 0x%lx\n", pointerValue);
    return 0;
}
```

**Output:**

```
ULONG_PTR value: 0xdeadbeef
```

---
### Data Types Pointers

The Windows API allows a developer to declare a data type directly or a pointer to the data type. This is reflected in the data type names where the data types that start with "P" represent pointers to the actual data type while the ones that don't start with "P" represent the actual data type itself.

### **1. `PHANDLE` vs `HANDLE`**

- `HANDLE` is the actual handle to an object.
- `PHANDLE` is a pointer to a `HANDLE`.

**Example:**

```c
#include <windows.h>
#include <stdio.h>

int main() {
    HANDLE hFile;        // Actual handle
    PHANDLE pHandle = &hFile;  // Pointer to the handle

    hFile = CreateFile(
        "example.txt", GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL
    );

    if (*pHandle == INVALID_HANDLE_VALUE) {
        printf("Failed to create file!\n");
    } else {
        printf("File created successfully. Handle value: %p\n", (void*)hFile);
        CloseHandle(hFile);
    }

    return 0;
}
```

**Output:**

```
File created successfully. Handle value: 0x0000000000000030
```

---

### **2. `PSIZE_T` vs `SIZE_T`**

- `SIZE_T` is an unsigned integer type used to represent sizes.
- `PSIZE_T` is a pointer to a `SIZE_T`.

**Example:**

```c
#include <windows.h>
#include <stdio.h>

int main() {
    SIZE_T size = 1024;        // Actual size value
    PSIZE_T pSize = &size;     // Pointer to size

    printf("Size value: %llu\n", (unsigned long long)size);
    printf("Pointer to size value: %llu\n", (unsigned long long)*pSize);

    return 0;
}
```

**Output:**

```
Size value: 1024
Pointer to size value: 1024
```

---

### **3. `PDWORD` vs `DWORD`**

- `DWORD` is a 32-bit unsigned integer.
- `PDWORD` is a pointer to a `DWORD`.

**Example:**

```c
#include <windows.h>
#include <stdio.h>

int main() {
    DWORD value = 100;       // Actual value
    PDWORD pValue = &value;  // Pointer to the value

    printf("Original DWORD value: %lu\n", value);

    // Modify the value using the pointer
    *pValue = 200;

    printf("Modified DWORD value via PDWORD: %lu\n", value);

    return 0;
}
```

**Output:**

```
Original DWORD value: 100
Modified DWORD value via PDWORD: 200
```

---

### **When Are These Used in APIs?**

These pointer data types are used in APIs when:

1. A function needs to **return a value** (e.g., updating a handle or size).
2. A function works with **references** instead of values.

For example:

- **`ReadFile` API** uses `LPDWORD` (pointer to DWORD) to return the number of bytes read:
    
    ```c
    BOOL ReadFile(
        HANDLE       hFile,
        LPVOID       lpBuffer,
        DWORD        nNumberOfBytesToRead,
        LPDWORD      lpNumberOfBytesRead, // Pointer to store bytes read
        LPOVERLAPPED lpOverlapped
    );
    ```
    
- **`GetProcessHandleCount` API** requires `PHANDLE` to store the returned process handle.

---

### ANSI & Unicode Functions

The majority of Windows API functions have two versions ending with either "A" or with "W". For example, there is [CreateFileA](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) and [CreateFileW](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilew). The functions ending with "A" are meant to indicate "ANSI" whereas the functions ending with "W" represent Unicode or "Wide".

The main difference to keep in mind is that the ANSI functions will take in ANSI data types as parameters, where applicable, whereas the Unicode functions will take in Unicode data types. For example, the first parameter for `CreateFileA` is an `LPCSTR`, which is a pointer to a constant null-terminated string of **8-bit** Windows ANSI characters. On the other hand, the first parameter for `CreateFileW` is `LPCWSTR`, a pointer to a constant null-terminated string of **16-bit** Unicode characters.

Furthermore, the number of required bytes will differ depending on which version is used.

`char str1[] = "maldev";` // 7 bytes (maldev + [null byte](https://www.tutorialandexample.com/null-character-in-c)).

`wchar str2[] = L"maldev";` // 14 bytes, each character is 2 bytes (The null byte is also 2 bytes)

### Windows API Example

Now that the fundamentals of the Windows API have been laid out, this section will go through the usage of the `CreateFileW` function.

#### Find the API Reference

It's important to always reference the documentation if one is unsure about what the function does or what arguments it requires. Always read the description of the function and assess whether the function accomplishes the desired task. The `CreateFileW` documentation is available [here](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilew).

#### Analyze Return Type & Parameters

The next step would be to view the parameters of the function along with the return data type. The documentation states _If the function succeeds, the return value is an open handle to the specified file, device, named pipe, or mail slot_ therefore `CreateFileW` returns a `HANDLE` data type to the specified item that's created.

Furthermore, notice that the function parameters are all `in` parameters. This means the function does not return any data from the parameters since they are all `in` parameters. Keep in mind that the keywords within the square brackets, such as `in`, `out`, and `optional`, are purely for developers' reference and do not have any actual impact.

```
HANDLE CreateFileW(
  [in]           LPCWSTR               lpFileName,
  [in]           DWORD                 dwDesiredAccess,
  [in]           DWORD                 dwShareMode,
  [in, optional] LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  [in]           DWORD                 dwCreationDisposition,
  [in]           DWORD                 dwFlagsAndAttributes,
  [in, optional] HANDLE                hTemplateFile
);
```

#### Use The Function

The sample code below goes through an example usage of `CreateFileW`. It will create a text file with the name `maldev.txt` on the current user's Desktop.

```
// This is needed to store the handle to the file object
// the 'INVALID_HANDLE_VALUE' is just to intialize the variable
Handle hFile = INVALID_HANDLE_VALUE; 

// The full path of the file to create.
// Double backslashes are required to escape the single backslash character in C
LPCWSTR filePath = L"C:\\Users\\maldevacademy\\Desktop\\maldev.txt";

// Call CreateFileW with the file path
// The additional parameters are directly from the documentation
hFile = CreateFileW(filePath, GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

// On failure CreateFileW returns INVALID_HANDLE_VALUE
// GetLastError() is another Windows API that retrieves the error code of the previously executed WinAPI function
if (hFile == INVALID_HANDLE_VALUE){
    printf("[-] CreateFileW Api Function Failed With Error : %d\n", GetLastError());
    return -1;
}
```

### Windows API Debugging Errors

When functions fail they often return a non-verbose error. For example, if `CreateFileW` fails it returns `INVALID_HANDLE_VALUE` which indicates that a file could not be created. To gain more insight as to why the file couldn't be created, the error code must be retrieved using the [GetLastError](https://learn.microsoft.com/en-us/windows/win32/api/errhandlingapi/nf-errhandlingapi-getlasterror) function.

Once the code is retrieved, it needs to be looked up in [Windows's System Error Codes List](https://learn.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-). Some common error codes are translated below:

- `5` - ERROR_ACCESS_DENIED
- `2` - ERROR_FILE_NOT_FOUND
- `87` - ERROR_INVALID_PARAMETER

### Windows Native API Debugging Errors

Recall from the _Windows Architecture_ module, NTAPIs are mostly exported from `ntdll.dll`. Unlike Windows APIs, these functions cannot have their error code fetched via `GetLastError`. Instead, they return the error code directly which is represented by the `NTSTATUS` data type.

`NTSTATUS` is used to represent the status of a system call or function and is defined as a 32-bit unsigned integer value. A successful system call will return the value `STATUS_SUCCESS`, which is `0`. On the other hand, if the call failed it will return a non-zero value, to further investigate the cause of the problem, one must check [Microsoft's documentation on NTSTATUS values](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-erref/596a1078-e883-4972-9bbc-49e60bebca55).

The code snippet below shows how error checking for system calls is done.

```
NTSTATUS STATUS = NativeSyscallExample(...);
if (STATUS != STATUS_SUCCESS){
    // printing the error in unsigned integer hexadecimal format
    printf("[!] NativeSyscallExample Failed With Status : 0x%0.8X \n", STATUS); 
}

// NativeSyscallExample succeeded
```

