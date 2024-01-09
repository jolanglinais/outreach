# ChatGPT Prompt Framework

![ChatGPT Prompt Framework Logo Header Image][headerimg]

This prompt engineering framework for ChatGPT acts as a guide for providing a structured set of instructions to the AI model in order to receive fuller, more accurate, and more desirable results.

I call this **SPICIER** because it is the best anagram I could think of, but it doesn't perfectly match the order of categories.

# S.P.I.C.I.E.R.

ℹ️ _This, especially **Input**, can and should be written for a computer, not a human._

1. **Role**: Give ChatGPT a persona to give a response through the lens of
2. **Instruction**: What you want to get out of ChatGPT
3. **Context**: Help ChatGPT understand the context
4. **Input**: The input data you want ChatGPT to respond to
5. **Source**: Reference websites to help shape the input to results
6. **Exclusions**: Further define parameters by omitting undesired results
7. **Parameters**: Specify specific options or constraints, *i.e. the tone of voice, formatting, length*

## Example

> ROLE:
> 
> Act as a software engineer expert in the following:
> 
> - TypeScript
> - Node.js
> - Koa
> - Prisma
> - Zod
> 
> INSTRUCTIONS:
> 
> - I will give you my code for a specific Koa middleware and the related Prisma schema
> - You will change the code to do the following:
>     - Check if `something123` exists.
>     - If it does, send a patch to X API and ignore both Prisma and the Airtable response
>     - If it does not, send a post to Y API and create the record in Prisma
> 
> CONTEXT:
> 
> - The `submit` Koa middleware to handles form submissions
> - The following are arbitrary unique identifier variables:
>     - `something123`
>     - `something456`
> 
> CODE:
> 
> ```
> INSERT CODE
> ```
> 
> EXCLUSIONS:
> 
> - Assume my web application code is working and is setup correctly

[headerimg]: ../images/gpt.png
