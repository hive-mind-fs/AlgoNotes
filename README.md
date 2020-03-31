# Puzzle Solver Algo (Bit Array Solution)


## Source Code

~~~typescript
import fs from 'fs';
import { promisify } from 'util';
const readFileAsync = promisify(fs.readFile);

const toVchar = (cs: string) => {
  let cv = 0;
  for (let i = 0; i < cs.length; i++) {
    let c = cs[i];
    {
      let ordinal =
        (function(c: any) {
          return c.charCodeAt == null ? c : c.charCodeAt(0);
        })(c) - 'a'.charCodeAt(0);
      cv |= 1 << ordinal;
    }
  }
  return cv;
};

const bitCount = (u: number) => {
  const uCount = u - ((u >> 1) & 0o33333333333) - ((u >> 2) & 0o11111111111);
  return ((uCount + (uCount >> 3)) & 0o30707070707) % 63;
};

const firstSetBit = (int: number) => int & -int;

const puzzlesForBoardSet = (Vchar: number) => {
  let puzzles = [];
  let decayingBoardSet = Vchar;
  while (decayingBoardSet !== 0) {
    let required = firstSetBit(decayingBoardSet);
    decayingBoardSet ^= required;
    puzzles.push([Vchar, required]);
  }
  return puzzles;
};

const puzzleMaster = (dict: any, minLen: number, maxLets: number) => {
  let wordSet = new Set();
  let wordsByVector = new Map();
  let boardSet = new Set();
  let puzzles = [];
  loop1: for (let i = 0; i < dict.length; i++) {
    let word = dict[i];
    if (word.length < minLen) {
      continue;
    }
    let charArr = word.split('');
    for (let j = 0; j < charArr.length; j++) {
      let c = charArr[j];
      if (
        c.charCodeAt(0) < 'a'.charCodeAt(0) ||
        c.charCodeAt(0) > 'z'.charCodeAt(0)
      ) {
        continue loop1;
      }
    }
    const vector = toVchar(charArr);
    const distinctLetterCount = bitCount(vector);
    if (distinctLetterCount > maxLets) {
      continue;
    }
    wordSet.add(word);
    wordsByVector.set(
      vector,
      wordsByVector.has(vector)
        ? [...wordsByVector.get(vector), word]
        : new Array(word)
    );
    if (distinctLetterCount === maxLets) {
      if (!boardSet.has(vector) ? boardSet.add(vector) : false) {
        puzzles.push(puzzlesForBoardSet(vector));
      }
    }
  }
  return { wordsByVector, puzzles };
};

const addSolutions = (
  result: any[],
  Vreq: number,
  Vopt: number,
  hashmap: Map<number, any[]>
) => {
  if (Vopt === 0) {
    result.push(hashmap.has(Vreq) ? hashmap.get(Vreq) : null);
  } else {
    let nextOneHot = firstSetBit(Vopt);
    let expandedVreq = Vreq | nextOneHot;
    let nextVopt = Vopt ^ nextOneHot;
    addSolutions(result, Vreq, nextVopt, hashmap);
    addSolutions(result, expandedVreq, nextVopt, hashmap);
  }
};

const solutionsTo = (
  Vlets: number,
  Vreq: number,
  hashmap: Map<number, any[]>
) => {
  const solutions = []; //new Set();
  const Vopt = Vlets & ~Vreq;
  addSolutions(solutions, Vreq, Vopt, hashmap);
  return solutions;
};

const createReqMap = () => {
  let reqMap = new Map();
  const alphabet = [
    'A',
    'B',
    'C',
    'D',
    'E',
    'F',
    'G',
    'H',
    'I',
    'J',
    'K',
    'L',
    'M',
    'N',
    'O',
    'P',
    'Q',
    'R',
    'S',
    'T',
    'U',
    'V',
    'W',
    'X',
    'Y',
    'Z'
  ];

  for (let i = 0; i < alphabet.length; i++) {
    reqMap.set(toVchar(alphabet[i]), alphabet[i]);
  }
  return reqMap;
};

const createLetterSetMap = (puzzles: any[], hashmap: Map<number, any[]>) => {
  let letters = [];
  let lettersMap = new Map();
  for (let i = 0; i < puzzles.length; i++) {
    letters.push(puzzles[i][0][0]);
    lettersMap.set(
      letters[i],
      hashmap
        .get(letters[i])[0]
        .replace(/(.)(?=.*\1)/g, '')
        .toUpperCase()
    );
  }
  return lettersMap;
};

const solver = (puzzles: any, hashmap: any) => {
  let solutions = [];
  const reqMap = createReqMap();
  const letterSetMap = createLetterSetMap(puzzles, hashmap);
  for (let i = 0; i < puzzles.length; i++) {
    for (let j = 0; j < 7; j++) {
      let Vchar = puzzles[i][j][0];
      let Vreq = puzzles[i][j][1];
      solutions.push([
        reqMap.get(Vreq),
        letterSetMap.get(Vchar),
        hashmap.get(Vchar),
        solutionsTo(Vchar, Vreq, hashmap)
          .flat(Infinity)
          .filter(x => x)
      ]);
    }
  }
  return solutions;
};

const SolvePuzzles = async (dictPath: string) => {
  const t0 = performance.now();

  console.log('Reading dictionary...');
  const dict = await readFileAsync(__dirname + dictPath, 'utf8');
  const words = await dict.split('\n'); 

  console.log('Generating puzzles...');
  const { wordsByVector } = puzzleMaster(words, 4, 7); 
  const { puzzles } = puzzleMaster(words, 4, 7); 

  console.log('Solutions:');
  console.log(solver(puzzles, wordsByVector)); //Take ~1421.117 milliseconds
};

(async () => {
  await SolvePuzzles('/ubuntu-wamerican.txt'));
})();
~~~

####To run the algo:
 
* Download and open this folder in your IDE: <http://www.filedropper.com/bitarrsolution>
* Then in your terminal run:

~~~
npm i -g typescript
npm i -g ts-node
npm i
s-node <PATH>/<NAME>.ts
~~~
	

##Deconstruction


#### Preprocessing:

* First, a dictionary text file is read into an array of its words. 
* Then each element in the dictionary array is converted into its character array (an array of the characters that make up a given word. e.g. `'word' => ['w', 'o', 'r', 'd']`). 
* Then the array is preprocessed to index each word by its character vector using the toVchar() function.

~~~typescript 
const toVchar = (cs: string) => {
  let cv = 0;
  for (let i = 0; i < cs.length; i++) {
    let c = cs[i];
    {
      let ordinal =
        (function(c: any) {
          return c.charCodeAt == null ? c : c.charCodeAt(0);
        })(c) - 'a'.charCodeAt(0);
      cv |= 1 << ordinal;
    }
  }
  return cv;
};
~~~

The character vector for a set of characters is the integer whose ith bit is set exactly if the ith letter of the alphabet is included in the character set. All indices are zero-based, and the least significant bits are numbered first. The alphabet is fixed to the 26 lowercase Latin characters as used in the English alphabet.

* e.g. the character vector for `['a', 'd', 'e', 'g']` is 0b1011001 or 89.

The character vector for a word is defined to be the character vector for the set of unique characters in that string. 

* e.g., the character set for both 'adage' and 'gagged' is {a, d, e, g}. Since both these strings share a character set they both share the character vector 89.

* the letiable cv is used as a storage for bits. Where every bit in the integer cv can be treated as a flag. 

* Eventually cv becomes an array of bits. Each bit in the array states whether the character with bit's index was found in the string or not. 

* The left shift operator `<<` moves all the bits in its first operand to the left by the number of places specified in the second operand. New bits are filled with zeros. Shifting a value left by one position is equivalent to multiplying it by 2, shifting two positions is equivalent to multiplying by 4, and so on.

Bitwise shifting any number x to the left by y bits yields `x * 2 ** y`. 

* e.g. `9 << 3` translates to `9 * (2 ** 3) = 9 * (8) = 72`.

The bitwise OR assignment operator `|=` uses the binary representation of both operands, does a bitwise OR operation on them and assigns the result to the letiable. $(x |= y)  	 \equiv (x  = x | y)$

~~~
let bar = 5;
bar |= 2; // 7
// 5: 00000000000000000000000000000101
// 2: 00000000000000000000000000000010
// -----------------------------------
// 7: 00000000000000000000000000000111
~~~

####Counting Bits:
~~~typescript
const bitCount = (u: number) => {
  const uCount = u - ((u >> 1) & 0o33333333333) - ((u >> 2) & 0o11111111111);
  return ((uCount + (uCount >> 3)) & 0o30707070707) % 63;
};
~~~


Instead of looping over the entire number, sum up the number in blocks of 3 (octal) and count them in parallel. This method has 0(1) time and space complexity.
See: <https://docs.microsoft.com/en-us/archive/blogs/jeuge/bit-fiddling-3>

If you were to initialize a bitArray at the beginning of the function e.g. let bitArray = []. And then push each of the results from bit count to it 

* e.g. `if (distinctLetterCount > maxLets) {continue;} bitSet.push(distinctLetterCount)` `puzzleMaster(['ABCDEF', 'GAGGED', 'ADAGE'], 4, 7)` would return the bitArray: `[ 6, 4, 4 ]`



####Generate the puzzles for a board set:
#####Getting the lowest set bit:

~~~typescript
const firstSetBit = (int) => int & -int;
~~~

The firstSetBit() method takes an integer and returns an integer value with at most a single one-bit, in the position of the lowest-order ("rightmost") one-bit in the specified integer value.


It returns zero if the specified value has no one-bits in its two's complement binary representation, that is, if it is equal to zero.

e.g.

~~~
Number = 170
Binary = 10101010
Number of one bits = 4
Highest one bit = 128
Lowest one bit = 2
~~~

The Bitwise AND operator `&` performs the AND operation on each pair of bits. a AND b yields 1 i.f.f. both a and b are 1. The truth table for the AND operation is:

~~~
a   b   a AND b
0   0   	0
0   1   	0
1   0  		0
1   1   	1
~~~

Bitwise ANDing any number x with 0 yields 0.

#####The two's complement negation `-int` of the integer is taken as follows:

* first you complement it, setting all zeroes to the right of the lowest set bit to one and the lowest set bit to zero.

* then you add one, setting the bits on the right to zero and the lowest set bit becomes one again, ending the carry chain.

* Now the negation of the number has the same "right part", up to and including the lowest set bit.

* everything to the left of the lowest set bit is the complement of the input.
Thus if you take the bitwise AND of a number with its negation `int & -int`, all the bits to the left of the lowest set bit are cancelled out.
This leaves you with the isolated lowest set bit.


          
####Generating puzzle for a board set

The puzzlesForBoardSet() function takes a character vector as an argument, which should have at least one bit set and returns all puzzles whose char vector is the supplied argument
and whose required letter has exactly one bit set.

~~~typescript
const puzzlesForBoardSet = (Vchar: number) => {
  let puzzles = [];
  let decayingBoardSet = Vchar;
  while (decayingBoardSet !== 0) {
    let required = firstSetBit(decayingBoardSet);
    decayingBoardSet ^= required;
    puzzles.push([Vchar, required]);
  }
  return puzzles;
};
~~~

~~~typescript
let puzzles = [];
let decayingBoardSet = Vchar;
~~~

an empty array called puzzles is initialized and the letiable decayingBoardSet is set to be the charVector supplied as an argument.

~~~typescript
let required = firstSetBit(decayingBoardSet);
~~~

While the decayingBoardSet is not equal to zero we set the letiable required to be the result of firstSetBit() with decayingBoardSet supplied as the argument
meaning required is equal to the lowest order one bit of the charVector.

~~~typescript
decayingBoardSet ^= required;
~~~

Then the bitwise XOR assignment operator (^=) is applied to the value of the required letiable.
^= uses the binary representation of both operands, does a bitwise XOR operation on them and assigns the result to the letiable.
the bitwise XOR operator performs the XOR operation on each pair of bits. a XOR b yields 1 if a and b are different.

The truth table for the XOR operation is:

~~~
a   b   a XOR b
0   0   0
0   1   1
1   0   1
1   1   0
~~~

e.g.
~~~
let bar = 5;
bar ^= 2; // 7
5: 00000000000000000000000000000101
2: 00000000000000000000000000000010
7: 00000000000000000000000000000111
~~~


~~~typescript
puzzles.push([Vchar, required]);
~~~

While the lowest order one bit of the charVector is not equal to zero an array containing the character vector and its corresponding lowest order one bit are pushed to the puzzles array. When the while loop ends the puzzles array is returned.


If you defined a character vector as follows: `const cv = toVchar(['A', 'D', 'G', 'E']);`
and then passed it as an argument to the puzzlesForBoardSet(cv) function the puzzles array would be returned as follows: `[ [ 89, 1 ], [ 89, 8 ], [ 89, 16 ], [ 89, 64 ] ]`

Where r is the required vector and cv is the character vector: $$ \sum r = cv $$

 

## Puzzle Master Function

~~~typescript
const puzzleMaster = (dict: any, minLen: number, maxLets: number) => {
  let wordSet = new Set();
  let wordsByVector = new Map();
  let boardSet = new Set();
  let puzzles = [];
  loop1: for (let i = 0; i < dict.length; i++) {
    let word = dict[i];
    if (word.length < minLen) {
      continue;
    }
    let charArr = word.split('');
    for (let j = 0; j < charArr.length; j++) {
      let c = charArr[j];
      if (
        c.charCodeAt(0) < 'a'.charCodeAt(0) ||
        c.charCodeAt(0) > 'z'.charCodeAt(0)
      ) {
        continue loop1;
      }
    }
    const vector = toVchar(charArr);
    const distinctLetterCount = bitCount(vector);
    if (distinctLetterCount > maxLets) {
      continue;
    }
    wordSet.add(word);
    wordsByVector.set(
      vector,
      wordsByVector.has(vector)
        ? [...wordsByVector.get(vector), word]
        : new Array(word)
    );
    if (distinctLetterCount === maxLets) {
      if (!boardSet.has(vector) ? boardSet.add(vector) : false) {
        puzzles.push(puzzlesForBoardSet(vector));
      }
    }
  }
  return { wordsByVector, puzzles };
};
~~~

* The puzzleMaster function takes a dictionary of words, a minimum word length, and a maximum number of letters allowed in a puzzle as arguments.

* We start by initializing a has set called wrodSet to store the processed words, a hash Map called wordsByVector, a set called boardSet to store the set of all character vectors some permutation of which is a valid word with exactly 7 unique letters, and an array called puzzles to store all the potential puzzles.

### Loop1:
* checks each word in the dictionary and if its length is under the minimum word length it is ignored
* JS continue (Breaks one iteration on specific condition and continues with the next iteration in the loop);

~~~typescript
for (let i = 0; i < dict.length; i++) {
    let word = dict[i];
    if (word.length < minLen) {
      continue;
    }
~~~

### Loop2:

~~~typescript
  let charArr = (word).split('');
  let c = charArr[j];
      if (
        c.charCodeAt(0) < 'a'.charCodeAt(0) ||
        c.charCodeAt(0) > 'z'.charCodeAt(0)
      ) {
        continue loop1;
      }
    }
~~~

* First take a word as string and converts it to an array of its characters. e.g. `'HELLO' => ['H', 'E', 'L', 'L', 'O']`
* The next piece checks the asccii value of each charachter in the array to make sure its in the latin alphabet if it fails the check ignore this word and the outer loop continues



~~~typescript
const vector = toVChar(charArr);
~~~

The next piece uses the toVChar() function to convert each words character array into a character vector as defined above.

If you were to initialize a vectorArray at the beginning of the function e.g. `let vectorArray = []`. And then push each of the results from toVChar() function to it 

e.g. 

~~~typescript
const vector = toVChar(charArr); vectorArray.push(vector)
~~~

`puzzleMaster(['ABCDEF', 'GAGGED', 'ADAGE'], 4, 7)` would return the vectorArray: `[ 63, 89, 89 ]`


####The next piece adds each word whose distinctLetterCount is less than the maximum number of letters allowed in a board to the wordSet:

e.g. `puzzleMaster(['ABCDEF', 'GAGGED', 'ADAGE'], 4, 7) => Set { 'ABCDEF', 'GAGGED', 'ADAGE' }`

~~~typescript
if (distinctLetterCount > maxLets) {continue;}
wordSet.add(word);
~~~

####The next piece attempts to emulate javas computeIfAbsent() method for hash maps: 

The computeIfAbsent(Key, Function) method of HashMap class is used to compute value
for a given key using the given mapping function, if key is not already associated with a value (or is mapped to null) and enter that computed value in Hash map else null.


Map.prototype.set(key, value) Sets the value for the key in the Map object and Returns the Map object.

Map.prototype.has(key) Returns a boolean asserting whether a value has been associated to the key in the Map object or not.

Map.prototype.get(key) Returns the value associated with a particular key.

e.g. If you initialize a new Map => `let hashMap = new Map();` and then set the value for the key 'A' to be 'X' =>  `hashMap.set('A', 'X');`
you can use the hashMap.has(key) method to set a value only on the condition that a value has been associated to the specified key in the Map object => `hashMap.set('A', hashMap.has('A')? hashMap.get('A') : 'B');` will return 'X' in our case.

In the case that that no value has been associated to the key 'A', the value 'B' would be assigned.

####You then check the hash map for each character vector

You see if the key is already associated with a word. If it is not then you add the word as an array as the keys value. if another word has the same character vector it is added to the array.

~~~typescript
 wordsByVector.set(
      vector,
      wordsByVector.has(vector)
        ? [...wordsByVector.get(vector), word]
        : new Array(word)
    );
~~~

`puzzleMaster(['ABCDEF', 'GAGGED', 'ADAGE'], 4, 7)` would return wordsByVector: `Map { 63 => 'ABCDEF', 89 => ['GAGGED', 'ADAGE'] };`

If you return the wordsByVector map for the dictionary.txt file it has size 48,906

####The next piece checks if the number of distinct letters is equal to the number of tiles allowed on the board. If the check returns true those character arrays are added to a set of potential character arrays for generating a board.

~~~typescript
if (distinctLetterCount === maxLets) {
    if (!boardSet.has(vector) ? boardSet.add(vector) : false) {
        puzzles.push(puzzlesForBoardSet(vector));
    }
};
~~~

This method will add the character vector only if it is not present in the HashSet
else the function will return False if the character vector is already present in the HashSet.

~~~typescript
if (!boardSet.has(vector) ? boardSet.add(vector) : false);
~~~

If the above method returns true the puzzlesForBoardSet() function (defined above) is called with the vector as an argument and the result is added to the puzzles array.

~~~typescript
puzzles.push(puzzlesForBoardSet(vector));
~~~

`puzzleMaster(['ABCDEFG', 'HIJKLMN'], 4, 7)` would return the puzzles array:

	[
	    [
	        [ 127, 1 ],
	        [ 127, 2 ],
	        [ 127, 4 ],
	        [ 127, 8 ],
	        [ 127, 16 ],
	        [ 127, 32 ],
	        [ 127, 64 ]
	    ],
	    [
	        [ 16256, 128 ],
	        [ 16256, 256 ],
	        [ 16256, 512 ],
	        [ 16256, 1024 ],
	        [ 16256, 2048 ],
	        [ 16256, 4096 ],
	        [ 16256, 8192 ]
	    ]
	]

Puzzles is an array of arrays of sets of character vectors and their corresponding first set bit required letter that were checked to see if the number of distinct letters were equal to the number of tiles allowed on the board.  

~~~typescript
return { wordsByVector, puzzles };
~~~

Finally you return and object with the wordByVector hash map and the puzzles array for later use.

###Getting back to words:


####Finding all words that contain the Required Vector and use no additional characters but those specified in the Optional Vector then adding those words to the given result set.

A one-hot is a group of bits among which the legal combinations of values are only those with a single high (1) bit and all the others low (0).
<https://en.wikipedia.org/wiki/One-hot>

~~~typescript
const addSolutions = (
  result: any[],
  Vreq: number,
  Vopt: number,
  hashmap: Map<number, any[]>
) => {
  if (Vopt === 0) {
    result.push(hashmap.has(Vreq) ? hashmap.get(Vreq) : null);
  } else {
    let nextOneHot = firstSetBit(Vopt);
    let expandedVreq = Vreq | nextOneHot;
    let nextVopt = Vopt ^ nextOneHot;
    addSolutions(result, Vreq, nextVopt, hashmap);
    addSolutions(result, expandedVreq, nextVopt, hashmap);
  }
};
~~~

####Finding all words that can be formed in the given puzzle.
~~~typescript
const solutionsTo = (
  Vlets: number,
  Vreq: number,
  hashmap: Map<number, any[]>
) => {
  const solutions = []; //new Set();
  const Vopt = Vlets & ~Vreq;
  addSolutions(solutions, Vreq, Vopt, hashmap);
  return solutions;
};
~~~

####Create a has map of character vectors for the required letter:
~~~typescript
const createReqMap = () => {
  let reqMap = new Map();
  const alphabet = [
    'A',
    'B',
    'C',
    'D',
    'E',
    'F',
    'G',
    'H',
    'I',
    'J',
    'K',
    'L',
    'M',
    'N',
    'O',
    'P',
    'Q',
    'R',
    'S',
    'T',
    'U',
    'V',
    'W',
    'X',
    'Y',
    'Z'
  ];

  for (let i = 0; i < alphabet.length; i++) {
    reqMap.set(toVchar(alphabet[i]), alphabet[i]);
  }
  return reqMap;
};
~~~

~~~javascript
Map {
  1 => 'A',
  2 => 'B',
  4 => 'C',
  8 => 'D',
  16 => 'E',
  32 => 'F',
  64 => 'G',
  128 => 'H',
  256 => 'I',
  512 => 'J',
  1024 => 'K',
  2048 => 'L',
  4096 => 'M',
  8192 => 'N',
  16384 => 'O',
  32768 => 'P',
  65536 => 'Q',
  131072 => 'R',
  262144 => 'S',
  524288 => 'T',
  1048576 => 'U',
  2097152 => 'V',
  4194304 => 'W',
  8388608 => 'X',
  16777216 => 'Y',
  33554432 => 'Z'
}
~~~

####Create a has map of character vectors for the letter set 
~~~typescript
const createLetterSetMap = (puzzles: any[], hashmap: Map<number, any[]>) => {
  let letters = [];
  let lettersMap = new Map();
  for (let i = 0; i < puzzles.length; i++) {
    letters.push(puzzles[i][0][0]);
    lettersMap.set(
      letters[i],
      hashmap
        .get(letters[i])[0]
        .replace(/(.)(?=.*\1)/g, '')
        .toUpperCase()
    );
  }
  return lettersMap;
};
~~~

~~~javascript
Map {
...
  17301661 => 'YACHTED',
  16786757 => 'YACKING',
  17170521 => 'YRDAGES',
  16925715 => 'YEARBOK',
  16916817 => 'YEARING',
  17694993 => 'YASTIER',
  21776400 => 'YLOWEST',
  16820560 => 'YELPING',
  19136913 => 'YESHIVA',
  16787800 => 'YELDING',
  17188888 => 'YODLERS',
  18497728 => 'YGHOURT',
  18759744 => 'YOGURTS',
  17981520 => 'YOUNGER',
  18368672 => 'YOTHFUL',
  17958164 => 'YUCKIER',
  17835332 => 'YUCKING',
  18352408 => 'YULTIDE',
  18616592 => 'YUMIEST',
  34359313 => 'ZEALOTS',
  34883601 => 'ZEALOUS',
  34349456 => 'ZENITHS',
  50757776 => 'ZEPHYRS',
...
}

}
~~~

####Finally solve all the puzzles by looking up their vectors in the maps
~~~typescript
const solver = (puzzles: any, hashmap: any) => {
  let solutions = [];
  const reqMap = createReqMap();
  const letterSetMap = createLetterSetMap(puzzles, hashmap);
  for (let i = 0; i < puzzles.length; i++) {
    for (let j = 0; j < 7; j++) {
      let Vchar = puzzles[i][j][0];
      let Vreq = puzzles[i][j][1];
      solutions.push([
        reqMap.get(Vreq),
        letterSetMap.get(Vchar),
        hashmap.get(Vchar),
        solutionsTo(Vchar, Vreq, hashmap)
          .flat(Infinity)
          .filter(x => x)
      ]);
    }
  }
  return solutions;
};
~~~

###Notes

During preprocessing, any words with more than 7 unique characters are discarded since they violate the rules of the game. All character sets of size exactly 7 are flagged as being possible letter sets since they are panagrams. After this all valid puzzles can be generated by taking each valid letter set and using each letter as the required letter.

the trick here speed wise is packing each character set into a single machine word as all these bitwise operations are essentially costless.	
the puzzles array is every possible puzzle (the letter set and each letter in the letter set as the required letter)

Then you lookup each word that includes the required letter and the any of the other letters....

There are 26 possible choices for the center letter and $25\choose 6$ possible combinations for the other letters. This makes $26 * {25\choose 6} = \frac{25!}{( 6! (25 - 6)! )} = 26 * 177,100 = 4,604,600$ possible honeycombs.

For each subset S with n elements, its power set contains 128 elements $|P(S)| = 2^7 = 128$

For each subset s use the precomputed tables to look up all words with character set S U {r}, where r is the required letter. Taking the union of these 128 sets completes the algorithm. 


####WordSet
All words that might appear in a puzzle. This includes all words composed only of characters in the alphabet (i.e., the 26 lowercase Latin characters), with length at least 4 and with at most 7 distinct letters.

####wordsByVector
A hash map containing the set of all valid words as values, keyed to their character vectors. Two words will be in the same bucket if and only if they contain the same sets of characters, not considering multiplicity. The union of all the values equals the full word set.


####Puzzles
The set of all potential puzzles. This is trivially formed by choosing each potential required letter for each potential letter set.





	Asccii Table
	Char Octal Dec Hex
	A     101    65  41
	B     102    66  42
	C     103    67  43
	D     104    68  44
	E     105    69  45
	F     106    70  46
	G     107    71  47
	H     110    72  48
	I     111    73  49
	J     112    74  4a
	K     113    75  4b
	L     114    76  4c
	M     115    77  4d
	N     116    78  4e
	O     117    79  4f
	P     120    80  50
	Q     121    81  51
	R     122    82  52
	S     123    83  53
	T     124    84  54
	U     125    85  55
	V     126    86  56
	W     127    87  57
	X     130    88  58
	Y     131    89  59
	Z     132    90  5a


##Example output

~~~json
[
    'N',
    'ABETING',
    [ 'abetting', 'abnegating', 'battening', 'beating' ],
    [
      'tint',      'gigging',  'ginning',    'inning',    'igniting',
      'ting',      'tinging',  'tinning',    'tinting',   'entente',
      'teen',      'tenet',    'tent',       'nine',      'intent',
      'nineteen',  'nite',     'tine',       'gene',      'gent',
      'egging',    'engine',   'geeing',     'genie',     'genii',
      'getting',   'ignite',   'netting',    'teeing',    'tenting',
      'tieing',    'tinge',    'tingeing',   'binging',   'binning',
      'gibing',    'biting',   'been',       'bent',      'bitten',
      'begging',   'begin',    'beginning',  'being',     'benign',
      'binge',     'bingeing', 'ebbing',     'begetting', 'betting',
      'gibbeting', 'anti',     'attain',     'taint',     'tannin',
      'titan',     'gang',     'gnat',       'tang',      'again',
      'aging',     'angina',   'gagging',    'gaging',    'gain',
      'gaining',   'ganging',  'nagging',    'agitating', 'attaining',
      'gating',    'giant',    'initiating', 'tagging',   'tainting',
      'tanning',   'tatting',  'ante',       'antenna',   'antennae',
      'eaten',     'neat',     'tenant',     'inane',     'initiate',
      'innate',    'engage',   'agent',      'gannet',    'negate',
      'tangent',   'teenage',  'ageing',     'engaging',  'anteing',
      'antigen',   'eating',   'gentian',    'negating',  'tenanting',
      ... 26 more items
    ]
  ],
  [
    'S',
    'ABETORS',
    [ 'abettors', 'boaster', 'boasters', 'boaters' ],
    [
      'soot',     'sots',      'toots',    'toss',    'tost',
      'tots',     'roost',     'roosts',   'roots',   'rotors',
      'rots',     'sort',      'sorts',    'tors',    'torso',
      'torsos',   'torts',     'trots',    'sees',    'sets',
      'settee',   'settees',   'tees',     'test',    'testes',
      'tests',    'errs',      'seer',     'seers',   'sere',
      'serer',    'ester',     'esters',   'reset',   'resets',
      'rest',     'rests',     'serest',   'setter',  'setters',
      'steer',    'steers',    'street',   'streets', 'stress',
      'stresses', 'teeters',   'terse',    'terser',  'tersest',
      'tester',   'testers',   'trees',    'tress',   'tresses',
      'toes',     'tosses',    'totes',    'errors',  'ores',
      'roes',     'rose',      'roses',    'sore',    'sorer',
      'sores',    'otters',    'resort',   'resorts', 'restore',
      'restorer', 'restorers', 'restores', 'retorts', 'rooster',
      'roosters', 'rosette',   'rosettes', 'roster',  'rosters',
      'sorest',   'sorter',    'sorters',  'stereo',  'stereos',
      'store',    'stores',    'terrors',  'tortes',  'totters',
      'trotters', 'bobs',      'boobs',    'boos',    'boss',
      'sobs',     'boost',     'boosts',   'boots',   'boors',
      ... 234 more items
    ]
  ],
  [
    'R',
    'ABHORED',
    [ 'abhorred', 'harbored', 'headboard' ],
    [
      'horror',  'error',     'here',       'hero',    'door',
      'odor',    'rood',      'deer',       'erred',   'redder',
      'reed',    'dodder',    'doddered',   'doer',    'erode',
      'eroded',  'odder',     'order',      'ordered', 'redo',
      'reorder', 'reordered', 'rode',       'rodeo',   'herd',
      'herded',  'horde',     'horded',     'boor',    'beer',
      'bore',    'borer',     'robber',     'robe',    'herb',
      'brood',   'bedder',    'bred',       'breed',   'breeder',
      'border',  'bordered',  'bored',      'brooded', 'brooder',
      'robbed',  'robed',     'roar',       'hoorah',  'area',
      'rare',    'rarer',     'rear',       'hare',    'hear',
      'hearer',  'rhea',      'radar',      'ardor',   'road',
      'hard',    'hoard',     'adder',      'dare',    'dared',
      'deader',  'dear',      'dearer',     'dread',   'dreaded',
      'rared',   'read',      'reader',     'reared',  'reread',
      'adore',   'adored',    'oared',      'roared',  'adhere',
      'adhered', 'harder',    'hardheaded', 'hared',   'header',
      'heard',   'redhead',   'redheaded',  'hoarded', 'hoarder',
      'barb',    'arbor',     'boar',       'abhor',   'harbor',
      'barber',  'bare',      'barer',      'bear',    'bearer',
      ... 31 more items
    ]
  ],
    [
      'sits',      'tits',     'ills',     'sill',      'sills',
      'lilts',     'list',     'lists',    'silt',      'silts',
      'slit',      'slits',    'still',    'stills',    'stilt',
      'stilts',    'tills',    'tilts',    'sees',      'sets',
      'settee',    'settees',  'tees',     'test',      'testes',
      'tests',     'eels',     'ells',     'else',      'lees',
      'less',      'lessee',   'lessees',  'sell',      'sells',
      'lest',      'lets',     'settle',   'settles',   'sleet',
      'sleets',    'steel',    'steels',   'tells',     'sises',
      'sissies',   'sissiest', 'site',     'sites',     'sties',
      'testiest',  'testis',   'ties',     'isle',      'isles',
      'leis',      'lies',     'lilies',   'lisle',     'sillies',
      'elites',    'elitist',  'elitists', 'islet',     'islets',
      'listless',  'littlest', 'silliest', 'sleetiest', 'sliest',
      'steeliest', 'stile',    'stiles',   'stillest',  'tiles',
      'titles',    'tittles',  'bibs',     'ibis',      'bits',
      'titbits',   'bills',    'bliss',    'bees',      'ebbs',
      'beets',     'beset',    'besets',   'best',      'bests',
      'bets',      'belles',   'bells',    'bless',     'blesses',
      'beetles',   'belts',    'blest',    'ibises',    'bites',
      ... 205 more items
    ]
  ],
  [
    'A',
    'ABJURED',
    [ 'abjured' ],
    [
      'aura',     'ajar',     'raja',    'area',     'rare',
      'rarer',    'rear',     'aurae',   'urea',     'radar',
      'added',    'dead',     'adder',   'dare',     'dared',
      'deader',   'dear',     'dearer',  'dread',    'dreaded',
      'rared',    'read',     'reader',  'reared',   'reread',
      'jade',     'jaded',    'jarred',  'adjure',   'adjured',
      'barb',     'babe',     'beau',    'barber',   'bare',
      'barer',    'bear',     'bearer',  'bureau',   'jabber',
      'jabberer', 'abjure',   'baud',    'daub',     'bard',
      'brad',     'drab',     'abed',    'baaed',    'bade',
      'bead',     'beaded',   'dabbed',  'daubed',   'abrade',
      'abraded',  'badder',   'barbed',  'barbered', 'bared',
      'barred',   'beard',    'bearded', 'bread',    'breaded',
      'debar',    'debarred', 'drabber', 'dauber',   'jabbed',
      'jabbered', 'abjured'
    ]
  ],
  ... 54633 more items
]
~~~
