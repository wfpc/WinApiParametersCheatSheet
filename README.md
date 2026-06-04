# How To Use Win32 API Parameters From MSDN

A quick reference for reading Win32 API signatures to use from MSDN(Microsoft Documentaion).

You can access these kind of types and more information by simply seeing related header files (.h) and see the types expected in VisualStudio by using (ctrl + left click) to reference actual defenations. 

I think this can be well for understanding as well.

---

# The Golden Rule

When you see:

```cpp
BOOL SomeFunction(
    TYPE1 param1,
    TYPE2 param2
);
```

You may ask yourself:

> "What type is Windows expecting me to provide?"

---

# Common Win32 Type Patterns

## 1. String Output Buffer

### MSDN says

```cpp
LPWSTR lpBuffer
```

### Translation

```cpp
WCHAR* lpBuffer
```

### What to Create

```cpp
WCHAR buffer[256];
```

### Why?

Windows writes text into your buffer.

### Example

```cpp
WCHAR username[256];
DWORD size = 256;

GetUserNameW(username, &size);
```

---

## 2. String Input

### MSDN

```cpp
LPCWSTR lpName
```

### Translation

```cpp
const WCHAR*
```

### What to Pass

```cpp
L"Hello"
```

or

```cpp
WCHAR name[] = L"Hello";
```

### Example

```cpp
CreateFileW(
    L"test.txt",
    ...
);
```

---

## 3. Number Output

### MSDN

```cpp
LPDWORD pdwValue
```

### Translation

```cpp
DWORD*
```

### What to Create

```cpp
DWORD value;
```

### What to Pass

```cpp
&value
```

### Example

```cpp
DWORD size = 256;

GetUserNameW(username, &size);
```

---

## 4. Number Input

### MSDN

```cpp
DWORD dwSize
```

### Translation

```cpp
unsigned long
```

### What to Create

```cpp
DWORD size = 256;
```

### What to Pass

```cpp
size
```

### Not This

```cpp
&size
```

Only pass `&size` if MSDN says `LPDWORD`.

---

## 5. Structure Output

### MSDN

```cpp
LPSTARTUPINFOW lpStartupInfo
```

### Translation

```cpp
STARTUPINFOW*
```

### What to Create

```cpp
STARTUPINFOW si;
```

### What to Pass

```cpp
&si
```

Windows fills the structure.

---

## 6. Structure Input

### MSDN

```cpp
const SECURITY_ATTRIBUTES* lpAttributes
```

### What to Create

```cpp
SECURITY_ATTRIBUTES sa{};
```

### What to Pass

```cpp
&sa
```

Windows reads it.

---

## 7. Handle Return

### MSDN

```cpp
HANDLE
```

### What to Create

```cpp
HANDLE hFile;
```

### Example

```cpp
hFile = CreateFileW(...);
```

---

## 8. Boolean Return

### MSDN

```cpp
BOOL
```

### Typical Usage

```cpp
if (Function(...))
{
    // Success
}
else
{
    DWORD error = GetLastError();
}
```

---

# Most Common Win32 Types

| MSDN Type | Meaning | Typical Usage |
|------------|----------|---------------|
| `WCHAR` | Unicode character | `WCHAR ch;` |
| `CHAR` | ANSI character | `CHAR ch;` |
| `LPWSTR` | Unicode string buffer | `WCHAR buffer[256];` |
| `LPCWSTR` | Input Unicode string | `L"Hello"` |
| `LPSTR` | ANSI string buffer | `CHAR buffer[256];` |
| `LPCSTR` | Input ANSI string | `"Hello"` |
| `DWORD` | 32-bit unsigned integer | `DWORD x;` |
| `LPDWORD` | Pointer to DWORD | `DWORD x; &x` |
| `BOOL` | Boolean value | `BOOL ok;` |
| `HANDLE` | Generic object handle | `HANDLE h;` |
| `HWND` | Window handle | `HWND hwnd;` |
| `HINSTANCE` | Module handle | `HINSTANCE hInst;` |

---

# Understanding Win32 Prefixes

## LP = Long Pointer

```cpp
LPDWORD
```

means

```cpp
DWORD*
```

---

## LPC = Long Pointer to Const

```cpp
LPCWSTR
```

means

```cpp
const WCHAR*
```

---

## Examples

```cpp
LPWSTR     -> WCHAR*
LPCWSTR    -> const WCHAR*

LPSTR      -> CHAR*
LPCSTR     -> const CHAR*

LPDWORD    -> DWORD*
LPBOOL     -> BOOL*
```

---

# Real MSDN Example: CreateProcessW

## MSDN Signature

```cpp
BOOL CreateProcessW(
    LPCWSTR lpApplicationName,
    LPWSTR lpCommandLine,
    LPSECURITY_ATTRIBUTES lpProcessAttributes,
    LPSECURITY_ATTRIBUTES lpThreadAttributes,
    BOOL bInheritHandles,
    DWORD dwCreationFlags,
    LPVOID lpEnvironment,
    LPCWSTR lpCurrentDirectory,
    LPSTARTUPINFOW lpStartupInfo,
    LPPROCESS_INFORMATION lpProcessInformation
);
```

---

## Translate Each Parameter

| MSDN Type | Meaning | What To Create |
|------------|----------|----------------|
| `LPCWSTR` | Input string | `L"notepad.exe"` |
| `LPWSTR` | Writable string | `WCHAR cmd[] = L"notepad.exe";` |
| `LPSECURITY_ATTRIBUTES` | Structure pointer | `nullptr` or `SECURITY_ATTRIBUTES sa;` |
| `BOOL` | Boolean | `FALSE` |
| `DWORD` | Integer | `0` |
| `LPVOID` | Generic pointer | `nullptr` |
| `LPCWSTR` | Input string | `nullptr` |
| `LPSTARTUPINFOW` | Structure output | `STARTUPINFOW si{};` |
| `LPPROCESS_INFORMATION` | Structure output | `PROCESS_INFORMATION pi{};` |

---

## Actual Usage

```cpp
STARTUPINFOW si{};
PROCESS_INFORMATION pi{};

si.cb = sizeof(si);

WCHAR cmd[] = L"notepad.exe";

BOOL ok = CreateProcessW(
    nullptr,
    cmd,
    nullptr,
    nullptr,
    FALSE,
    0,
    nullptr,
    nullptr,
    &si,
    &pi
);
```

---

# The 10-Second Rule

When reading MSDN:

| If You See | Think |
|------------|--------|
| `LPWSTR` | I need a `WCHAR` buffer |
| `LPCWSTR` | I pass a Unicode string |
| `LPSTR` | I need a `CHAR` buffer |
| `LPCSTR` | I pass an ANSI string |
| `LPDWORD` | Create a `DWORD` and pass `&var` |
| `DWORD` | Pass a number |
| `LPSTRUCT` | Create a structure and pass `&struct` |
| `HANDLE` return | Store the handle |
| `BOOL` return | Check success/failure |

---

# Quick Mental Translation

Whenever you see a Win32 typedef, mentally replace it:

```cpp
LPDWORD   -> DWORD*
LPWSTR    -> WCHAR*
LPCWSTR   -> const WCHAR*

LPSTR     -> CHAR*
LPCSTR    -> const CHAR*
```

If you can translate the typedef, you can understand the API signature.

---

# Practical Workflow

1. Read the function signature.
2. Translate Win32 typedefs into normal C++ types.
3. Determine:
   - Input string?
   - Output string?
   - Input number?
   - Output number?
   - Input structure?
   - Output structure?
4. Create the required variables.
5. Pass pointers only when the API expects a pointer type (`LPXXX`).

Example:

```cpp
BOOL GetUserNameW(
    LPWSTR lpBuffer,
    LPDWORD pcbBuffer
);
```

Translation:

```cpp
BOOL GetUserNameW(
    WCHAR* buffer,
    DWORD* size
);
```

Usage:

```cpp
WCHAR username[256];
DWORD size = 256;

GetUserNameW(username, &size);
```
--- 


Good luck  :)
