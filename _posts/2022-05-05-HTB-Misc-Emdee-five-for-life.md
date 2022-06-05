## HTB Misc - Emdee five for life
---


![](/docs/images/emdee-five-for-life/emdee1.png)


This was a pretty easy scripting challenge that involved pulling the data from the website and sending an input form. 

I'm going to be using [Selenium](https://www.selenium.dev/documentation/webdriver/) in Python, but the process is pretty similar for Java, C#, Ruby, or Kotlin since they all support the Selenium WebDriver.

But Python is easy so yeah.


#### Challenge Format and Info

The [Emdee five for life](https://app.hackthebox.com/challenges/67) challenge is listed as a Misc challenge on HackTheBox, but personally it's more of a programming category challenge. HTB doesn't have that category so into Misc it goes.


![](/docs/images/emdee-five-for-life/emdee2.png)

Upon opening the page, we're greeted by a powder blue background with an randomly generated string of text and a space for an MD5 hash.

So let's input something:


![](/docs/images/emdee-five-for-life/emdee3.png)

Looks like there's a super short window for inputting the value. I'm gonna skip the fuzzing and stuff and just start on the script.


#### Scripting Stuff

Firstly we need to get the random value once the page is loaded in.

The beginning part of the script that I made looks like this (im pretty bad at coding so don't judge):

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Firefox()
driver.get('http://138.68.188.223:30448')
uuuuuhhhh = driver.find_elements(By.TAG_NAME, "h3")

item = 0
print('Random String:')
for u in uuuuuhhhh:
    item = u.text
    print(u.text)

```


Now I'm lazy so there is most definitely an easier way to save a global variable.

Put the output works just fine, since there is only one `<h3>` tag on the page, the string, it output perfectly. 


![](/docs/images/emdee-five-for-life/emdee4.png)


Alright now we can work on the md5 part.

The code I used for this part looks like this:

```python
import hashlib

md5hash = hashlib.md5(item.encode())
md5hash = md5hash.hexdigest()

print('MD5 Hashed Value:')
print(md5hash)
```

So now this hashes the value in Python using the [hashlib](https://docs.python.org/3/library/hashlib.html) library, saves and prints the values.


![](/docs/images/emdee-five-for-life/emdee5.png)

So now all we have to do is input the values in the form and we're golden.

The code I used for this final part is below:

```python
input_box = driver.find_elements(By.TAG_NAME, "input")
input_box[0].send_keys(md5hash)

input_box[1].click()
```

So the logic behind this is that there are 2 input fields on the page. The first one is the MD5 input field, and the second is the 'Submit' button. So Selenium already has some easy functions to process these and we can just fill them up.

![](/docs/images/emdee-five-for-life/emdee6.png)

And there's our flag.
