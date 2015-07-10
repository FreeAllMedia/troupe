# Troupe.js

<!--TOC-->

## Introduction

Troupe is an agile framework that assists in writing applications that are isolated from the libraries that power them.

As opposed to monolithic framework architecture, Troupe keeps application and library logic isolated from each other to take advantage of powerful benefits such as the ability to:

* Swap libraries without any changes to application logic.
* Utilize libraries from any language seamlessly.
* Take organizational ownership of library interfaces, without taking responsibility over their underlying functionality.

## Methodology

Troupe is mostly held together by a methodology that helps its users make informed decisions, as opposed to more rigid frameworks that make decisions for its users.

For this reason, it's important to read through the following sections to understand why Troupe is designed as it is before jumping right in to development.

### Isolating Applications from Libraries

Traditionally applications are written with direct calls to libraries, like in this simple example:

``` javascript
// app.js

import PersonEngine from "person-engine";
import BikeLib from "bikelib";
import Streetso from "streetso";

const person = new PersonEngine();
const bike = new BikeLib();
const street = new Streetso();

bike.add(person);

street.addEntity(bike);
```

Direct calls certainly work, but when the library changes in any significant way, applications depending on the library will also require changes just to maintain existing functionality:

``` javascript
// app.js

import Personi from "personi";
import Biketastic from "biketastic";
import Streetorama from "streetorama";

const person = new Personi();
const bike = new Biketastic();
const street = new Streetorama();

bike.addPerson(person);

street.addVehicle(bike);
```

Troupe solves this problem by wrapping all libraries into containers called `components`:

``` javascript
// bike-component-bikelib.js

import Bike from "bikelib";

const privateData = new WeakMap();

export default class BikeComponent {
  constructor() {
    privateData.set(this, {
      bike: new Bike()
    });
  }

  addPassenger(passenger) {
    const bike = privateData.get(this).bike;

    bike.add(passenger);
  }
}
```

This technique allows us to change the inside of components, without disturbing applications:

``` javascript
// bike-component-biketastic.js

import Bike from "biketastic";

const privateData = new WeakMap();

export default class BikeComponent {
  constructor() {
    privateData.set(this, {
      bike: new Bike()
    });
  }

  addPassenger(passenger) {
    const bike = privateData.get(this).bike;

    bike.addPerson(passenger);
  }
}
```

In turn, this allows the application to remain unchanged when swapping libraries:

``` javascript
// app-bikelib.js

import Person from "person-component";
import Bike from "bike-component-bikelib";
import Street from "street-component";

const person = new Person();
const bike = new Bike();
const street = new Street();

bike.addPassenger(person);

street.addEntity(bike);
```

``` javascript
// app-biketastic.js

import Person from "person-component";
import Bike from "bike-component-biketastic";
import Street from "street-component";

const person = new Person();
const bike = new Bike();
const street = new Street();

bike.addPassenger(person);

street.addEntity(bike);
```

### Interface Ownership

Libraries can vary wildly in interface style. Some may use callback, while others use promises. Some may have a chained interface. Some may put arguments in a certain order, while others are in a different order.

Traditionally, these interfaces are controlled by their respective library owners, and if we want a change them, we would need to convince the library owners to make the change.

By encapsulating libraries into components, organizations are given an opportunity to take ownership of library interfaces, without altering anything about the library itself.

This is different from forking a library because you continue to receive patches and feature updates from library owners as usual, but now have the ability to change the library's interface at will to match your organization's preferred coding style.

**For example**, the following `person-library` has a few ways of calling its methods such as:

* `person.position.xyz`
* `person.walkForward`
* `person.walkBackward`
* `person.voice.speak`

What if we want to normalize its interface to become...

* `person.position`
* `person.walk`
* `person.talk`

...yet we don't want to alter or take ownership of the library itself?

``` javascript
// person-component.js

import PersonLibrary from "person-library";

const privateData = new WeakMap();

class PersonComponent {
  constructor(attributes) {
    privateData.set(this,{
      person: new PersonLibrary(attributes)
    });

    this.attributes = person.attributes;
    this.position = person.position.xyz;
  }

  walk(distance, direction="forward") {
    const person = privateData.get(this).person;
    switch (direction) {
      case "forward":
        person.walkForward(distance);
      case "backward":
        person.walkBackward(distance);
    }
  }

  talk(words) {
    const person = privateData.get(this).person;
    person.voice.speak(words);
  }
}
```

### Component Testing

Sometimes, libraries are poorly tested, under tested, or not tested at all. Other times, libraries can show 100% passing test coverage, but have methodological errors in how they are tested.

Independently unit testing components doesn't just verify that your interface is working properly *now*; it also verifies that all of your applications will *continue* operating as normal even if you completely swap out libraries or write your own custom solutions.

For example, here's what independent test coverage of the aforementioned `person-component` might look like (depending on your choice of testing system):

``` javascript
import sinon from "sinon";
import PersonComponent from "person-component";

describe("PersonComponent(attributes)", () => {
  let attributes,
      person;

  beforeEach(() => {
    attributes = {
      name: "Bob Belcher"
    };

    person = new PersonComponent(attributes);
  });

  describe(".attributes", () => {
    it("should equal the attributes passed to the constructor", () => {
      person.attributes.should.eql(attributes);
    });
  });

  describe(".position", () => {
    it("should equal [0, 0, 0] by default", () => {
      person.position.should.eql([0, 0, 0]);
    });
  });

  describe(".walk(distance, direction)", () => {
    it("should increase x position by the supplied distance when direction is 'forward'", () => {
      person.walk(10, "forward");
      person.position.should.eql([10, 0, 0]);
    });

    it("should decrease x position by the supplied distance when direction is 'backward'", () => {
      person.walk(5, "backward");
      person.position.should.eql([-5, 0, 0]);
    });
  });

  describe(".talk(words)", () => {
    it("should print words to STDOUT", () => {
      const write = sinon.spy(process.stdout, "write");
      const words = "Hello World!";
      person.talk(words)
      write.calledWith(words).should.be.true;
      process.stdout.restore();
    });
  });
});
```

#### Continuous Integration

#### Continuous Quality Auditing

### Component Consumability

### Component Design Contract



Troupe leaves the design of components in your hands, so it is important to create an organization-wide `Component Design Contract` to dictate exactly how your components are to be built.





## Components

## Generators
