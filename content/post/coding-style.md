Singleton 的寫法
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
```

## if 的寫法，Whitch is better?
```javascript
// a.
if (dog) {
  create(dog)
}

// b.
if (dog) create(dog)

// c.
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