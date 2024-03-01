# Preparing the Host

## Host BIOS Settings

In order to run a hypervisor on your machine, you must enable virtualization
in your computers bios settings.

- VT-x on Intel
- SVM on AMD

## Disable Hyper-v

By default, Windows now runs under a hypervisor. This becomes problematic
as it prevents VMWare from running its virtual machines under its own
hypervisor as it's forced to use Hyper-v's hypervisor. This causes
VMWare virtual machines to be slow, unreliable, and sometimes unresponsive.

To disable Hyper-v:
1. Open `Turn Windows Feature On or Off`
2. Disable all "virtualization" and "hyper-v" related features
3. Open powershell as an administrator and run `bcdedit /set hypervisorlaunchtype off`
4. Restart your computer