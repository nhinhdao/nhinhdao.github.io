---
layout: post
title:      "Handle Asynchronous Functions"
date:       2019-03-26 17:58:16 -0400
permalink:  handle_asynchronous_functions
# header-img: "https://i.imgur.com/rnqtsyy.png"
---

![Sync and Async Task](https://i.imgur.com/rnqtsyy.png)
> Sync vs Async (credit: webi.io)

Before working with **asynchronous functions** (short hand **async functions**), we should know about **synchronous functions** or regular execution flow of a program.
 

In programming, **synchronous** code can be understood as executions follow sequence flow. Which means each statement in the code is executed **one after the other**. This means each statement will wait for the previous one to finish executing.

```
console.log('First');
console.log('Second');
console.log('Third');
```

The code above will execute in one after one flow, outputting **“First”, “Second”, “Third”** to the console. That’s an example of function call synchronously.

On the other hand, **Asynchronous** code bring its execution outside of the program flow, allowing the code after that line to be executed immediately without waiting. **fetch** in Javascript or **setState** in React are examples of this function call:

```
console.log('First');
fetch(example_url).then(() => console.log('Second'));
console.log('Third');
```

In the example above, the output will be: **“First”, “Third”, “Second”**. This is because the function passed into **fetch** is not called immediately – it has to wait for the **fetch** function to **finish getting data** from the other website before it get called. While the fetching function is on excecution, the next line: `console.log('Third')` is called without waiting for the fetch to be done, output **Third** to the console. Then when the fetch function finishs it work, the function `() => console.log('Second') `now then be called and output **Second** to the console.

or check out this setState example below:


```
state = {count: 1};
		 
changeCount = () => {
		 console.log('Count before: ', this.state);
		 this.setState({count: 2});
		console.log('Count after: ', this.state)
}
```

Guess what is printed out in the console: 

```
Count before: 1
Count after: 1
```

How come? Well, **setState** in React is an **async function**, so it won't follow the regular excecution flow. while this.setState is called and spend it time to execute the statement, the next line `console.log('Count after: ', this.state)` is excetuted and print out the previous state that is Count: 1.

So with these **async functions** like in above examples, what can we do to have it render what we intended it to render.

One of the simple solutions is you can wait for the task to be done before calling the next line of code:


```
state = {count: 1};
		 
changeCount = () => {
		console.log('Count before: ', this.state);
		this.setState({count: 2}, () => console.log('Count after: ', this.state))
}
```


Better solution, meet **ES2017/ES8** [ASYNC FUNCTIONS](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).


```
state = {count: 1};

increaseStateCount = () => {
  this.setState({count: 2})
}

async callNow(){
  console.log('Count before: ', this.state);
  await this.increaseStateCount();
  console.log('Count after: ', this.state)
}
```

Both above solutions will output your desired result:

```
Count before: 1
Count after: 2
```


A quick fix for waiting problem is that you should render some prepared content while users are waiting for the main content to be loaded.

Think about the case where your users can click on a button and a list of kitties will be shown on the screen. Behind the scene, you need to load hundreds of cat images from an API endpoint  using **fetch** (or jQuery Get request, axios get...), what should you do so that your users don't have to look at a blank while page while waiting for the data to be fetched:

```
state = {catImages: []};

//click on View cats button will bring you to this function:
getCatImages = () = {
   fetch(catPicsUrl).
      then(resp => resp.json()).
	    then(images => this.setState({catImages: images});
}

render(){

   //Render a prepared content because at this moment, catImages is still an empty array
   if(this.state.catImages.length === 0){
	    return (
			<LoadingComponent />
			)
	  }
		return (
      this.state.catImages.map(cat => {
      <li key={cat.id}><img src={cat.image} alt={cat.name} /></li>
    }
		
)
```

![Loading screen](https://i.imgur.com/FH3pKLX.gif)
> Cat Loading Gif (credit: Imgur - TantalizingBanana)

Then when your catImages array is filled with new content, you can now display it on your pages:

![Kitties List](https://i.imgur.com/rEfFgbs.jpg)
> Cats (credit: MSN.com)

Thanks for reading!


