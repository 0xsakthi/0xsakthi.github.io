---
title: "Programming - Blue"
classes: wide
tag: 
  - "Cpp"
  - "Cheat-sheet"
  - "DSA"
header:
  teaser: /assets/images/cpp.png
ribbon: Blue
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
# vectors 
``` cpp
#include <bits/stdc++.h>
#include <vector>
using namespace std;

int main(){
	vector<int> v;
	int n = 8;
	v = {1,2,3};
	int sum = 0;
	while(!v.empty()){ //empty() returns true or false value
		sum += v.back(); //back() returns last value of array
		v.pop_back(); //pop_back() to delete last value of array
	}
	cout << sum; //output is 6
	cout << v.front(); //output is 1(it returns front value)
	v.insert(v.end(),9); //we can insert the value in end of the array
	v.insert(v.begin(),4); // we can insert the value in begin of the array
	cout << v.back(); //9
	cout << v.front();//4
}
```
