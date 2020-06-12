# Tweedown + Twindle.js
Tweedown: a superset of [GFM (GitHub Flavored Markdown)](https://github.github.com/gfm/) that allows defining [Twee](https://github.com/iftechfoundation/twine-specs/blob/master/twee-3-specification.md)-like passages.

Twindle.js: A JavaScript library for compiling Tweedown files to web applications (HTML/CSS/JavaScript).

## Why?
Twine works for folks, but how about if interactive fiction could be written with the ease and options of markdown as well as the extensibility of an IDE? You could use git! VSCode has Live Share and cool stuff like that!

The aim is also to reduce the hoops one has to go through to extend the basic functionality.

## Tweedown File Type
It is recommended Tweedown files have a `.twdn` file extension.

## Tweedown Notation
Tweedown files are composed of one or more passages.  Passages consist of a header and content section.

### Passage Header
Each header must be a single line and is composed of the following components (in order):

1. Required start token, a double colon (`::`), that must begin the line.
2. Required passage name.

NOTE: Each component after the start token may be preceded by one or more spaces for readability.

**Examples:**
Minimal working example:

```
:: An overgrown path
```

### Passage Content
The content section begins on the next line after the passage header and continues until the next passage header or the end of file.

One of the goals is to replicate the functionality of Twine's Chapbook format, with the caveat of respecting GFH and sticking as closely to it as possible. So, links will work like in GFH, and implementations of stuff like cycling links will be an intuitive step from GFH links. We also won't implement features in a way that will cause confusion for the user or the compiler (for example, Chapbook's modifiers look too much like links, as they also use `[]` brackets).

In addition to GFM, you can use HTML inside of the passage content. However, if adding GFM inside an HTML element, you must add `markdown=1` inside the element's tag.

#### Variables & Tracery

Twindle.js uses Tracery.js for procedural generation. Twindle.js also stores and returns variables in the same way as grammars for consistency. All of these **symbols** should be defined directly under the `:: Passage Name`.

You must have the symbol name in quotes, then a **rule** that's either a single item in quotation marks, or a comma-separated array in brackets (`[]`). Each item in the array must be in double quotion marks. You cannot have a commas after the last item in the array, and the entire line needs to end with a comma if there's another variable/grammar on a line after it.
```
:: Passage Name
"name": "Phillip",
"animal": ["unicorn","raven","sparrow","scorpion","coyote","eagle","owl","lizard","zebra","duck","kitten"]
```
 You must have an empty line under your variables/grammars. Call an array by surrounding the symbol with `#`.

```
:: Passage Name
"name": "Phillip",
"animal": ["unicorn","raven","sparrow","scorpion","coyote","eagle","owl","lizard","zebra","duck","kitten"]

#name# has a pet #animal#.
```
Tracery's modifiers also work, however we are using [Pluralize.js](https://github.com/blakeembrey/pluralize) so that plurals are correct insetad of only adding an s at the end.
> Sometimes you want to be able to capitalize or pluralize a symbol, or add a/an without worrying about which is correct. We provide a few modifier that you can put on the end of symbols with a dot separating them: 'animal.s.capitalize' will make an animal plural and capitalize it.
> 
> Useful modifiers are 's' for plural, 'ed' for past tense, 'capitalize' and 'capitalizeAll', and '.a' to add a/an in front.
```
#name#'s #animal#.s.
```

#### Line Modifiers
**Line modifiers** apply special handling to a line or group of lines within a passage (lines are grouped if there are no empty lines between them). All line modifiers start with a period. You can add a modifier on its own line before, inbetween, or after lines. Because modifiers and their parameters are predefined, you don't have to put brackets or quotation marks, so they are easy to enter.

##### Generate `.gen`
The `.gen` modifier adds text using artificial intelligence. The story so far is fed into the algorithm (there is also entity recognition), along with other configurations. You can specify a word limit for the generator.

`.gen` can be put between empty lines or directly after a line. If there is no period at the end of the last line, the algorithm will generate the rest of the sentence, automatically appending to it. By default, `.gen` sets up for the next line or group of lines, so results will not be too wild of a tangent.

```
:: Passage Name
.gen model: cyberpunk, limit: 20 words
```
You can also specify a passage that is a target. If you do, the generation will incorporate that passage (and, notably, end as it ends).
```
:: Passage Name
.gen model: cyberpunk, limit: 20 words, target: Another Passage
```

> **Is it just one model?**  
> We have trained models for various genres, though you can leave out this parameters for it to use the general model.
>
> **Why does my text have code in it?**  
> We train using lines from Tracery.js projects (stripping out variables, etc.) because there is a greater chance they include markup such as GFM and SSML. We convert GFM to SSML for text-to-speech but intentional SSML adds more nuanced pauses, emphases and such.
>
> **How do we improve or customize the generation?**  
> Aside from the parameters of `.gen` and the target text, you can hover over the text and click the gear icon that appears. This will allow you to modify the text. Your changes will be sent to improve the generation, and will also be saved in your playthrough.

##### Parser `.parser`
The `.parser` modifier displays a text prompt. Text entered into the promopt is intepreted by the AI algorithm used by `.gen`, with any next line or group of lines serving as a target for the alogrithm's output. Again, you can specify a word limit. 

##### Append `.apend`
The `.apend` modifier, when placed between two lines, appends the second line to the end of the first line. You can use this for effects such as the `.reveal` modifier.

##### Reveal `.reveal`
`.reveal` shows a link with text that when clicked changes into different text. You can use it to append another passage or in conjunction with `.gen` (the link text is passed to the generator if you do this).
```
:: Passage Name
You walk up to the door and 
.append
.reveal [open it]("swing it open to reveal an empty closet.")

You walk up to the door and 
.append
.reveal [open it](Another passage)

You walk up to the door and 
.append
.reveal [open it](.gen)
```

#### Dropdown & Cycling Menus
Dropdown menus provide a selection of choices, which may be links (or not, if you want to do something like `.gen`).
```
You walk up to the next door and 
.append
.dropdown for: "door-choice", text: ["open it", "lock it", "kick it down"], reveal: ["open it to see an empty closet", "lock it.", .gen]
```
Cycling menus work similarly, but are like choices on a wheel. In this example, we change it to a link menu. If you do both reveal and link, the reveal text becomes the link text. We also use the `.redirect` modifier so that the page redirects to the selection's associated passage.
```
You walk up to the next door and 
.append
.cycling for: "door-choice" text: ["open it.", "lock it!", "kick it down!!!"], links: ["Opened Passage", "Locked Passage", "Kicked Passage"]
.wait
.redirect target: "door-choice"
```
These menus save the selection to a variable. The variable name from the examples above would be "door-choice".

**Tip:** You can also do dropdown & cycling menus using variables and links. The arrays have to have the same number of items. The default is cycling, and the redirect will be automatic. However, no variable will be saved using this method.
```
:: Passage Name
"door-choice-text": ["open it.", "lock it!", "kick it down!!!"],
"door-choice-links": ["Opened Passage", "Locked Passage", "Kicked Passage"]

You walk up to the next door and [#door-choice-text#](#door-choice-links#).dropdown
```
This has the same effect:
```
You walk up to the next door and ["open it.", "lock it!", "kick it down!!!"]("Opened Passage", "Locked Passage", "Kicked Passage").cycling
```

##### Voice `.voice`
The `.voice` modifier adds a voice track to lines.
```
:: Passage Name
"I am speaking to you!"
.voice file: voices/line1.mp3, start-time: 04:02.500, duration: 3 seconds
```
You can use a text-to-speech engine's request URL as well as filter parameters; these even work with Tracery and you can format it using ssml (so long as the engine supports it). Note: Markdown formatting is not sent to the engine. Also, note that ssml doesn't modify the text - just the voiceover. This example uses [Google's text-to-speech engine](https://cloud.google.com/text-to-speech).

**Note:** Google's text-to-speech is a paid service. It's my hope that organizations such as the [Interactive Fiction Technology Foundation](https://iftechfoundation.org/) can host Twindle.js & work something out with Google! There's also [Mozilla's TTS](https://github.com/mozilla/TTS), [Microsoft's TTS](https://azure.microsoft.com/en-us/services/cognitive-services/text-to-speech/#product-overview) and [Amazon Polly](https://aws.amazon.com/polly/).

```
:: Passage Name
"random-voice": [engine: #engine#, intonation: #intonation#, pitch: #pitch#, speakingRate: #speakingRate#],
"engine": "https://texttospeech.googleapis.com/v1beta1/text:synthesize",
"intonation": ["expressive", "somewhat expressive", "flat"],
"pitch": [.5, 1, 1.25, 2, 2.5],
"speakingRate": [.5, 1, 1.5]

"Robot talking here!"
.voice engine: https://texttospeech.googleapis.com/v1beta1/text:synthesize

"This is a _little_ more complicated!"
.voice #random-voice#

<emphasis level="strong">To be</emphasis>
<break time="200ms"/> or not to be, <break time="400ms"/>
<emphasis level="moderate">that</emphasis>
is the question.<break time="400ms"/>
Whether ‘tis nobler in the mind to suffer
The slings and arrows of outrageous fortune,<break time="200ms"/>
Or to take arms against a sea of troubles
And by opposing end them.
.voice #random-voice#
```

##### Continue `.cont`
If you add the `.cont` (continue) macro after a paragraph (regardless of whether `.voice` is used), the focus is moved to a button/link and the rest of the passage will not display until it is clicked/tapped or Enter is pressed.

##### Wait `.wait`
If voice is used, the `.wait` macro is like `.cont` but automatically moves to the next paragraph when the voice is done. You can add a length of time, for example if there is no voice file (e.g., `.wait duration: 200ms`).

If you have used the `.reveal`, `.dropdown` or `.cycling` modifiers, `.wait` will wait until a selection is made.
```
:: Passage Name
You walk up to the door and 
.append
.reveal [open it](.gen)
.wait

You walk up to the next door and
.append 
.dropdown text: ["open it", "lock it", "kick it down"], reveal: ["open it to see an empty closet", "lock it.", .gen]
.wait
```

##### Expires `.expires`
You can remove a line of text after a length of time using the `.expires` macro. You must put a time with it, e.g. `.expires duration: 200ms`. This must be put **after** the affected line. `.expires` only affects the single line above it.

##### Redirect `.redirect`
If you add the `.redirect` macro after a line, it will redirect to another passsage if another selection is not made in the alloted time. E.g., `.redirect duration: 3s, target: Time's up passage`. The time must be listed first, then the passage name.

```
:: Passage Name
"Blah blah blah"
.cont

Lorem ipsum dolor sit amet consectetur adipisicing elit. Non, aut! Nobis laborum doloribus culpa, non a repellat quod. Officia, velit.
.wait duration: 200ms

.voice file: voices/line1.mp3, start-time: 04:06.000, end-time: 04:09.000
"Blah blah blah"
.wait

* Go to [another passage](Next Passage 1)
* Or go to [this passage](Alternative Passage 2) maybe?
.expires duration: 200ms
.redirect duration: 3s, target: Time's up passage
```

##### Back `.back`
The `.back` modifier inserts a link to go back to the previous passage. **Note:** This is not `.undo`. Your story still contains the passage you go back from.

In this example, the link will say "Back".
```
:: Passage Name
There's nothing here.

.back
```
You can define the text that the back link appears as.
```
:: Passage Name
There's nothing here.

.back Let's go back.
```
You can also put .back in a link instead of a passage name.
```
:: Passage Name
There's nothing here, so you should [go back](.back) to where you came from.
```

##### Undo `.undo`
The `.undo` modifier works like `.back` except it doesn't keep the passage in your story. You can have `.undo` in every passage in your story if you like.

##### Restart `.restart`
The `.restart` modifier works like `.undo` except it restarts the story from the origin passage.

## Special Passages
I'm not sure if these should be included, but I am concerned for compatibility with [IFDB](https://ifdb.tads.org/). Basically, `:: StoryTitle` and `:: StoryData` would create/mofidy a `<tw-storydata>` element with various attributes, directly after the `id="story"` element that Twindle.js inserts into. Because Twindle.js runs on load, it would have to be added manually or with the aid of a VSCode extension.

### `StoryTitle`

The project's name. Maps to `<tw-storydata name>`.

### `StoryData`
 A JSON chunk encapsulating various Twine 2 compatible details about your project.  The currently supported properties include:

- `ifid`: (string) Required.  Maps to `<tw-storydata ifid>`.
- `format`: (string) Optional.  Maps to `<tw-storydata format>`.
- `format-version`: (string) Optional.  Maps to `<tw-storydata format-version>`.
- `start`: (string) Optional.  Maps to `<tw-passagedata name>` of the node whose `pid` matches `<tw-storydata startnode>`.
- `tag-colors`: (object of tag(string):color(string) pairs) Optional.  Pairs map to `<tw-tag>` nodes as `<tw-tag name>`:`<tw-tag color>`.
- `zoom`: (decimal) Optional.  Maps to `<tw-storydata zoom>`.

NOTES:

- An IFID (Interactive Fiction IDentifier) uniquely identifies compiled projects—see [The Treaty of Babel](https://babel.ifarchive.org/).  Twine 2 uses v4 (random) [UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier), using only capital letters, and Twee 3 compilers must follow suit to ensure maximum compatibility.
- For readability, it is recommended that the JSON be pretty-printed (line-broken and indented).
- It is recommended that the outcome of a decoding error should be to: emit a warning, discard the metadata, and continue processing the file.

**Example:**

```
:: StoryData
{
	"ifid": "D674C58C-DEFA-4F70-B7A2-27742230C0FC",
	"format": "Tweedown",
	"format-version": "1",
	"start": "My Starting Passage",
	"tag-colors": {
		"bar": "green",
		"foo": "red",
		"qaz": "blue"
	},
	"zoom": 0.25
}
```

### `Start`

The default starting passage.  May be overridden by the story metadata or compiler command.

## Twindle.js
A Javascript converter to HTML that can be used client side (in the browser) or server side (with NodeJS). Perhaps a fork extending [Showdown](https://github.com/showdownjs/showdown), with the showdown-icon extension thrown in for its great benefit.
* Saves choices in local storage. If the page loads and there is no target, but local storage has saved data, a modal displays to continue one's game(s).

Passages are converted to `<section>` elements with `id` attributes matching the passage name. If multple passages have the same name, an error is thrown. The CSS displays only the targeted `id`. Download Beaker Browser and see [this site](hyper://2d45c980a237f768319c13b057af4053d84d0ac3a860ab03091b1e51bfac252d/index.html) for an example of something similar to output.

This CDN converts the `.twdn` file with the same filename as the `.html` file, and inserts it into the HTML element with `id="story"`. So for `index.html` you should have `index.twdn`.

Tweedown Example:
  ```
  :: Passage Name
  Lorem ipsum dolor sit amet consectetur adipisicing elit. Non, aut! Nobis laborum doloribus culpa, non a repellat quod. Officia, velit.
  * Go to [another passage](Next Passage 1)
  * Or go to [this passage](Alternative Passage 2) maybe?
  ```
  Would create:
  ```
  <section id="Passage-Name">
  <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Non, aut! Nobis laborum doloribus culpa, non a repellat quod. Officia, velit.</p>
    <ul>
      <li>Go to <a href="#Next-Passage-1">another passage</a></li>
      <li>Or go to <a href="#Alternative-Passage-2">this passage</a> maybe?</li>
    </ul>
  </section>
  ```
  Which would look like:
  > Lorem ipsum dolor sit amet consectetur adipisicing elit. Non, aut! Nobis laborum doloribus culpa, non a repellat quod. Officia, velit.
  > * Go to [another passage](#Next-Passage-1)
  > * Or go to [this passage](#Alternative-Passage-2) maybe?

### Options

**generateHTML** - boolean, `true` by default. Set to `false` if you don't want the gamebook generated in the HTML (for example, if you are making an RPG and using the next option, you may instead call the JSON from your game's UI)