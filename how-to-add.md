---
title: How to Add Q&A
nav_order: 13
description: Templates and steps for adding questions, topics and sections
---

# How to Add Q&A
{: .no_toc }

Everything you need to extend this handbook.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>On this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## 1. Add a question to an existing page

Open the topic page (for example `core-java/collections-framework.md`) and paste this block **above the HTML comment at the bottom**:

~~~markdown
## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use lists, tables and code blocks.

```java
// code example
```

**Key points**
- Point one
- Point two

**Follow-up questions**
- Follow-up one

---
~~~

| Level | Label line |
|:--|:--|
| Easy | `{: .label .label-green }` |
| Medium | `{: .label .label-yellow }` |
| Hard | `{: .label .label-red }` |

Callouts you can add inside an answer:

~~~markdown
{: .tip }
A short interview tip.

{: .trap }
A common mistake to avoid.
~~~

## 2. Add a new topic page to a section

Create a file such as `spring-boot/caching.md`:

~~~markdown
---
title: Caching
parent: Spring Boot
nav_order: 9
---

# Caching
{: .no_toc }

One-line description of this topic.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## First question?

Easy
{: .label .label-green }

**Answer**

Your answer.

---
~~~

Rules:
- `parent` must match the section title **exactly**, for example `Spring Boot`.
- `title` must be unique across the whole site.
- `nav_order` controls the order inside the section.

## 3. Add a new section

1. Create a folder, for example `testing/`.
2. Add `testing/index.md`:

   ~~~markdown
   ---
   title: Testing
   nav_order: 13
   has_children: true
   permalink: /testing/
   ---

   # Testing
   {: .no_toc }

   Description of the section.
   {: .fs-6 .fw-300 }
   ~~~

3. Add topic pages inside the folder with `parent: Testing`.

Ready-made copies of these templates are in the `_templates/` folder of the repository.

## 4. Preview locally

```bash
bundle exec jekyll serve --livereload
```

Open http://localhost:4000/tech-interview-handbook/
