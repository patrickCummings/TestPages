---
title: "Formatting"
date: 2018-11-09T14:32:45-08:00
draft: true
---

## Formatting

### Braces
When programming in C braces should be used by if, else, for, do and while statements. Single line if statements are allowable when the statement is not complex. 

```
if( myStatement ){
 int temp = 0;
 return temp;
}
else{
 somethingElse…;
}

// Statement can be on a single line if simple.
if(myStatement) return 0;
// Also valid
myStatement ? return 0 : somethingElse…;
```

### Variable Declarations/Prefixes

Variables declared in a .var file should only have the initial value specified if it is non zero. Val
```
//Global variables
gMyGlobalVariable

//Member structure
gMyGlobalStructure.in..

//Members Variables
gMyGlobalStructure.in.par.pMyPointer		//pointer
gMyGlobalStructure.in.par.myParameter	//general
gMyGlobalStructure.io.diMyDigitalInput	//IO Point

//For stuctured text dynamic Variables
dMyDynamicVariable ACCESS gMyGlobalStructure.IN.PAR.pMyPointer
dMyDynamicVariable.myMembers..

//For c pointers
MyPoint_type* pMyVariable = gMyGlobalStructure.IN.PAR.pMyPointer;
pMyVariable->myMembers..
MyPoint_type* pMyArray = &MyArray
```
### Structures
Types should use upper camel case and have _type at the end.

```
//General type name
MySubsystemIn_type
	
//Due to name length restrictions (32 characters maximum)
//These structure names are both acceptable for:
mySubsystem.in.par.myParameters

MySubsystemInPar_type
or 
MySubsystemPar_type
```

### Function Calls <a id="introduction"></a>
```
//Function Calls should contain spaces after a comma
myFunction(&myStruct, &myString, strlen(myString));
```