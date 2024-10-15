<!-- This file is the homepage of your Log4brains knowledge base. You are free to edit it as you want -->

# Architecture knowledge base

Welcome 👋 to the architecture knowledge base of hostr.
You will find here all the Architecture Decision Records (ADR) of the project.

## Definition and purpose

> An Architectural Decision (AD) is a software design choice that addresses a functional or non-functional requirement that is architecturally significant.
> An Architectural Decision Record (ADR) captures a single AD, such as often done when writing personal notes or meeting minutes; the collection of ADRs created and maintained in a project constitutes its decision log.

An ADR is immutable: only its status can change (i.e., become deprecated or superseded). That way, you can become familiar with the whole project history just by reading its decision log in chronological order.
Moreover, maintaining this documentation aims at:

- 🚀 Improving and speeding up the onboarding of a new team member
- 🔭 Avoiding blind acceptance/reversal of a past decision (cf [Michael Nygard's famous article on ADRs](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html))
- 🤝 Formalizing the decision process of the team

## Usage

This website is automatically updated after a change on the `master` branch of the project's Git repository.
In fact, the developers manage this documentation directly with markdown files located next to their code, so it is more convenient for them to keep it up-to-date.
You can browse the ADRs by using the left menu or the search bar.

The typical workflow of an ADR is the following:

![ADR workflow](/l4b-static/adr-workflow.png)

The decision process is entirely collaborative and backed by pull requests.

## Adding ADRs 

You can use the `docker` image to add a new ADR in package `Documentation-tools':

```bash
docker run --rm -ti -v $(pwd):/workdir -p 4004:4004 thomvaill/log4brains adr new -p Documentation-tools
```

## Generate the ADR site

Generate the ADR documentation site with:

```bash
mkdir -p ./adr/site/log4brains/
docker run --rm -ti -v $(pwd):/workdir -p 4004:4004 thomvaill/log4brains build --out ./adr/site
```

You need to add the `--out ./docs/site/log4brains` option as we want to include the generated `ADR` site *into* the `mkdocs` generated site (the `ADR` site is published under .../docs/site/log4brains).  

## More information

- [Log4brains documentation](https://github.com/thomvaill/log4brains/tree/master#readme)
- [What is an ADR and why should you use them](https://github.com/thomvaill/log4brains/tree/master#-what-is-an-adr-and-why-should-you-use-them)
- [ADR GitHub organization](https://adr.github.io/)
