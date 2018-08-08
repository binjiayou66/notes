1. \_\_attribute\_\_((constructor))与\_\_attribute\_\_((destructor))，前者在main之前调用，后者在main之后调用（注：由于iOS应用程序main函数被UIApplicationMain()方法卡住，所以后者并不会被调用。经测试，杀死iOS应用程序，也不会调用。）

   ```c++
   #include<stdio.h> 
   __attribute__((constructor)) 
   void before_main() { 
      printf("before main\n"); 
   } 

   __attribute__((destructor)) 
   void after_main() { 
      printf("after main\n"); 
   } 

   int main(int argc, char **argv) { 
      printf("in main\n"); 
      return 0; 
   }
   ```

   ```
   //这个例子的输出结果将会是：
   before main
   in main
   after main
   ```

   ​

2. dyld挂钩子

   ```c++
   /*
    * The following functions allow you to install callbacks which will be called   
    * by dyld whenever an image is loaded or unloaded.  During a call to _dyld_register_func_for_add_image()
    * the callback func is called for every existing image.  Later, it is called as each new image
    * is loaded and bound (but initializers not yet run).  The callback registered with
    * _dyld_register_func_for_remove_image() is called after any terminators in an image are run
    * and before the image is un-memory-mapped.
    */
   void _dyld_register_func_for_add_image(void (*func)(const struct mach_header* mh, intptr_t vmaddr_slide));
   void _dyld_register_func_for_remove_image(void (*func)(const struct mach_header* mh, intptr_t vmaddr_slide));
   ```

   ​

3. ​