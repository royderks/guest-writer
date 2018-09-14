## Double Checking Technical Instructions

While writing technical articles, one of the most common mistakes that authors make is to forget some technical instructions. For example, imagine that you are writing an article that teaches readers how to develop RESTful APIs with Node.js and Express. If you forget to mention readers to install an NPM module that is important, they won't be able to achieve the expected result (in this case, they won't have a runnable Express app).

To avoid this popular mistake, what you **must do** is to double check the instructions that you wrote **after** you finish the first draft.

You might be thinking: "Really? This is a lot of work!" Well, **it is not**.

Double checking your technical instructions is quite easy. You have to act like a new reader who just started reading your article would act. The **biggest** difference is that you **don't have to read** the content that is not a technical instruction per se. You have to focus on what is technical.

For example, you have to:

- Create directories as you instruct on the article (you do instruct them to create the directories, right?).
- Create the files as you instruct on the article (again, make sure you mention all files).
- Execute any command that is needed (e.g., `mkdir my-new-directory`, `touch what-ever-file`, `npm install express`, etc).
- Copy the content of the code snippets and paste them on the correct files.

One thing to keep in mind is that you **must** do this in a new environment. That is, you have to **recreate from scratch** your sample **just by** following the instructions on the article.

If, in the end of the article, you end up with a running project, then you are good to go. If you end up with an app that outputs errors and that do not work, then you know you have gaps in your article.

Another thing that you have to keep in mind is that you cannot "cheat". That is, you cannot execute commands from the top of your head that are not clearly instructed in your article. Also, you cannot type (or copy) any code that is not presented in your article. This is what would be considered "cheating" in this case.

> Just as an example, I doubled checked the instructions on [this (huge) article](https://auth0.com/blog/react-tutorial-building-and-securing-your-first-app/) in less than 20 minutes.

By the way, things you **don't** have to do:

- Create new accounts on services that you use (e.g., Auth0, Firebase, mLab, Heroku, etc).
- Configure new elements on the services that you use (e.g., a new Auth0 Application, or a new database on mLab, etc).

For those, you can simply use the existing ones. The goal here is to double check if the article contains everything related to the code.

Have fun!