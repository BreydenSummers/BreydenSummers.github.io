---
layout: post
title:  "SOC CTF 2024 Reversing Writeup"
summary: "My solutions to all the problems in the Reversing Section of the in-house CTF for USU"
author: BreydenSummers
date: '2024-04-02 22:07:15 +0000'
category: CTF
thumbnail: /assets/img/posts/SOC-CTF-2024/header.png
keywords: CTF, Utah State University, OSINT
permalink: /blog/SOC-CTF-2024-Reversing-Writeup/
---

## File Carving
As the name suggests for this problem we will have to do some file carving. A simple way to get text out of a program is to use strings. We can compare that with grep to look into the file for our flag:
```bash
strings JustAnExecutableFile | grep -A 10 SOC
```
From that we get this output:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/d762f2ec-3b84-4aec-9af1-867e30eb83f3)<br>
We can Just guess and reconstruct the flag and flag markers until we get: SOC_CTF{passwords_in_executables_is_a_great_idea}

## Good Duck Debugger
For this challenge we will want to use GDB to step throught the problem to solve this. To start we can start up gdb with the file:
```bash
gdb ./GoodDuckDebugger
```
From here we can run the info functions command to see what functions are in the program:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/b5c4954a-58a9-4ac3-9d2b-0a2340e9f3c2)<br>
Lets take a look in main:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/88b06b14-44e0-43d9-8635-9209d13284ff) <br>
We can see that the eax register is modified at the bottom of the function so we will just break on the main function and step through it to get our register. We can break on main using this command as well as view the location of the program counter as well:
```bash
break main
layout asm
run
```
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/d4a8d025-f710-4884-8a90-6faa2e81828e)<br>

Now that our program is currently executing in the main function we can see the memory location for the return function we can grab that value and run the program until that point.(c stands for continue)
```bash
break *0x401142
c
```
Now we can just retreive the value of the register:
```bash
print/d $eax
```
Then you can just wrap that value in the flag markers to get the flag: SOC_CTF{307019}.


## No True Cryptographer
For this problem You'll need to fire up ghidra. Or this [website](https://dogbolt.org/) can get you started. When we look at the decompiled program there are a few things we can note. Firstly that this is an array initialization:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/3da36496-da11-43b8-9759-60074d4a2b49) <br>
And we can see it getting assigned down here:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/76dded64-93bb-40c2-8f5a-6b75e89f2270)<br>
If we convert the values to ascii we can see that the values are too large so lets start with integers. So we can treat it like an array of integers for now. Now we will start wanting to rename variables so we can understanding what is going on in the program. We can start with the array pointer at the beginning of the array. <br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/6557aa33-9665-49b4-a676-af948af2a7b8) <br>
Next we can work on what it is doing to the input to the program. We can see this function:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/158cc68e-12b4-4ff5-ab2d-75279435e8b4)<br>
It looks like it is looping through a string (*chr) and counting up the length we can change the variable names to reflect that:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/aa77ffaa-bc3e-447e-931a-15698d8da669)<br>

In the next bit ofcode I would have to say that the array is being written over with a new array:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/2c6bb1d5-f3f2-46ff-b7df-b1edee418945)<br>
So that is pretty interesting. So we probably are not meant to see the data in the original array. Now we can analyse the last bit of code in the function:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/7518cad9-18bb-470c-9c16-254149d88b05)<br>
I'll go ahead and name some of the variables in this function:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/a9f92530-2f3b-4d71-9061-240c4032ee1f)<br>
It is now understandable enough for us to figure out what is going on in this function. For us to understand this we have to understand that an int is 4 bytes in C. And since we are working directly in memeory we can access values in the array by offseting it by 4 bytes times the position variable. Now that we know that we can see that we are adding a random number to value at the end of the array. And then for the  rest of the values we are just adding the value at the current position and the value at the next position and putting the resultant value in the current position. Now that we know this dependancy relation we can finally solve the problem! Assuming the original values in the array are what we need we can undo what this program did to get our unencrypted text. I wrote a quick python script to do this:
```python
result = [162, 146, 162, 162, 151, 154, 193, 195, 183, 223, 213, 203, 219, 225, 216, 229, 216, 178, 194, 220, 210, 212, 221, 211, 196, 177, 183, 210, 220, 229, 219, 216, 210, 179, 188, 209, 220, 311, 196]
for j in range(len(result), - 1, -1):
    if j >= 0 and j < len(result) - 1:
       result[j] = result[j] - result[j+1]
print(''.join(map(chr, result)))
```
I get this text out: SOC_CTF{Hopefully_Someone_Removes_ThisÄ. We already know the format for the flag so we can replace that character with } and get this as our flag: SOC_CTF{Hopefully_Someone_Removes_This}. 



Here is my reversed program for reference:
```c

undefined8 main(int param_1,undefined8 *param_2)

{
  int Random_Number;
  undefined8 uVar3;
  ulong uVar4;
  time_t tVar5;
  undefined8 **ppuVar6;
  long in_FS_OFFSET;
  undefined8 *local_108;
  int local_fc;
  int input_length;
  int i;
  int j;
  int Random_Number_Between_41_and_80;
  long input_string_first_char;
  long input_length_minus_one;
  undefined *Array_pointer;
  undefined4 local_c8;
  undefined4 local_c4;
  undefined4 local_c0;
  undefined4 local_bc;
  undefined4 local_b8;
  undefined4 local_b4;
  undefined4 local_b0;
  undefined4 local_ac;
  undefined4 local_a8;
  undefined4 local_a4;
  undefined4 local_a0;
  undefined4 local_9c;
  undefined4 local_98;
  undefined4 local_94;
  undefined4 local_90;
  undefined4 local_8c;
  undefined4 local_88;
  undefined4 local_84;
  undefined4 local_80;
  undefined4 local_7c;
  undefined4 local_78;
  undefined4 local_74;
  undefined4 local_70;
  undefined4 local_6c;
  undefined4 local_68;
  undefined4 local_64;
  undefined4 local_60;
  undefined4 local_5c;
  undefined4 local_58;
  undefined4 local_54;
  undefined4 local_50;
  undefined4 local_4c;
  undefined4 local_48;
  undefined4 local_44;
  undefined4 local_40;
  undefined4 local_3c;
  undefined4 local_38;
  undefined4 local_34;
  undefined4 local_30;
  long local_20;
  long lVar1;
  
  ppuVar6 = &local_108;
  local_fc = param_1;
  local_108 = param_2;
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  if (param_1 == 2) {
    local_c8 = 0xa2;
    local_c4 = 0x92;
    local_c0 = 0xa2;
    local_bc = 0xa2;
    local_b8 = 0x97;
    local_b4 = 0x9a;
    local_b0 = 0xc1;
    local_ac = 0xc3;
    local_a8 = 0xb7;
    local_a4 = 0xdf;
    local_a0 = 0xd5;
    local_9c = 0xcb;
    local_98 = 0xdb;
    local_94 = 0xe1;
    local_90 = 0xd8;
    local_8c = 0xe5;
    local_88 = 0xd8;
    local_84 = 0xb2;
    local_80 = 0xc2;
    local_7c = 0xdc;
    local_78 = 0xd2;
    local_74 = 0xd4;
    local_70 = 0xdd;
    local_6c = 0xd3;
    local_68 = 0xc4;
    local_64 = 0xb1;
    local_60 = 0xb7;
    local_5c = 0xd2;
    local_58 = 0xdc;
    local_54 = 0xe5;
    local_50 = 0xdb;
    local_4c = 0xd8;
    local_48 = 0xd2;
    local_44 = 0xb3;
    local_40 = 0xbc;
    local_3c = 0xd1;
    local_38 = 0xdc;
    local_34 = 0x137;
    local_30 = 0xc4;
    input_string_first_char = param_2[1];
    for (input_length = 0; *(char *)(param_2[1] + (long)input_length) != '\0';
        input_length = input_length + 1) {
    }
    input_length_minus_one = (long)input_length + -1;
    uVar4 = (((long)input_length * 4 + 0xfU) / 0x10) * 0x10;
    for (; ppuVar6 != (undefined8 **)((long)&local_108 - (uVar4 & 0xfffffffffffff000));
        ppuVar6 = (undefined8 **)((long)ppuVar6 + -0x1000)) {
      *(undefined8 *)((long)ppuVar6 + -8) = *(undefined8 *)((long)ppuVar6 + -8);
    }
    lVar1 = -(ulong)((uint)uVar4 & 0xfff);
    if ((uVar4 & 0xfff) != 0) {
      *(undefined8 *)((long)ppuVar6 + ((ulong)((uint)uVar4 & 0xfff) - 8) + lVar1) =
           *(undefined8 *)((long)ppuVar6 + ((ulong)((uint)uVar4 & 0xfff) - 8) + lVar1);
    }
    for (i = 0; i < input_length; i = i + 1) {
      *(int *)((long)ppuVar6 + (long)i * 4 + lVar1) = (int)*(char *)(input_string_first_char + i);
    }
    Array_pointer = (undefined *)((long)ppuVar6 + lVar1);
    *(undefined8 *)((long)ppuVar6 + lVar1 + -8) = 0x1014c3;
    tVar5 = time((time_t *)0x0);
    *(undefined8 *)((long)ppuVar6 + lVar1 + -8) = 0x1014ca;
    srand((uint)tVar5);
    *(undefined8 *)((long)ppuVar6 + lVar1 + -8) = 0x1014cf;
    Random_Number = rand();
    Random_Number_Between_41_and_80 = Random_Number % 41 + 40;
    *(int *)(Array_pointer + (long)(input_length + -1) * 4) =
         *(int *)(Array_pointer + (long)(input_length + -1) * 4) + Random_Number_Between_41_and_80;
    for (j = 0; j < input_length; j = j + 1) {
      *(int *)(Array_pointer + (long)j * 4) =
           *(int *)(Array_pointer + (long)j * 4) + *(int *)(Array_pointer + (long)(j + 1) * 4);
    }
    uVar3 = 0;
  }
  else {
    printf("Usage: %s <string>\n",*param_2);
    uVar3 = 1;
  }
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar3;
}


```





