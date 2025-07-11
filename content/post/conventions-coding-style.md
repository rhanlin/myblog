+++
title = "開發慣例: Coding style"
description = ""
date = 2025-01-27 20:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

## Singleton 的寫法

```javascript
class AnimalManager {
  constructor() {
    this.init()
  }

  init() {
    this.name = 'AnimalManager'
  }
}

export default new AnimalManager()

class AnimalManager {
  constructor() {
    if (Animal.instance) {
      return Animal.instance
    }

    this.init()
    Animal.instance = this
  }

  init() {
    this.name = 'AnimalManager'
  }
}

export default AnimalManager


// typescript 可以用靜態工廠模式 (Static Factory Pattern) 
class AnimalManager {
    private static instance: AnimalManager;
    
    public static getInstance(): AnimalManager {
        if (!AnimalManager.instance) {
            AnimalManager.instance = new AnimalManager();
        }
        return AnimalManager.instance;
    }
}

```

## if 的寫法

```javascript
// Good
if (dog) {
  doSomething()
  create(dog)
}

// Good
if (dog) create(dog)

// Bad
dog && create(dog)
```

## get/set in class

```javascript
// Bad
class Dog {
  #dogname
  
  constructor() {
    this.#dogname = 'Droopy'
  }

  get dogName() {
    return this.#dogname
  }

  set dogName(name) {
    // Not doing anything else. 
    this.#dogname = name
  }
}

// Good
class Dog {  
  constructor() {
    this.dogname = 'Droopy'
  }
}
```