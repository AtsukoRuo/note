# Regular Expression

Regular Expression（Regex）is symbols representing a text pattern that allows us to match text



在JavaScript中创建以及使用Regex

~~~javascript
let txt;
let regex1 = new RegExp("Hello");
let regex2 = /world/;					//suggested

regex1.test(txt);				     // return true if pattern is found in txt
regex1.exec(txt);					// return array of matches. and other properties(index, input) attached to that array 


txt.match(regex1);					 
txt.search(regex1);					
txt.replace(regex1, "hi");			
txt.split(regex1);					
~~~

![image-20230801104737803](assets\image-20230801104737803.png)

![image-20230801105509973](C:\Users\AtsukoRuo\Desktop\note\Git\assets\image-20230801105509973.png)



## Flags

~~~javascript
/pattern/flags
new RegExp("pattern". "flags");
~~~

- `g`：global
- `i`：case insensitive
- `m`：multi-line



## Metacharacters

![image-20230801111023993](C:\Users\AtsukoRuo\Desktop\note\Git\image-20230801111023993.png)

通过`\`可以转义元字符、表示控制字符、表示一些普通字符



控制字符的表示

- `\t`：制表符
- `\n`：换行符
- `\s`：空格，或者在pattern中直接敲入空格即可



- `.`：通配符，匹配**任意一个**（必须有一个字符，不能没有）字符（除换行符）

  `/h.t/g`

  ![image-20230801111525345](C:\Users\AtsukoRuo\Desktop\note\Git\assets\image-20230801111525345.png)

  注意最后一个匹配是包含Tab



