# MoTionPatternsBook
Don't know how to use MoTion? Complicated DSL? No worries!
We have a solution for you. A way to benefit from AI so it will create patterns for you.
This is a book that helps you to create MoTion patterns using AI;

## MoTion
If you want the AI to know how to use MoTion you can refer to [this page](https://github.com/alesshosry/MoTionPatternsBook/blob/main/MoTion.md).

## FASTTypeScript(TypeScript AST) + MoTion
If you want to let the AI create patterns that match TypeScript AST using MoTion, you can ask it to refer to [this page](https://github.com/alesshosry/MoTionPatternsBook/blob/main/FASTTypeScript-MoTion.md).

# Usage

For the moment, you can open any AI agent, provide it with the md files (better for the moment to upload files) and paste something like this message:

_Ok i Will give you 2 files to read: one that describe MoTion, which allows you to create a pattern to do pattern matching in Pharo over models. And another one that describes how to use MoTion with FASTTypeScript, which is a metamodel that allows you to represent the AST of TypeScript in Pharo. Inside this documentation, all FASTTypeScript classes that represent TypeScript entities are listed.
Given these two docuemntations, I want you to create an example of Typescript, that contains 3 methods in a class: one with switch case, one with if else, and one with other statements.
Then you create a pattern in MoTion to match the parsed typescript code that contain switch case_

This example was tested on Mistral AI (Without License), ChatGPT(Without license) and Copilot (with License). All three agents were able to generate the example and the pattern correctly excpet for one, with mini mini error in a property name.

For the future:
- I will try to benefit from AI skills for markdown files which allow you to use directly the link in AI agent instead of uploading files manually
- I will try to adapt it to be used in Pharo directly ... we are ambitious but will give it a try :)
- More documentations will be added for other FAST metamodels, and why not Famix also :)


Feel free to contribute :) 
