# Cheatsheet to connect Arduino with WSL2
1. On Windows PowerShell:
   ```bash
   usbipd list # to see the correct busid
   usbipd bind --busid <busid> # DO IT ONLY FIRST TIME to share the busid
   usbipd attach --wsl --busid <busid> # attach the busid
   ```
3. On Ubuntu:
   ```bash
   lsusb # to check the usb connection
   sudo chmod 777 /dev/<usb_port> #i.e., sudo chmod 777 /dev/ttyACM0
   ```
