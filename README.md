# api documentation for  [speakeasy-nlp (v0.2.12)](http://www.github.com/nhunzaker/speakeasy)  [![npm package](https://img.shields.io/npm/v/npmdoc-speakeasy-nlp.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-speakeasy-nlp) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-speakeasy-nlp.svg)](https://travis-ci.org/npmdoc/node-npmdoc-speakeasy-nlp)
#### A simple natural language processor for node javascript.

[![NPM](https://nodei.co/npm/speakeasy-nlp.png?downloads=true)](https://www.npmjs.com/package/speakeasy-nlp)

[![apidoc](https://npmdoc.github.io/node-npmdoc-speakeasy-nlp/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-speakeasy-nlp_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-speakeasy-nlp/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-speakeasy-nlp/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-speakeasy-nlp/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Nate Hunzaker",
        "email": "nate.hunzaker@gmail.com"
    },
    "bugs": {
        "url": "https://github.com/nhunzaker/speakeasy/issues"
    },
    "dependencies": {
        "levenshtein": "*"
    },
    "description": "A simple natural language processor for node javascript.",
    "devDependencies": {
        "vows": "~0.8"
    },
    "directories": {},
    "dist": {
        "shasum": "c24661a1938b7b4340d8583d3d1ec9cb5bcae7c8",
        "tarball": "https://registry.npmjs.org/speakeasy-nlp/-/speakeasy-nlp-0.2.12.tgz"
    },
    "gitHead": "b6e53fc1577a8cc1dd780d602d30e0337394827c",
    "homepage": "http://www.github.com/nhunzaker/speakeasy",
    "keywords": [
        "natural language"
    ],
    "license": "MIT",
    "main": "index.js",
    "maintainers": [
        {
            "name": "natehunzaker",
            "email": "nate.hunzaker@gmail.com"
        },
        {
            "name": "nhunzaker",
            "email": "nate.hunzaker@viget.com"
        }
    ],
    "name": "speakeasy-nlp",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/nhunzaker/speakeasy.git"
    },
    "scripts": {
        "test": "vows"
    },
    "version": "0.2.12"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module speakeasy-nlp](#apidoc.module.speakeasy-nlp)
1.  [function <span class="apidocSignatureSpan">speakeasy-nlp.</span>classify (speech, debug)](#apidoc.element.speakeasy-nlp.classify)
1.  [function <span class="apidocSignatureSpan">speakeasy-nlp.</span>closest (string, words)](#apidoc.element.speakeasy-nlp.closest)
1.  object <span class="apidocSignatureSpan">speakeasy-nlp.</span>sentiment

#### [module speakeasy-nlp.sentiment](#apidoc.module.speakeasy-nlp.sentiment)
1.  [function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>analyze (phrase)](#apidoc.element.speakeasy-nlp.sentiment.analyze)
1.  [function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>negativity (phrase)](#apidoc.element.speakeasy-nlp.sentiment.negativity)
1.  [function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>positivity (phrase)](#apidoc.element.speakeasy-nlp.sentiment.positivity)



# <a name="apidoc.module.speakeasy-nlp"></a>[module speakeasy-nlp](#apidoc.module.speakeasy-nlp)

#### <a name="apidoc.element.speakeasy-nlp.classify"></a>[function <span class="apidocSignatureSpan">speakeasy-nlp.</span>classify (speech, debug)](#apidoc.element.speakeasy-nlp.classify)
- description and source-code
```javascript
function classify(speech, debug) {
  var text = speech || process.argv.slice(2).join(' ')
  var words = lexer.lex(text)
  var tagged = []
  var action = null
  var subject = null
  var owner = null

  // PREPROCESSING
  // -------------------------------------------------- //

  // Auto correct for missing punctuation
  if (getType(words.slice(-1)[0]) !== '.') {
    words.push('.')
  }

  // Classify!
  // -------------------------------------------------- //

  tagged = tagger.tag(words)

  if (debug) console.log(words)
  if (debug) console.log(tagged)

  var verbs = getTypes(tagged, 'VB')
  var nouns = getTypes(tagged, 'NN')
  var pronouns = getTypes(tagged, 'PRP') // finds all posessive pronouns
  var actions = getTypes(tagged, 'W')
  var adjectives = getTypes(tagged, 'JJ')
  var preps = getTypes(tagged, 'IN')
  var determiners = getTypes(tagged, 'DT')
  var to = getTypes(tagged, 'TO')

  // ACTION
  // Answers : "What should the nodebot do after given a
  // command"
  // -------------------------------------------------- //

  // Are there known action words present?
  if (actions.length > 0) {
    action = actions[0]
    // Are there base verbs present? Then it's probably
    // the first verb
  } else if (getTypes(tagged, 'VB', true).length > 0) {
    action = getTypes(tagged, 'VB', true)[0]
    // If there are nouns and prepositions then the action
    // is the first noun
    // ex: Repeat that last action
  } else if (nouns.length > 0 && preps.length > 0) {
    action = nouns[0]
    // Are there at least any adjectives that might work?
  } else if (adjectives.length > 0) {
    action = adjectives[0]
  }

  // Lowercase the action if we find one
  action = (action) ? action.toLowerCase() : action

  // OWNERSHIP
  // Answers : "Who is associated with the target of the
  // action?"
  // -------------------------------------------------- //

  // If there is posessive pronouns and we have an action, then
  // the owner is the last posessive word
  if (pronouns.length > 0 && action) {
    // The answer must begin where the action starts
    // ex: "Do you know [What time it is?]
    var answer = words.slice(words.indexOf(action))
    var lastPro = pronouns.slice(-1)[0]
    if (answer.indexOf(lastPro) > 0 || getTypes(answer, 'VB').length === 0) {
      owner = lastPro
    } else {
      owner = getBetween(answer, ['DT', 'TO'], '.')
      owner = stripTypes(owner, ['.', 'VB', 'VBZ']).join(' ')
    }
    // No ? Let's try between a preposition and
    // determiners/nouns
    // Example: "Compile all javascript as application.js"
  } else if (determiners.length > 0 && preps.length > 0) {
    var prepIndex = words.indexOf(preps[0])

    owner = words.slice(prepIndex + 1, -1).join(' ')
    // Hmm, now let's try between the action and the word "to"
  } else if (to.length > 0) {
    owner = getBetween(words, 'VB', 'TO').slice(0, -1).join(' ')
    // At this point, we can really only guess that
    // the owner is between the verb and the end of the
    // statement
  } else if (verbs.length > 0) {
    var self = nouns.filter(function (word) {
      return word.match(/^i$/i)
    })

    if (self.length) {
      owner = self.slice(0)
    } else {
      owner = getBetween(words, ['VBZ', 'VBP'], ['.']).slice(0, -1)
    }

    // Do we have the word "I" in here?
    if (owner.indexOf('I') === 0) {
      owner = owner.slice(0, 1)
    }

    // Strip accidental determinates and punctuation
    owner = stripTypes(owner, ['DT', '.']).join(' ')
  }

  // SUBJECT
  // Answers : "What is this statement about?"
  // -------------------------------------------------- //
  if (getType(words[0]) === 'VBZ' && words.indexOf(action) === words.length - 2) {
    subject = words.slice(1, -2).join(' ').trim()
  } else if (determiners.length === 0 && preps.length === 0 && getType(action) === 'VB') {
    subject = words.slice(1, -1).join(' ')
    // If ownership and there are prepositions, scan for words bteween
    // determinates and posessive words, and prepositions
  } else if (preps.length > 0) {
    if (preps.length > 1 || determiners.leng ...
```
- example usage
```shell
...

''' javascript

var speak = require("./speakeasy-nlp");

// Analyze sentences at a basic level
// ------------------------------------- //
speak.classify("What is your name?")             //=> { action: "what", owner: "listener", subject: "name" }
speak.classify("Do you know what time it is?")   //=> { action: "what", owner: "it", subject: "time" }

// Sentiment analysis
// ------------------------------------- //
speak.sentiment.negativity("I hate your guts")   //=> { score: 1, words: [hate] }
speak.sentiment.positivity("I love you")         //=> { score: 1, words: [love] }
...
```

#### <a name="apidoc.element.speakeasy-nlp.closest"></a>[function <span class="apidocSignatureSpan">speakeasy-nlp.</span>closest (string, words)](#apidoc.element.speakeasy-nlp.closest)
- description and source-code
```javascript
closest = function (string, words) {

    var shortest = words.toString().length
    ,   bestFit  = "";

    if (typeof words === 'string') {
        words = lexer.lex(words);
    }

    words.forEach(function(word) {

        var distance = lev(string, word);

        if (distance < shortest) {
            bestFit  = word;
            shortest = distance;
        }

    });

    return bestFit;
}
```
- example usage
```shell
...

speak.sentiment.analyze("I love you, but you smell something aweful")
// (Negative scores dictate a stronger influence of negative words)
//=> { score: -1, positive: { ... }, negative: { ... } }

// Closest word
// ------------------------------------- //
speak.closest("node", ["foo", "nodejs", "baz"])     //=> "nodejs"

'''


## Philosophy

The goal of this project is not to be the next final solution for natural language processing. There are plenty of
...
```



# <a name="apidoc.module.speakeasy-nlp.sentiment"></a>[module speakeasy-nlp.sentiment](#apidoc.module.speakeasy-nlp.sentiment)

#### <a name="apidoc.element.speakeasy-nlp.sentiment.analyze"></a>[function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>analyze (phrase)](#apidoc.element.speakeasy-nlp.sentiment.analyze)
- description and source-code
```javascript
function analyze(phrase) {
  var pos = positivity(phrase)
  var neg = negativity(phrase)

  return {
    score: pos.score - neg.score,
    comparative: pos.comparative - neg.comparative,
    positive: pos,
    negative: neg
  }
}
```
- example usage
```shell
...
speak.classify("Do you know what time it is?")   //=> { action: "what", owner: "it", subject: "time" }

// Sentiment analysis
// ------------------------------------- //
speak.sentiment.negativity("I hate your guts")   //=> { score: 1, words: [hate] }
speak.sentiment.positivity("I love you")         //=> { score: 1, words: [love] }

speak.sentiment.analyze("I love you, but you smell something aweful")
// (Negative scores dictate a stronger influence of negative words)
//=> { score: -1, positive: { ... }, negative: { ... } }

// Closest word
// ------------------------------------- //
speak.closest("node", ["foo", "nodejs", "baz"])     //=> "nodejs"
...
```

#### <a name="apidoc.element.speakeasy-nlp.sentiment.negativity"></a>[function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>negativity (phrase)](#apidoc.element.speakeasy-nlp.sentiment.negativity)
- description and source-code
```javascript
function negativity(phrase) {
  // Cache it
  var tokens = phrase.toLowerCase().split(' ')
  var hits = 0
  var words = []

  tokens.forEach(function (t) {
    if (negativeWords.indexOf(t) > -1) {
      hits++
      words.push(t)
    }
  })

  return {
    score: hits,
    comparative: hits / tokens.length,
    words: words
  }
}
```
- example usage
```shell
...
// Analyze sentences at a basic level
// ------------------------------------- //
speak.classify("What is your name?")             //=> { action: "what", owner: "listener", subject: "name" }
speak.classify("Do you know what time it is?")   //=> { action: "what", owner: "it", subject: "time" }

// Sentiment analysis
// ------------------------------------- //
speak.sentiment.negativity("I hate your guts")   //=> { score: 1, words: [hate] }
speak.sentiment.positivity("I love you")         //=> { score: 1, words: [love] }

speak.sentiment.analyze("I love you, but you smell something aweful")
// (Negative scores dictate a stronger influence of negative words)
//=> { score: -1, positive: { ... }, negative: { ... } }

// Closest word
...
```

#### <a name="apidoc.element.speakeasy-nlp.sentiment.positivity"></a>[function <span class="apidocSignatureSpan">speakeasy-nlp.sentiment.</span>positivity (phrase)](#apidoc.element.speakeasy-nlp.sentiment.positivity)
- description and source-code
```javascript
function positivity(phrase) {
  // Cache it
  var tokens = phrase.toLowerCase().split(' ')
  var hits = 0
  var words = []

  tokens.forEach(function (t) {
    if (positiveWords.indexOf(t) > -1) {
      hits++
      words.push(t)
    }
  })

  return {
    score: hits,
    comparative: hits / tokens.length,
    words: words
  }
}
```
- example usage
```shell
...
// ------------------------------------- //
speak.classify("What is your name?")             //=> { action: "what", owner: "listener", subject: "name" }
speak.classify("Do you know what time it is?")   //=> { action: "what", owner: "it", subject: "time" }

// Sentiment analysis
// ------------------------------------- //
speak.sentiment.negativity("I hate your guts")   //=> { score: 1, words: [hate] }
speak.sentiment.positivity("I love you")         //=> { score: 1, words: [love] }

speak.sentiment.analyze("I love you, but you smell something aweful")
// (Negative scores dictate a stronger influence of negative words)
//=> { score: -1, positive: { ... }, negative: { ... } }

// Closest word
// ------------------------------------- //
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
