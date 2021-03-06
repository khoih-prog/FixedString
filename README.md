# Arduino/ESP8266 Fixed String

String library for embedded systems that won't make your heap fragmented

###### Why this library was created?

 - Arduino String uses `malloc()` internally and will eventually make heap fragmented and program will crash
 - C strings are too hard to use and it's way too easy to overflow buffer and that will be hard to debug
 -Heap fragmentation is especially dangerous in Embedded systems due to low memory, e.g Arduino has 2k bytes only, esp8266
 - I strongly recommend reading following articles:
   - [The evils of arduino strings](https://hackingmajenkoblog.wordpress.com/2016/02/04/the-evils-of-arduino-strings/)
   - [What is heap fragmentation](https://cpp4arduino.com/2018/11/06/what-is-heap-fragmentation.html)
###### Features  
  - No need to use plain string function like strlen, strcmp anymore
  - It's safer than using raw arrays
  - String size needs to be specified upfront, like in C string:
    So instead of using:
    `char[20] str = "some string"`
    use:
    `SimpleString<20> str = "some string"`
  - c_str() method will return null terminated string
  - length() will return actual string length, not including null character
  - appendFormat() adds requested format string into FixedString, e.g: `str.appendFormat("%s %d", "abc", 10)` will append `abc10`
  - Supports storing binary buffers with '\0' characters
  - It has overhead of 5 bytes, 2 bytes for length, 2 bytes for capacity and 1 byte to allow returning null terminated string. For example, FixedString<20> will use 25 bytes of memory
  - When string buffer overrun occurs, program won't crash, instead glabl variable FixedString_OverflowDetected will be set to true.
###### Cons
  - C++ Generics are used to specify buffer size which makes compile time longer and program size larger
  - It is not expandable, you have to know maximum expected size of string at compile time, same as `char[NN]` array  
###### Todo
  - Add more methods, for example replace, trim - contributions are welcome!
###### Example:
 
 Following program:
 
```C++
#include <FixedString.h>


void setup() 
{
   Serial.begin(115200);   
   delay(500);
    FixedString<100>  str= "abc";
    str.debug();
    str.append('x');
    str.debug();
    str.append('y');
    str += "12345";
    str.debug();
    str += "12";
    str.debug();

    FixedString<100> str1 = FixedString<100>("Some ") + " may never live";
    str1.debug();

    FixedString<50> str3 = " but the crazy will never die";
    str1 += str3;

    str1.debug();

    FixedString<8> formatString;
    formatString.appendFormat("%d %d %d", 10, 20, 30);
    
    formatString.debug();
    // string is full, this append will fail and set FixedString_OverflowDetected to true
    formatString.append('X'); 
    formatString.debug();

    FixedString<4> formatString1;

    // this will fail as format string result will have 5 byte length
    formatString1.appendFormat("%d %d", 10, 20);
    formatString1.debug();

    formatString1.appendFormat("%d%d", 10, 20);
    formatString1.debug();
}

void loop() 
{
   if(FixedString_OverflowDetected)
   {
    Serial.println("Fatal error, one of FixedString were to small to fit content!");
    for(;;) delay(500);
   }  
}
```
Will output:
```
content: 'abc' length: 3, free = 97
content: 'abcx' length: 4, free = 96
content: 'abcxy12345' length: 10, free = 90
content: 'abcxy1234512' length: 12, free = 88
content: 'Some  may never live' length: 20, free = 80
content: 'Some  may never live but the crazy will never die' length: 49, free = 51
content: '10 20 30' length: 8, free = 0
content: '10 20 30' length: 8, free = 0
Fatal error, string overun detected!
content: '' length: 0, free = 4
Fatal error, string overun detected!
content: '1020' length: 4, free = 0
Fatal error, string overun detected!
Fatal error, one of FixedString were to small to fit content!

```
