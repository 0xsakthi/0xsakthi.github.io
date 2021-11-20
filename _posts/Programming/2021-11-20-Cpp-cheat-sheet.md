---
title: "Programming - Blue"
classes: wide
tag: 
  - "Cpp"
  - "Cheat-sheet"
  - "DSA"
header:
  teaser: /assets/images/cpp.png
ribbon: Sky Blue
description: "CheatSheet - Cpp"
categories:
  - pg
---

## "cctype Library"

*change the lowercase to uppercase and uppercase to lowercase using **toupper()** and **tolower()** Functions*

``` cpp
#include <iostream>
#include <cctype>
using namespace std;

int main(){
	char character;
	cin >> character;
	if (character>='a' && character<='z')
	{
		cout<<(char) toupper(character);
	}
	else if(character>='A' && character<='Z'){
		cout << (char) tolower(character);
	}
	else{
		cout<<"other characters";
	}

	return 0;
}
```
