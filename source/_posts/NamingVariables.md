---
title: Guideline on Naming Variables
tags:
   - Code
   - Guideline
---

> “There are only two hard things in computer science: cache invalidation and naming things.” — Phil Karlson


## Follow S-I-D

A name must be Short, Intuitive and Descriptive.

```javascript
/* Bad */
const a = 5 // "a" could mean anything
const isPaginatable = (postsCount > 10) // "Paginatable" sounds extremely unnatural
const shouldPaginatize = (postsCount > 10) // Made up verbs are so much fun!
```

**Suggested:**

```javascript
/* Good */
const postsCount = 5
const hasPagination = (postsCount > 10)
const shouldDisplayPagination = (postsCount > 10) // alternatively
```

## Avoid Contraction

Do not use contractions. They contribute to nothing but decreased readability of the code. Finding a short, descriptive name may be hard, but contraction is not an excuse for not doing so. For instance

```javascript
/* Bad */
const onItmClk = () => {}
/* Good */
const onItemClick = () => {}
```

## Avoid Context Duplication

Always remove the context from a name if that doesn’t decrease its readability.

```javascript
class MenuItem {
  /* Method name duplicates the context (which is "MenuItem") */
  handleMenuItemClick = (event) => { ... }
  
  /* Reads nicely as `MenuItem.handleClick()` */
  handleClick = (event) => { ... }
}
```

## Should Reflect expected result

```javascript
/* Bad */
const isEnabled = (itemsCount > 3)
return <Button disabled={!isEnabled} />

/* Good */
const isDisabled = (itemsCount <= 3)
return <Button disabled={isDisabled} />
```

## Consider Singular/Plurals

Like a prefix, variable names can be made singular or plural depending on whether they hold a single value or multiple values.

```javascript
/* Bad */
const friends = 'Bob';
const friend = ['Bob', 'Tony', 'Tanya'];
/* Good */
const friend = 'Bob';
const friends = ['Bob', 'Tony', 'Tanya'];
```

## Use meaningful and pronounceable variable names

```javascript
const yyyymmdstr = moment().format("YYYY/MM/DD");  // simply awful
// Instead
const currentDate = moment().format("YYYY/MM/DD");
```

The [original article](https://betterprogramming.pub/a-useful-framework-for-naming-your-classes-functions-and-variables-e7d186e3189f) is more complete and has other suggestion regarding naming good practices would suggest you check it.

## Reference

* <https://betterprogramming.pub/a-useful-framework-for-naming-your-classes-functions-and-variables-e7d186e3189f>