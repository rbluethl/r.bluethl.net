## We went to Ecuador to build an App

## A Short Backstory

A good friend of mine, [Norbert](https://twitter.com/TheHuethman), moved to the United States a few years ago. Since I still live in Austria, we don't see each other very often. For that reason, we've made it a habit to meet once a year - *somewhere* in the world. This summer, we went to Ecuador. And quite accidentally, we built an app together...

Norbert and I met somewhere around 2011-2012 at a company where I was doing my apprenticeship at that time. We've been software engineers for all our careers and over the years, we've worked on many projects together - most of which didn't end up anywhere. 

## Ecuador 2021

In the meantime, we both work as freelance software engineers - giving us the flexibility to work from anywhere at any time. When we flew to Ecuador this year, we were actually planning on working on our ongoing projects during the week, having dinner/drinks at night and visiting some cool places in our spare time. Well, things turned out a bit differently.

Instead of going out for drinks, we decided to do an entire rewrite of [AutoForward SMS](https://autoforwardsms.com/), an app that Norbert acquired on [Flippa](https://flippa.com/) earlier this year. (If you're interested in the full story, you should definitely [read this post on Indie Hackers](https://www.indiehackers.com/post/how-i-acquired-a-mobile-app-on-flippa-and-grew-it-from-700-mrr-to-5-4k-d50b9a8e82)). 

The app's functionality? Forwarding text messages (SMS) to an E-Mail or URL based on a user defined ruleset. Norbert had told me about the acquisition and his intention to rewrite the app before, but there were no concrete plans until our get-together in Quito. 

After several conversations, we spontaneously decided to partner up and start rebuilding AutoForward SMS immediately while we were together. So while we were putting in the ours at our client projects, we were diligently drafting, designing and developing the new app every free minute.

## The App

### Why Rewrite?

When Norbert took over AutoForward SMS in early 2021, it became clear very quickly that the tech stack was fairly outdated, the code base was in poor condition and the UI/UX could be largely improved. Also, the app should become more scalable, reliable and generally be prepared for future growth - something that couldn't be done very easily.

### The New Tech Stack's Premise

For the reasons mentioned above, the new tech stack needed to meet some criteria:

* Cloud-native, serverless architecture
* Cross-platform mobile app framework
* Minimalistic admin UI to manage users/subscriptions
* Generally lightweight and straightforward technology, no overhead

Considering our skillsets and existing knowledge, we then went with the following technologies:

* **Firebase Cloud Functions** — Backend
* **Firebase Firestore** — Database
* **Flutter** — Mobile App
* **Vue & Tailwind** — Admin UI
* **WordPress** — Website

## The Result

We dedicated our entire workcation pushing the app forward and we actually managed to develop a large chunk of the entire new product (App, Backend, Admin UI, Website) in less than two weeks. When we parted ways at the end of August, we were 80% done and decided to continue working on the remaining 20% remotely. 

And so far, everything worked out really well. By the time of writing this blog post, we have already deployed and rolled out the new product to our customers. Our new tech stack is rock-solid and just a joy to work with, both in terms of simplicity and stability.

We challenged ourselves to develop a solid product in a really short time period whilst working on other time-consuming projects, which turned out to work really well. When you commit yourself to a fun project with a good friend, there are literally no limits.

## What's Next?

Now that we have successfully made the transition to the new product, we have a lot of work to do at marketing the product, bringing it into the Google Play Store and generally polishing and improving the App. Our target is to attract new customers and making AutoForward SMS the #1 solution for forwarding text messages.

If you enjoyed this post, you can [follow my journey on Twitter](https://twitter.com/rbluethl).