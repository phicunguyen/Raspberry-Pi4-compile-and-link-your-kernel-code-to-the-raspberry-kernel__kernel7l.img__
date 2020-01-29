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
    (I'm not using device tree and using the legacy gpio.)

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

