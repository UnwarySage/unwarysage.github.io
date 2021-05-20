{:title "Anouncing Scurvy"
:layout :post
:tags ["programming" "clojure" "reagent" "projects"]}

I made a character generator for Knave, availible [here!](https://unwarysage.github.io/scurvy)

# ...Knave?
Recently, I've been on something of an Old School Renaissance RPG kick. I've always enjoyed reading OSR blogs, the sheer variety and depth of original ideas in them is staggering.
However, something tipped over, and I finally started looking for the rules that people use to run OSR games.
Considering the homebrewing tradition that is extremely prevalent in the OSR community, that's a little strange a quest at times.

However, I followed a trail of recommendations and found [Knave](https://www.drivethrurpg.com/product/250888/Knave).
I was struck by the simplicity and concision of the rules. I found [a collection](https://dungeonsandpossums.com/2020/04/some-great-knave-rpg-resources/) of tools for Knave, and it included several character generators, which tickled my fancy.

# The Idea
I noticed, reading over the code for [Lawbreaker](https://lawbreaker.herokuapp.com/), and thought... Why does this need a backend?
Storing the random seed in the URL would allow bookmarking and sharing characters as small links.

This seemed an ideal project, small enough I wouldn't get caught in scope-creep, and with a designated walk-away point, as adding additional features to the generator section post-launch would invalidate already built characters, unless done extremely carefully.

I turned it over in my head for a day or so, and decided to enact the idea.

# The Tools
I wanted to use [Bulma](https://tailwindcss.com/). It was familiar to me, and I knew I could be effective quickly with it. If I wished, I could later rewrite the UI in [tailwind-css](https://tailwindcss.com/) as a learning exercise, but for now, I wanted momentum.

Similarly, as the site I was envisioning had only one button, re-frame was definitely overkill. I looked at rum, but decided instead to use the already familiar library reagent that underpins re-frame.

I initially used deps.edn as my dependency system, and really enjoyed the low-hassle configuration. However, as I neared release, the paucity of documentation on using `:advanced` compilation mode with [deps.edn](https://clojurescript.org/guides/webpack) came to bite me.

[shadow-cljs](https://shadow-cljs.github.io/docs/UsersGuide.html) however, was an near drop-in replacement, and very well-documented on top. The switch only took about half an hour, including the time to set up the optimizing build.

# The Code
Moving quickly, I established three namespaces.

* `scurvy.page`, which would handle all the html, state, and so on. The plumbing of the app.
* `scurvy.tables`, which was a namespace and not an `.edn` file to allow for programmatic generation of the item details. Saved a lot of typing.
* `scurvy.character`, which would do the actual number crunching and assembly, drawing from `scurvy.tables`.

I made some simple display components that would just dump the character map data to the screen, and simultaneously worked on building up the generator, and the views to display what had been generated. Back and forth, in very rapid iterations.

With some quick regex work in a function called `copy-parse`, I could extract the tables from the pdf very quickly, and so that barely slowed me, despite the size of the namespace.

Once the generator was more or less working, and the display was nearly done, I took a pass through the Knave booklet and made sure that everything matched the 'on paper' rules. I caught a few bugs, such as unarmored characters having +1 to their armor defense.

Finally, I looked around for some programmable random number generators that would work in ClojureScript. [cljx-sampling](https://github.com/ashenfad/cljx-sampling) fit the bill perfectly,
and as I'd made sure all of the functions in `scurvy.character` used a single `roll-die` function, I just needed to create a new namespace, `scurvy.random` setup the groundwork for seedable random numbers, and refactor the create-character function to optionally take a seed and set the PRNG from it before generating.

This could probably be improved with a macro, something like `with-open`, but for the PRNG.

Some more work in `scurvy.page` and some research in the Javascript API the ClojureScript can access, and I could load and set the seed in the URL.

I added the name database that Lawbreaker pulled from, and did some polish work, such as notifications of unusual builds, and Scurvy was ready.

I switched to shadow-cljs, and built an optimized version. Once that was done, I adapted this blog's build script to handle the Clojure and ClojureScript dependencies, added caching because the amount of dependencies pulled in for ClojureScript compilation was rather high, and Scurvy was out and in the wild.

# Retrospective
I had set out to fully finish something, and made choices to reflect that. I was frankly overwhelmed by how fast this made me. Scurvy was built, from first file to published, in under two days. Less than a full workday, if it had been the only thing I worked on. I'm *extremely* happy with the momentum I kept.

Things I could definitely improve would include:

* Accessibility. I need to read more deeply about this, but I consider it to be a glaring omission. Once I know more, I will be returning to fix its absence.
* Re-do the CSS, rolling my own for more control and learning. Consider using purge-css or bulma's own "require what you need only" tools to cut down the page size even more.
* Add a printable version button, and/or one that exports to a copy pastable text format for ease.
* Make sure that Scurvy writes to the browser history, so that you can use the browser back button to re-find that amazing character.


I hope that [Scurvy](https://unwarysage.github.io/scurvy/) serves you well!
