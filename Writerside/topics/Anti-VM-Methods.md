# Anti VM Methods

In order to understand how to bypass anti-vm measures. We must first
understand how they work. Here's a few popular methods and their implementations.

TODO: Add more examples and methods

## CPUID Based Attacks

CPUID is an x86 instruction that returns details about the processor.
This however becomes an issue as it allows an application to query things
such as the "hypervisor flag" and "hypervisor brand."

This method is commonly seen in anti cheats and software packers/obfuscators
such as VMProtect and Enigma.

### Example {id="cpuid-attack-example"}

```C++
	int cpu_info[4]; // Buffer for the cpuid data
	__cpuid(cpu_info, 1); // Calling cpuid with the buffer and 0x1
	if ((cpu_info[2] >> 31) & 1) { // Checking the hypervisor flag
    // Resetting the cpuid buffer
		cpu_info[1] = 0;
		cpu_info[2] = 0;
		cpu_info[3] = 0;
		__cpuid(cpu_info, 0x40000000); // Calling cpuid in order to get the hypervisor brand
    // Checking to see if the hypervisor brand is "Microsoft Hv"
		if (cpu_info[1] == 0x7263694d && cpu_info[2] == 0x666f736f && cpu_info[3] == 0x76482074) {
			cpu_info[1] = 0;
			__cpuid(cpu_info, 0x40000003);
			if (cpu_info[1] & 1) // Check the hyperv leaf
				return false;
		}
		return true;
	}
```

## SMBIOS String Matching

The SMBIOS table is populated with information passed from the BIOS of the 
computer. It can be used to identify things such as the manufacturer
and model of the computer. Under a virtual machine this could include
information such as the hypervisor.

This method is commonly seen in anti cheats and software packers/obfuscators
such as VMProtect.

### Example {id="smbios-string-match-example"}

```go
import (
	"fmt"
	"github.com/digitalocean/go-smbios/smbios"
	"strings"
	"time"
)

// Common strings that can be found in the SMBIOS table
var commonStrings = []string{
	"qemu",
	"virtualbox",
	"vbox",
	"vmware",
	"kvm",
	"hyperv",
}

func biosCheck() bool {
	rc, _, _ := smbios.Stream() // Retrieve the SMBIOS table
	defer rc.Close() // Close the stream on function return
	d := smbios.NewDecoder(rc) // Open a new decoder
	ss, _ := d.Decode() // Decode the table
	for _, s := range ss { // Iterate over the SMBIOS table
		data := strings.ToLower(fmt.Sprint(s)) // Convert to a string and lower
		if StringArrayContains(commonStrings, data) { // Check if the string contains a common word
			return true
		}
	}
	return false
}
```

## MAC Addresses

Applications can query network adapters using `GetAdaptersAddresses`
and check the `vendor` of the mac address of the adaptor. 
The vendor in most virtual machines will identify the hypervisor.

## Process Enumeration

Probably the simplest way of identifying a virtual machine.
Applications can query the list of running processes on a computer
and compare them against a list of hardcoded names. For example
VMWares SVGA helper.