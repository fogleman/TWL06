## TWL06: The Official Scrabble Dictionary

A convenient, self-contained, 515 KB Scrabble dictionary module, ideal
for use in word games.

Functionality:

* Check if a word is in the dictionary.
* Enumerate all words in the dictionary.
* Determine what letters may appear after a given prefix.
* Determine what words can be formed by anagramming a set of letters.

Sample usage:

    >>> import twl
    >>> twl.check('dog')
    True
    >>> twl.check('dgo')
    False
    >>> words = set(twl.iterator())
    >>> len(words)
    178691
    >>> twl.children('dude')
    ['$', 'd', 'e', 's']
    >>> list(twl.anagram('top'))
    ['op', 'opt', 'pot', 'to', 'top']

Provides a simple API using the TWL06 (official Scrabble tournament) 
dictionary. Contains American English words that are between 2 and 15 
characters long, inclusive. The dictionary contains 178691 words.

Implemented using a DAWG (Directed Acyclic Word Graph) packed in a 
binary lookup table for a very small memory footprint, not only on 
disk but also once loaded into RAM. In fact, this is the primary
benefit of this method over others - it is optimized for low memory
usage (not speed).

The data is stored in the Python module as a base-64 encoded, 
zlib-compressed string.

Each record of the DAWG table is packed into a 32-bit integer.

    MLLLLLLL IIIIIIII IIIIIIII IIIIIIII

* M - More Flag
* L - ASCII Letter (lowercase or '$')
* I - Index (Pointer)

The helper method `_get_record(index)` will extract these three elements 
into a Python tuple such as `(True, 'a', 26)`.

All searches start at index 0 in the lookup table. Records are scanned 
sequentially as long as the More flag is set. These records represent all 
of the children of the current node in the DAWG. For example, the first
26 records are:

    0 (True, 'a', 26)
    1 (True, 'b', 25784)
    2 (True, 'c', 11666)
    3 (True, 'd', 39216)
    4 (True, 'e', 33704)
    5 (True, 'f', 50988)
    6 (True, 'g', 46575)
    7 (True, 'h', 60884)
    8 (True, 'i', 56044)
    9 (True, 'j', 67454)
    10 (True, 'k', 65987)
    11 (True, 'l', 76093)
    12 (True, 'm', 68502)
    13 (True, 'n', 83951)
    14 (True, 'o', 79807)
    15 (True, 'p', 89048)
    16 (True, 'q', 88465)
    17 (True, 'r', 113967)
    18 (True, 's', 100429)
    19 (True, 't', 125171)
    20 (True, 'u', 119997)
    21 (True, 'v', 134127)
    22 (True, 'w', 131549)
    23 (True, 'x', 136449)
    24 (True, 'y', 136058)
    25 (False, 'z', 136584)

The root node contains 26 children because there are words that start 
with all 26 letters. Other nodes will have fewer children. For example,
if we jump to the node for the prefix 'b', we see:

    25784 (True, 'a', 25795)
    25785 (True, 'd', 28639)
    25786 (True, 'e', 27322)
    25787 (True, 'h', 29858)
    25788 (True, 'i', 28641)
    25789 (True, 'l', 29876)
    25790 (True, 'o', 30623)
    25791 (True, 'r', 31730)
    25792 (True, 'u', 32759)
    25793 (True, 'w', 33653)
    25794 (False, 'y', 33654)

So the prefix 'b' may be followed only by these letters:

    a, d, e, h, i, l, o, r, u, w, y

The helper method `_get_child(index, letter)` will return a new index
(or `None` if not found) when traversing an edge to a new node. For
example, `_get_child(0, 'b')` returns 25784.

The search is performed iteratively until the sentinel value, $, is
found. If this value is found, the string is a word in the dictionary.
If at any point during the search the appropriate child is not found,
the search fails - the string is not a word.

See also:

* http://code.activestate.com/recipes/577835-self-contained-twl06-dictionary-module-500-kb/
* http://en.wikipedia.org/wiki/Official_Tournament_and_Club_Word_List
* http://www.isc.ro/lists/twl06.zip
