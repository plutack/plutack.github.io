---
title: "Solving the Research Rabbit Hole: Building Tools to Stay Focused"
date: 2024-11-21T13:22:21+01:00
draft: false
toc: false
images:
tags: 
  - dev-notes
  - code
  - projects
---


## Overcoming the Research Rabbit Hole

Ever found yourself researching a topic of interest, only to stumble upon something you don’t quite understand? If you’re like me, this often leads to opening another tab to dig deeper. Then, before you know it, you’re lost in a recursive search cycle, only this time, there’s no base case in sight.

As a developer, staying focused during problem-solving is critical. But I kept running into this distraction trap, especially after I started learning Go and grappling with its unique quirks. It was clear I needed a solution to maintain my flow state for as long as possible.

This realization inspired me to create two tools tailored to the environments where I spend most of my time: the browser and the terminal. Let me introduce you to **Gemini-Quick-Prompt** and **Gemini Helper**. Sure, the names are clunky, but they get the job done! Both tools leverage Google’s Gemini model, which I chose primarily because it’s free and comes with an extensive SDK library for seamless integration.


## **Tool #1: Gemini-Quick-Prompt**

This tool is as simple as it gets. The entire program is just 44 lines of Go, and its purpose is straightforward: you provide input via command-line arguments, and it returns context on your query. If your query relates to something that can be formatted like a Unix manpage, it does exactly that.

Why did I build this? Often, while programming, I find that my terminal editor LSP (Language Server Protocol) hints aren’t detailed enough to help me fully understand a function or concept(skill issues, if I am being totally honest lol). Before now, I would open a browser tab and start Googling, but that always risked breaking my focus. With Gemini-Quick-Prompt, I can stay in the terminal, minimizing interruptions and keeping my brain in "programming mode."


## **Tool #2: Gemini Helper**

While Gemini-Quick-Prompt keeps me grounded in the terminal, Gemini Helper is a browser extension designed for Chrome and Mozilla-based browsers. It allows you to crop a portion of your browser screen, process it through a backend, and get context delivered as a pop-up on the bottom-right corner of the screen.

Developing this was challenging but rewarding. It took me back to the beginning; HTML, browser APIs, and more. The **HTML5 Canvas** element became a cornerstone of this project, and through it, I gained deeper insights into:

- Callbacks and promises: Handling asynchronous operations effectively.
- Higher-order functions: Using functions that accept other functions as arguments or return them as results.
- Browser extension architecture: Understanding how content scripts, background scripts, and the manifest.json file work together.
- Cross-browser differences: Adapting the extension for both Chrome and Mozilla-based browsers.# A Practical Use of Higher-Order Functions

One of the most enlightening moments came when I applied higher-order functions practically. Although I understood their theory, I hadn’t truly appreciated their utility in real-world scenarios until I encountered issues with managing variables in my event listeners.

Initially, I defaulted to using global variables to share state across my code. While it worked, it was clunky and error-prone. Then, I realized I could encapsulate state by using a higher-order function, making my code cleaner and more modular.

Here’s an example of what I mean:
Using a Higher-Order Function
```js
function doSomethingWithState(state){
return function doSomething(event){
		// dosomething with state 
		console.log('State:', state, 'Event:', event);
	}
}

globalThis.addeventListener("mousedown", doSomethingWithState(state))
```

instead of 
```js
let state = "a particular state"
function doSomething(event){
		// dosomething with state as a global variable
		console.log('State:', state, 'Event:', event);
	}
	
}

globalThis.addeventListener("mousedown", doSomething)
```

In the first example, the `state` variable is encapsulated within the higher-order function, keeping the global namespace clean and avoiding potential conflicts. This approach also makes it easier to reason about and test the code, as the dependency (the state) is explicitly passed and not hidden in a global scope.

This practical application of higher-order functions helped me appreciate their power in scenarios where state management and modularity are crucial.

These weren’t just technical lessons; they were eye-opening experiences that deepened my understanding of modern web development.


## The Results and What’s Next

Both tools have significantly improved my workflow. For instance, instead of opening Google or ChatGPT, copying and pasting a query, and waiting for results, I can now highlight text or capture screen content to get immediate insights. This reduced overhead has been a game-changer.

Looking ahead, I plan to enhance Gemini Helper by adding a **text selection mode**. With this feature, you could simply highlight text on a web page and get context without cropping the screen. I believe this addition will make the tool even more versatile and further streamline my research process.

---

## Closing Thoughts

Building these tools wasn’t just about solving a problem; it was about learning and growing as a developer. From re-learning old concepts to integrating modern APIs, the journey has been invaluable. I’m excited to continue refining these tools and exploring how they can help others stay in their flow states too.

### Resources: 
- [Gemini quick prompt github repo](https://github.com/plutack/gemini-quick-prompt)
- [Gemini helper github repo](https://github.com/plutack/gemini-helper)
