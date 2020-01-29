# Raspberry-Pi4-compile-and-link-your-kernel-code-to-the-raspberry-kernel__kernel7l.img__
This is here for me as reference and hope will help someone want to run their code in the raspberry pi kernel.

Let say I want the kernel to set gpio5 (pin 29 on the raspberry pi 4) to 1 when it bootup. (if you connect a led on gpio5, it will light up).

I'm using Ubuntu to cross compile the raspberry pi, both linux and tool are on my root directory.
    
    Please follow this document to clone the raspberry pi on your machine
        
        https://www.raspberrypi.org/documentation/linux/kernel/building.md
     

The linux and tools are on my root directory.

      /home/sitara/linux
      /home/sitara/tools
      
I created a directory under /linux/drivers call hello_gpio5

        sitara@ubuntu:~/linux/drivers$ mkdir hello_gpio5
        sitara@ubuntu:~/linux/drivers/hello_gpio5$
      
It contains a c and make file.        
      sitara@ubuntu:~/linux/drivers/hello_gpio5$ ls
      hello_gpio5.c  Makefile
      
 The code for hello_gpio5.c is shown below. 
    (I'm not using device tree and I'm using the legacy gpio just for demo purpose)

          #include <linux/init.h>
          #include <linux/module.h>
          #include <linux/kernel.h>
          #include <linux/gpio.h>
          #include <linux/interrupt.h>

          static unsigned int GPIO5_PIN = 5;

          static int __init hello_gpio5_init(void) {
                  gpio_request(GPIO5_PIN, "hello-gpio5");
                  gpio_direction_output(GPIO5_PIN, 0);
                  gpio_set_value(GPIO5_PIN, 1);
                  pr_info("hello_gpio5_init\n");
                  return 0;
          }

          static void __exit hello_gpio5_exit(void) {
                  gpio_free(GPIO5_PIN);
                  pr_info("hello_gpio5_exit\n");
          }

          module_init(hello_gpio5_init);
          module_exit(hello_gpio5_exit);

          MODULE_AUTHOR("gpio5");
          MODULE_DESCRIPTION("gpio5 example");
          MODULE_LICENSE("GPL"); 
 
 First, it requests a gpio5, then set the direction to be output and set the value to 1. 
  

And the Makefile           
    
    obj-y			+= hello_gpio5.o       
    
The next step is add your hello_gpio5 to the drivers Makefile.

Add the lines below to the /linux/drivers/Makefile at the end of the Makefile.

    obj-y                           += hello_gpio5/

To compile the code, go to linux directory.
    
    sitara@ubuntu:~$ cd linux/
    sitara@ubuntu:~/linux$ pwd
    /home/sitara/linux
 
Tell where all the tool resides by export it (copy and run the two export on your ubuntu)
 
    export CCPREFIX=~/tools/arm-bcm2708/arm-bcm2708-linux-gnueabi/bin/arm-bcm2708-linux-gnueabi-  
    export KERNEL_SRC=~linux
 
Run the below commands on your ubuntu. it will take couple hours.

    KERNEL=kernel7l
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2711_defconfig
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs
    ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=./modules make modules_install

Wait for it done then run the command below to generate the kernel7l.img.

    ./scripts/mkknlimg arch/arm/boot/zImage kernel7l.img   
    
Now copy the kernel7l.img to your raspberry pi from your ubuntu. Your IP address might not be the same.

    scp kernel7l.img pi:10.1.10.77:
    
I see the kernel7l.img on my raspberry pi as shown below.    
    
    pi@raspberrypi:~ $ ls kernel7l.img 
    kernel7l.img
    
Now copy kernel7l.img to /boot. (make sure save your kernel7l.img to kernel7l.img.save before run this command).
    
    pi@raspberrypi:~ $ sudo cp kernel7l.img /boot
    pi@raspberrypi:~ $ sudo reboot

Wait for it reboot then use dmesg and look for 'hello_gpio5_init'

    [    0.384307] [vc_sm_connected_init]: end - returning 0
    [    0.385132] hello_gpio5_init
    [    0.385444] Initializing XFRM netlink socket
    
Wow, too many steps.

Have fun.
    
