# Computed Properties

```javascript

fullName: function(){
     return this.get(“firstName”) + “ “ + this.get(“lastName”)
}.property(‘firstName’, ‘lastName’)

```

---

# Big Computed Properties

```javascript

# The currently signed in user can kick the can if they have either of their feet, 
# if they are in the same area as a can, and if the cop is not a problem.
canKickCan: function(){
     return (this.get(‘currentlySignedInUser.hasLeftFoot’) || this.get(‘currentlySignedInUser.hasRightFoot’)) && 
          (this.get(‘currentlySignedInUser.area.cans.length’) > 0) && 
          !((this.get(‘cop.attention’) > 5) && (this.get(‘cop.area’) == this.get(‘currentlySignedInUser.area’)))
}.property(‘currentlySignedInUser.hasLeftFoot’, ‘currentlySignedInUser.hasRightFoot’, 
  ‘currentlySignedInUser.area’, ‘currentlySignedInUser.area.cans.length’, ‘cop.area’, ‘cop.attention’)
```

---

# Big Wrong Comments

```javascript
# The currently signed in user can kick the can if they have either of their feet, 
# if they are in the same area as a can, and if the cop is not a problem.
canKickCan: function(){
     return (this.get(‘currentlySignedInUser.hasLeftFoot’) && this.get(‘currentlySignedInUser.hasRightFoot’)) && 
          (this.get(‘currentlySignedInUser.area.cans.length’) > 0) && 
          !((this.get(‘cop.attention’) > 5) && (this.get(‘cop.area’) == this.get(‘currentlySignedInUser.area’)))
}.property(‘currentlySignedInUser.hasLeftFoot’, ‘currentlySignedInUser.hasRightFoot’, 
  ‘currentlySignedInUser.area’, ‘currentlySignedInUser.area.cans.length’, ‘cop.area’, ‘cop.attention’)
```

---

## Goals

- Easy to Understand
- Easy to Edit
- Short
- No comments necessary

---

# No Comments
### For individual functions

---
 
1. Break into logical sub-properties
2. Use Ember.computed to make it beautiful

---

Instead of this

```javascript
  canKickCan: function(){
    return (this.get(‘currentlySignedInUser.hasLeftFoot’) || 
      this.get(‘currentlySignedInUser.hasRightFoot’)) &&
      ...
  }.property('currentlySignedInUser.hasLeftFoot', 'currentlySignedInUser.hasRightFoot', ...)
```

do this

```javascript
  user: function(){
    return this.get('currentlySignedInUser')
  }.property('currentlySignedInUser'),

  userHasAFoot: function(){
    return this.get('user.hasLeftFoot') || this.get("user.hasRightFoot")
  }.property('user.hasLeftFoot', 'user.hasRightFoot'),

  canKickCan: function(){
    return this.get('userHasAFoot') &&
      ...
  }.property('userHasAFoot', ...)
```

---

More functions

```javascript
  userCanSeeCan: function(){
    return this.get('user.area.cans.length') > 0
  }.property('user.area.cans.length'),

  copPayingAttention: function(){
    return this.get('cop.attention') > 5
  }.property('cop.attention'),

  copInUserArea: function(){
    return this.get(‘cop.area’) == this.get(‘currentlySignedInUser.area’))
  }.property('cop.area', 'user.area'),

  copNotAProblem: function(){
    return !(this.get('copPayingAttention') && this.get('copInUserArea'))
  }.property('copPayingAttention', 'copInUserArea')
```

---

More understanding

```javascript
  canKickCan: function(){
    return this.get('userHasAFoot') && this.get('userCanSeeCan') && this.get('copNotAProblem')
  }.property('userHasAFoot', 'userCanSeeCan', 'copNotAProblem')
```

---

Way More Understanding

```javascript
  canKickCan: function(){
    return this.get('userHasAFoot') && this.get('userCanSeeCan') && this.get('copNotAProblem')
  }.property('userHasAFoot', 'userCanSeeCan', 'copNotAProblem')
```

vs

```javascript
canKickCan: function(){
     return (this.get(‘currentlySignedInUser.hasLeftFoot’) && this.get(‘currentlySignedInUser.hasRightFoot’)) && 
          (this.get(‘currentlySignedInUser.area.cans.length’) > 0) && 
          !((this.get(‘cop.attention’) > 5) && (this.get(‘cop.area’) == this.get(‘currentlySignedInUser.area’)))
}.property(‘currentlySignedInUser.hasLeftFoot’, ‘currentlySignedInUser.hasRightFoot’, 
  ‘currentlySignedInUser.area’, ‘currentlySignedInUser.area.cans.length’, ‘cop.area’, ‘cop.attention’)
```

---

But we created boilerplate

```javascript
  canKickCan: function(){
    return this.get('userHasAFoot') && this.get('userCanSeeCan') && this.get('copNotAProblem')
  }.property('userHasAFoot', 'userCanSeeCan', 'copNotAProblem'),

  user: function(){
    return this.get('currentlySignedInUser')
  }.property('currentlySignedInUser'),

  userHasAFoot: function(){
    return this.get('user.hasLeftFoot') || this.get("user.hasRightFoot")
  }.property('user.hasLeftFoot', 'user.hasRightFoot'),

  userCanSeeCan: function(){
    return this.get('user.area.cans.length') > 0
  }.property('user.area.cans.length'),

  copPayingAttention: function(){
    return this.get('cop.attention') > 5
  }.property('cop.attention'),

  copInUserArea: function(){
    return this.get(‘cop.area’) == this.get(‘user.area’))
  }.property('cop.area', 'user.area'),

  copNotAProblem: function(){
    return !(this.get('copPayingAttention') && this.get('copInUserArea'))
  }.property('copPayingAttention', 'copInUserArea')
```

---

#Ember.computed to the rescue!

```
canKickCan: Ember.computed.and(‘userHasAFoot’, ‘userCanSeeCan’, ‘copNotAProblem’),

user: Ember.computed.alias(‘currentlySignedInUser’),

userHasAFoot: Ember.computed.or(‘user.hasLeftFoot’, ‘user.hasRightFoot’),

userCanSeeCan: Ember.computed.notEmpty(‘user.area.cans’),

copPayingAttention: Ember.computed.gt(‘cop.attention’, 5),

copInUserArea: function(){
  return this.get(‘cop.area’) == this.get(‘user.area’))
}.property('cop.area', 'user.area'),

copAProblem: Ember.computed.and(‘copPayingAttention’, ‘copInUserArea’),

copNotAProblem: Ember.computed.not(‘copAProblem’)
```
---

```javascript
  canKickCan: function(){
    return this.get('userHasAFoot') && this.get('userCanSeeCan') && this.get('copNotAProblem')
  }.property('userHasAFoot', 'userCanSeeCan', 'copNotAProblem'),
```

vs

```javascript
  canKickCan: Ember.computed.and(‘userHasAFoot’, ‘userCanSeeCan’, ‘copNotAProblem’),
```

---

```javascript
  user: function(){
    return this.get('currentlySignedInUser')
  }.property('currentlySignedInUser'),
```

vs

```javascript
  user: Ember.computed.alias(‘currentlySignedInUser’),
```

---

```javascript
  userHasAFoot: function(){
    return this.get('user.hasLeftFoot') || this.get("user.hasRightFoot")
  }.property('user.hasLeftFoot', 'user.hasRightFoot'),
```

vs

```javascript
  userHasAFoot: Ember.computed.or(‘user.hasLeftFoot’, ‘user.hasRightFoot’),
```

---

```javascript
  userCanSeeCan: function(){
    return this.get('user.area.cans.length') > 0
  }.property('user.area.cans.length'),
```

vs

```javascript
  userCanSeeCan: Ember.computed.notEmpty(‘user.area.cans’),
```

---

```javascript
  copPayingAttention: function(){
    return this.get('cop.attention') > 5
  }.property('cop.attention'),

```

vs

```javascript
  copPayingAttention: Ember.computed.gt(‘cop.attention’, 5),
```

---

```javascript
  copInUserArea: function(){
    return this.get(‘cop.area’) == this.get(‘user.area’))
  }.property('cop.area', 'user.area'),
```

vs

```javascript
  copInUserArea: Ember.computed.equal('cop.area', 'user.area'),
```

---

#Oops

That last one doesn't work that way.

Ember.computed.equal works like this:

```javascript
var Hamster = Ember.Object.extend({
  napTime: Ember.computed.equal('state', 'sleepy')
});
var hamster = Hamster.create();
hamster.get('napTime'); // false
hamster.set('state', 'sleepy');
hamster.get('napTime'); // true
hamster.set('state', 'hungry');
hamster.get('napTime'); // false
```

So we leave that one alone

---

```javascript
  copNotAProblem: function(){
    return !(this.get('copPayingAttention') && this.get('copInUserArea'))
  }.property('copPayingAttention', 'copInUserArea')
```

vs

```javascript
  copAProblem: Ember.computed.and(‘copPayingAttention’, ‘copInUserArea’),
  copNotAProblem: Ember.computed.not(‘copAProblem’)
```

---

Leading to this:

```
canKickCan: Ember.computed.and(‘userHasAFoot’, ‘userCanSeeCan’, ‘copNotAProblem’),

user: Ember.computed.alias(‘currentlySignedInUser’),

userHasAFoot: Ember.computed.or(‘user.hasLeftFoot’, ‘user.hasRightFoot’),

userCanSeeCan: Ember.computed.notEmpty(‘user.area.cans’),

copPayingAttention: Ember.computed.gt(‘cop.attention’, 5),

copInUserArea: function(){
  return this.get(‘cop.area’) == this.get(‘user.area’))
}.property('cop.area', 'user.area'),

copAProblem: Ember.computed.and(‘copPayingAttention’, ‘copInUserArea’),

copNotAProblem: Ember.computed.not(‘copAProblem’)
```

---

No comments needed

---

Documentation: `http://emberjs.com/api/#method_computed_alias`

More Macros: `https://github.com/jamesarosen/ember-cpm`