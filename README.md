My Friend Rafi had an idea of making a manikin head with a computer that displays a ai chat, roleplaying as a depiction of a IRobot robot. He asked me if i could help with the electronics and i agreed to help.

From my experience and knowledge, I figured the plan would be something along the lines of:
1. Get a Raspberry pi/equivalent and a small LCD display (SPI)
2. install and setup the OS
3. Enable SPI on the SBC and connect the LCD display
4. Install ollama and a language model
5. Run ollama
6. Profit
It seemed pretty easy.

I searched on the market for a Raspberry pi, but our budget was very limited so i had to scour for a cheaper SBC. After searching for possibilities online, the one that is readily available is the Orange Pi SKU. I decided to get the Orange Pi Zero 3 2GB for it's similarity to the Raspberrypi and the lower price. With only 2GB of RAM, we would need to use a very light Language model such as Deepseek-r1:1.5b or llama3.2:1b.

With that in mind, I listed the components needed to be purchased:

| Component                | Quantity      | Price  | Link                                                                                                     |
| ------------------------ | ------------- | ------ | -------------------------------------------------------------------------------------------------------- |
| Orange Pi Zero 3         | 1             | 499000 | https://www.tokopedia.com/badank/orange-pi-zero-3-2gb-ram<br>                                            |
| LCD Display 3.5 Inch TFT | 1             | 160000 | https://www.tokopedia.com/solarperfect/lcd-display-module-3-5-inch-tft-touch-screen-for-raspberry-pi<br> |
| Type C power 5V/3A       | If needed     | -      | -                                                                                                        |
| SD Card                  | at least 16gb | -      | -                                                                                                        |
| Keyboard                 | If needed     | -      | -                                                                                                        |
With the components purchased, I went to work. I instructed Rafi on downloading the official Ubuntu server distro provided by Orange Pi and started configuring the OS with Tailscale so that i can connect to it remotely. We didn't need the GUI of Linux, as the terminal is sufficient enough to display the language model and this saves space and RAM. I then started configuring the OPi for the display, but in doing so, I encountered a lot of problems.
- There was a old way of setting up the display using "fbtft" command. But the version of Linux given by Orange Pi has already **deprecated** that command and made it obsolete.
- I had to find a workaround and came across the Armbian forum threads about connecting the display by changing the "Tree Directory".
- It took a while to understand it as i followed the thread discussion.

The thread discussion used a file with the .dts extension that can be compiled with the "orangepi-add-overlay" command. This will store it for the Kernel branch as a driver to run at startup (.dto). I can check the kernel logs for SPI and the driver by using "dmesg | grep spi". But alas, it seemed like the SPI was not able to find the device for some odd reason. I ended up wanting to switch to the community driven Armbian Debian to see if it would make a difference

At first, the Armbian OS seemed promising, with it being able to detect the device, although there was still no output. I tried many configurations and read through the thread many times to see if I was missing something but it all came to no results. I read some threads about the CS pin not being enabled properly from the armbian-config, and having to manually replace the spidev_1_1.dtbo to enable it. I don't know if this was the one to make a difference at the end, but at this time it still didn't work. I modified the ili9341.dts many times, mostly changing the GPIO pins to make sure it's the correct one.  

The most useful [thread](https://forum.armbian.com/topic/46824-orange-pi-zero-3-ili9486-tft-lcd/) i found is a user trying the same thing as we are right then. They included their findings and code to be compiled as an overlay for the OPi, essentially the drivers for the display. They also included the code for when the display is directly connected, which really helped a lot. 

Having fresh installed the Armbian OS the second time, I put everything i knew into a sequential process:

1. Install WiringOP to get the gpio command
2. enable SPI through the armbian-config
3. check whether the CS1 pin is on
4. It was expected to be off either way
5. Copy over the sun50i-h616-spidev1_1.dtbo and sun50i-h616-spi-spidev.dtbo to /boot/dtb-6.6.72-current-sunxi64/allwinner/overlay
6. "sudo reboot"
7. The CS1 pin should display ALT4, meaning that it's configured correctly
8. Check again with "ls /dev/spi*", there should be a output of "spidev1.1"
9. "nano ili9341.dts" to edit the configuration
10. Copy and paste the configuration from the thread, making sure the GPIO pin for the CS0 is at PH9 and the touch at PC15
11. Save and exit
12. "sudo armbian-add-overlay ili9341.dts", It should be added to the overlay of the /boot/armbianEnv.txt
13. "sudo reboot"
14. The display should now work, you can check the kernel logs with "dmesg | grep spi"

It took quite a while to get the screen working, but I'm happy that I could get it to work. With that out of the way, installing ollama is a easy curl command and "ollama run <model>". 