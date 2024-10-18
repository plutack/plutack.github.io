---
title: "Creating a Simple Email Sender App"
date: 2024-10-18T09:51:18+01:00
draft: false
toc: false
images:
tags: 
  - code
  - projects
  - Go
  - Wails
  - Cross-platform
---

## A Custom Email Solution for a Client: From Mailing Lists to Wails
I was recently approached by a client looking to send the same email to multiple recipients. My first thought was to set up a mailing list, which seemed like the obvious solution. However, the client insisted this would not be a frequent exercise, with only around 50 intended recipients. Since my client was not very technically inclined, setting up a mailing list was quickly ruled out. Not only would it involve a recurring cost (even when not in use), but it might also be too complex for the client to manage.

## Exploring Alternatives: From Scripting to Hosting

As a JavaScript developer, I initially thought it wouldn’t be difficult to solve this problem with a custom script. In the past, I had worked on a project where I sent customized emails based on user data. This solution even involved sending unique messages via the Gemini API. However, this approach quickly became impractical because it would require the client to have basic knowledge of running scripts from a terminal, which was unrealistic.

I then considered hosting the solution on the web. While that seemed like a workable approach, it added unnecessary complexity. I’d have to deal with setting up a domain and finding a free or paid hosting service. Another thought was to spin up a Docker container that auto-started on the client’s system whenever it was turned on. But again, Docker would introduce extra abstractions and complications for someone who wasn’t technically savvy.

## The Move to Compiled Languages and GUI Considerations

Next, I thought about using a compiled language to create a single binary file that the client could run easily. However, this plan originally involved building a Command-Line Interface (CLI), which wouldn’t work for this particular user. A Terminal User Interface (TUI) was clearly not the right solution either. I then turned my attention to building a Graphical User Interface (GUI).

One of the first things I built during my coding journey was a Python GUI with Tkinter. While it functioned, the overall experience and the interface design left a lot to be desired—likely due to my limited experience at the time. I wanted to find a balance between creating a user-friendly tool and keeping the development process simple for myself.

## Enter Wails: A Balanced Solution

In my search, I came across Wails, which seemed like the perfect solution. Wails allows you to use JavaScript to design the frontend while writing the backend functionality in Go. It handles the necessary bindings between the frontend and backend for smooth interaction. Since I was already learning Go and was very familiar with JavaScript, this seemed like an excellent fit. Creating a simple frontend to handle data input would be easy, and it gave me an opportunity to refine my Go skills.

The benefits didn’t stop there. Unlike my experience with Python GUIs, I could now create a polished, simple-looking interface. Compared to Electron-based GUIs (which bundle Chromium or V8 and result in huge executables), Wails would give me a lightweight solution. A Wails app ships with the Go runtime, ensuring a single, small executable—much smaller than an Electron app, which was important for such a simple task.

## How the Application Works

The solution I developed uses the `gomail.v2` package with SMTP over TLS to send emails securely. On the frontend, the user inputs their email address and an app-specific password, which they generate after enabling two-factor authentication (2FA). This ensures secure communication with their email provider.

For the message body, the user can either type it directly into the application or import it from a `.txt` file. Similarly, recipient details can be entered manually or imported from a `.csv` or `.xlsx` file. The frontend ensures the validity of these files, checking for necessary fields like `firstname`, `lastname`, and `email address`. For each recipient, the email will be personalized with a greeting like: **"Dear Firstname Lastname"**, making the emails feel more personal.

### Real-Time Feedback and Logging

A logging section is built into the application to provide the user with real-time progress updates. Logs from the backend are sent to the frontend and displayed in a"show logs screen" with key milestone events highlighted using a ShadCN toast component. This ensures the user is informed about the progress of the email-sending process and any issues that may arise, offering a user-friendly experience with clear feedback.

#### Application Interface Overview

To give you a clearer picture, here's a visual representation of the application interface:
{{< figure src="images/email-sender.png" alt="image" caption="Email Sender UI" class="big" >}}
## Final Product and Cross-Platform Support
The end result was a lightweight 10MB Windows executable that the client could easily run without any further guidance. One of the key features was the ability to compile the same codebase for both Linux and macOS, ensuring flexibility for future needs.

By using Wails and Go, the compilation process was straightforward and well-supported for cross-platform builds, allowing for minimal overhead in delivering a truly universal solution. This also meant that as the project scaled or the client’s requirements evolved, they wouldn’t be tied to a single operating system—an important advantage for businesses today.

### Resources: 
- [Simple nodeJS birthday reminder app using Gemini API](https://github.com/plutack/Birthday-Reminder)
- [Email sender github repo](https://github.com/plutack/email-sender)
- [Creator of Wails video: Building Desktop Applications using Wails](https://www.youtube.com/watch?v=13Ufa9i8cFo&t=115s)
- [Wails documentation](https://wails.io/docs/introduction)
