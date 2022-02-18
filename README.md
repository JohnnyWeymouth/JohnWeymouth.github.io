# Two ways to sanitize user input using JavaScript

## Introduction

### Importance of sanitizing your user inputs

Before we begin this tutorial, it is vital to convey the importance of protecting against Cross Site Scripting and other injection methods. 

Sanitizing user input is key for protecting your site. Effectively, a hacker can run code on the user’s browser by entering it in a place where the user can input text or value (such as a date). 

Of the different ways malicious users can run code instead of normal user input, the one we will be discussing today (and working to prevent) is Cross Site Scripting.


### What is Cross Site Scripting?

[OWASP](https://owasp.org/www-community/attacks/xss/) gives a good overview of what Cross Site Scripting is, and how it is often used.

> “Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. XSS attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end user. Flaws that allow these attacks to succeed are quite widespread and occur anywhere a web application uses input from a user within the output it generates without validating or encoding it.”

Cross site scripting, or XSS, is one of the most common forms of attacks on the web. XSS combined with sql and other injection methods are some of the most popular attacks. In 2017, they were ranked number one by [OWASP](https://owasp.org/www-project-top-ten/2017/Top_10) in the Top 10 Application Security Risks. With this in mind, it is so crucial to defend against these types of attacks.

Let’s get into an example where a website does not protect against cross site scripting. A user inputs into a form, but instead of inputting what he or she is supposed to, our malicious user inputs
> <script>alert("You have been hacked!");</script>

into the form data, it could cause issues. If the form data is used later in the website, the alert “You have been hacked!” will display on the screen.

This is because, when the variable holding the malicious user’s form data is read into the html, the browser reads it as JavaScript, and not as text. The tags around the line of code indicate to the browser that this is code and should be treated like code. The browser obeys and will display it when loading the html that contains the form data.



## Tutorial


For each of these methods, we will be trying to change the unsanitized input in order to make it safe. When the browser reads the html that contains the new sanitized input, the input will not be interpreted as code within the html, but as text input.

### Method 1: “Manual”

#### Background info

Both of these methods will sanitize the user inputs by replacing any tags (< or > symbols. They are often referred to as less than and greater than respectively) with ‘& lt’ and ‘& gt’.
> Note: in the actual code where we replace these, there is NO SPACE between & and lt, nor & and gt. I had to put a space for reasons that will become clear below.

Lt stands for less than and gt stands for greater than. The reason we want to format user input tags like that, instead of, for example, just deleting them, is because the browser will still be able to display any greater than and less than symbols that were put there innocently.

For example, if a user makes their username ‘I_<3_Cybersecurity’, we would not want a welcome message to the user to be 'Welcome I_3_Cybersecurity!'

At the same time, we still need to get rid of those pesky cross site scripting attacks.

The solution- the reason we are replacing the less than and greater than symbols with ‘& lt’ and ‘& gt’ is because all browsers know to replace ‘& lt’ and ‘& gt’ with ‘<’ and ‘>’ when displaying html (and when displaying many other formats).
> As you now likely understand, the reason I have been writing ‘& lt’ and ‘& gt’ with spaces is because your browser would just see them in the markdown as ‘&lt’ and ‘&gt’.

And thus we get to what I would like to describe as "manual" method.

#### Code

Manual method works by writing some code to do the process step by step as described above. We will be using an html form as our input.

Here's the code:
```js
function SanitizeFormDataThenDoSomething(event) {
  let formData = Object.fromEntries(new FormData(event.currentTarget))
  formData.text = formData.text.replaceAll("<","&lt;")
  formData.text = formData.text.replaceAll(">","&gt;")
  // formData of text is now sanitized.
  // You can now do something with it
}
```
> MOST xss attacks will not be able to counter this. A really really clever hacker could still get in, but this tutorial is not made to perfectly defend against all injection attacks. It is just showing how to defend against most of them. Using a trustworthy external file made to defend against very robust xss, you can do far better against these high profile attackers.


#### Event Listener when using HTML Form
The 'event' argument indicates the data from the form that is submitted because that form has been marked with an event listener. This is necessary to do beforehand to accomplish this tutorial. Here is an example of this.

```html
<form onsubmit="SanitizeFormDataThenDoSomething(event)">
        <input name="text" type='text' required/> <br>
        <input name="date" type='date' required/> <br>
        <button><span>Submit this form</span></button>
</form>
```

#### Explanation

Now that setup is out of the way, let's explain how this code works. Alternatively you might have just scrolled down to this point, grabbed the code, and scampered off like a raccoon that just got into the cat's food bowl. That's ok. Sometimes that's all you really need.

![Thief](https://c.tenor.com/ZELZMDeQNjIAAAAC/raccoon-stealing.gif)

For those of you that are still reading after that raccoon metaphor, let's dive into the JavaScript that makes the sanitization possible.

The event is prepared as a new FormData method using the built in JavaScript code. [Visual Studio Code](https://microsoft.github.io/PowerBI-JavaScript/interfaces/_node_modules_typedoc_node_modules_typescript_lib_lib_dom_d_.formdata.html) explains this built in JavaScript.

> "[This code p]rovides a way to easily construct a set of key/value pairs representing form fields and their values, which can then be easily sent using the XMLHttpRequest.send() method. It uses the same format a form would use if the encoding type were set to "multipart/form-data".

[Modzilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries) explains the next part of that line.
>"The Object.fromEntries() method takes a list of key-value pairs and returns a new object whose properties are given by those entries.""

Our formData is initialized as this object of the data submitted from the html form. It is ready to go!

The attribute of this object for text is set to equal a modified version of itself. This modified version iterates through any "<" characters and then replaces them with the non-code variant. It does this using the built in replaceALL function. In the next line, the same is done for all ">" symbols. 

The text of the form is now clean of any tags, which the browser would have identified as code to run. Instead, the text attribute will hold a tag-less string that the browser can display as it was typed. Don't worry, user 'I_<3_Cybersecurity'. Your username will still display correctly.



### Method 2: "The Voodoo Method"
The name of this method comes courtesy of my TA, who named it such after he saw it for the first time. It is rather strange, using JavaScript built in functions in a way they were not meant to be used. I will be heavily relying on a tutorial by [RemarkableMark](https://remarkablemark.org/blog/2019/11/29/javascript-sanitize-html/).

We won't bother to use the same formData.text example as we did above. Afterall, strings from forms are not the only elements we might want to sanitize. This method will work for the above example as well, though.

#### Code
```js
let unsanitizedInput = "<(some sort of dangerous xss)>"
let dummyElementTool = document.createElement('div')
dummyElementTool.innerText = unsanitizedInput
let sanitizedInput = dummyElementTool.innerHTML
// now, sanitizedInput = "&lt(some sort of dangerous xss)&gt"
```

First we initialize our unsanitized input. This could be from an html element or a form, etc. Either way, we are storing it as a string here. 

From here, we create a new element. I called it 'dummyElementTool' because its only purpose is to sanitize our tainted string. By setting the innerText of dummyElementTool equal to our tainted string, it replaces tags with our & lt and & gt values. This is because '.innerText' will automatically do this to any string within it. It is not the main purpose of '.innerText', but that is all we will be using it for, hence why my friend called this "the Voodoo method." It takes advantage of just one property, ignoring all others to accomplish something that '.innerText' was not intended to do.

We initialize a new variable (or just redefine the old one if you insist on still using it) and set it equal to dummyElementTool.innerHTML, effectively accessing the code free version of the string. It is now clean!

## Additional Resources
https://remarkablemark.org/blog/2019/11/29/javascript-sanitize-html/
> This website is the original "voodoo method". It does not go into detail as to why it works, but it is a good reference for anyone who wants to see another example of the source code.

https://owasp.org/www-community/attacks/xss/
> This website, also cited earlier, is a really good article on cross site scripting. It is understandable for nearly anyone who is somewhat familiar with javascript and html. It goes into far greater detail about what xss is and does.

https://stackoverflow.com/questions/19030742/difference-between-innertext-innerhtml-and-value
> This is a forum on stack overflow where users have answered someone who asked the difference between .innerHTML and .innerText. This is really useful for anyone who would want to learn about how the "voodoo method" works in more detail

https://www.hacksplaining.com/prevention/xss-stored
> This source gives a detailed report of the different ways XSS can be used (and thus why we need to defend against it). It gives additional ways to defend against xss where the malicious user is targeting another's cookies. 
