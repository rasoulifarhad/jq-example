## JQ

See [JSON object values into CSV with jq](https://qmacro.org/blog/posts/2022/05/19/json-object-values-into-csv-with-jq/)

### Recap

#### Processing JSON with jq

In the "Invoking jq" section of the [manual](https://stedolan.github.io/jq/manual/), it says:

> jq filters run on a stream of JSON data. The input to jq is parsed as a sequence of </br>
> whitespace-separated JSON values which are passed through the provided filter one at </br>
> a time. The output(s) of the filter are written to standard out, again as a sequence </br>
> of whitespace-separated JSON data. </br>

#### Basic filters

> Identity: .

> Object Identifier-Index: .foo, .foo.bar

> Optional Object Identifier-Index: .foo?

> Generic Object Index: .[<string>] -- You can also look up fields of an object using syntax like .["foo"] (.foo above is a shorthand version of this, but only for identifier-like strings).

> Array Index: .[2]

> Array/String Slice: .[10:15]

> Array/Object Value Iterator: .[]

  > echo '{"array":[1,2]}' | jq '.array[]'
  > echo '{"object":{"a":1,"b":2}}' | jq '.object[]'

> .[]? Like .[], but no errors will be output if . is not an array or object.

> Comma: ,
 
  > echo '1 2' | jq '[0,.]'
  > echo '1 2' | jq '{x:.,y:(2*.)}'

> Pipe: |

> map : map(x) is equivalent to [.[] | x]. In fact, this is how it's defined.

#### Types and Values

> Array construction: []  // [.foo, .bar, .baz] 

> Object Construction: {} // {user: .user, title: .title}

> Recursive Descent: ..   // use ..|.a? to find all the values of object keys "a" in any object found "below" .

#### Builtin operators and functions

See [Builtin operators and functions](https://stedolan.github.io/jq/manual/#Builtinoperatorsandfunctions)

#### Conditionals and Comparisons

See [Conditionals and Comparisons](https://stedolan.github.io/jq/manual/#ConditionalsandComparisons)

#### Regular expressions (PCRE)

See [Regular expressions (PCRE)](https://stedolan.github.io/jq/manual/#RegularexpressionsPCRE)

#### Advanced features

See [Advanced features](https://stedolan.github.io/jq/manual/#Advancedfeatures)

#### Try this

1. 

```
echo '{"array":[1,2]}' | jq '.array[]'

Response:

1
2

```

2. 

```
echo '{"object":{"a":1,"b":2}}' | jq '.object[]'

Response:

1
2

```

3. 

```
echo '1 2' | jq -c '[0,.]'

Response:

[0,1]
[0,2]

```

4. 

```
echo '1 2' | jq -c '{x:.,y:(2*.)}'

Response:

{"x":1,"y":2}
{"x":2,"y":4}

```

5. 

```
echo '1 2 3' | jq .+1

Response:

2
3
4

```

6. 

```
echo '1 2 3' | jq add

Response:

jq: error (at <stdin>:1): Cannot iterate over number (1)
jq: error (at <stdin>:1): Cannot iterate over number (2)
jq: error (at <stdin>:1): Cannot iterate over number (3)

```

7. 

```
echo '1 2 3' | jq -s add

Response:

6

```

It transforms the input from a sequence of discrete values (numbers) into a single composite value (an array of numbers).
 
``` 
echo '1 2 3' | jq -s .

Response:

[
  1,
  2,
  3
]
```

But this:

```
echo '1 2 3' | jq  .

Response:

1
2
3

```

8. ( echo '[1,2,3]' | jq add  ) == ( echo '1 2 3' | jq -s add	 )

```
echo '[1,2,3]' | jq add

Response:

6

```

9. 

```
echo '[1,2,3] [4,5,6]' | jq add

Response:

6
15

```

10. 

```
echo '[1,2,3] [4,5,6]' | jq '[add] | add'

Response:

6
15

```

11. 

```
echo '[1,2,3] [4,5,6]' | jq '[add]'

Response:

[
  6
]
[
  15
]

```

12.  

```
echo '[1,2,3] [4,5,6]' | jq -s '[.[] | add]'

Response:

[
  6,
  15
]

```

13. 

````
echo '[1,2,3] [4,5,6]' | jq -s '[.[] | add]  | add'

Response:

21

```

14. 

```
echo '[1,2,3] [4,5,6]' | jq -n 'reduce inputs as $array (0; .+($array|add))'

Response:

21

```

> map(x) is equivalent to [.[] | x]. In fact, this is how it's defined.

15. 

```
echo '[1,2,3] [4,5,6]' | jq -s 'map(add) | add'

Response:

21

```


15. 

```
seq 3 | jq '.'
seq 3 | jq -c '.'
seq 3 | jq -c -s '.'
seq 3 | jq  -s add

```

See [introducing-zq](https://www.brimdata.io/blog/introducing-zq/)
See [zq](https://zed.brimdata.io/docs/commands/zq/)

**Recap** JSON values:

- object
- array
- string
- number
- "true"
- "false"
- "null"
  
See [Introducing JSON](https://www.json.org/json-en.html)

#### Dataset

```json

{
  "@odata.context": "https://services.odata.org/V4/Northwind/Northwind.svc/$metadata#Customer_and_Suppliers_by_Cities",
  "value": [
    {
      "City": "Berlin",
      "CompanyName": "Alfreds Futterkiste",
      "ContactName": "Maria Anders",
      "Relationship": "Customers"
    },
    {
      "City": "México D.F.",
      "CompanyName": "Ana Trujillo Emparedados y helados",
      "ContactName": "Ana Trujillo",
      "Relationship": "Customers"
    },
    {
      "City": "México D.F.",
      "CompanyName": "Antonio Moreno Taquería",
      "ContactName": "Antonio Moreno",
      "Relationship": "Customers"
    },
    ......
    
    {
      "City": "Paris",
      "CompanyName": "Spécialités du monde",
      "ContactName": "Dominique Perrier",
      "Relationship": "Customers"
    }
  ],
  "@odata.nextLink": "../../../v4/northwind/northwind.svc/Customer_and_Suppliers_by_Cities?$skiptoken='Sp%C3%A9cialit%C3%A9s%20du%20monde','Customers'"
}

```

#### Goal: Convert JSON into csv

```

"Berlin","Alfreds Futterkiste","Maria Anders","Customers"
"México D.F.","Ana Trujillo Emparedados y helados","Ana Trujillo","Customers"
"México D.F.","Antonio Moreno Taquería","Antonio Moreno","Customers"

```

#### Iterating through the entities

1. 

```
jq '.' entities.json

```

Response:

```json
{
  "@odata.context": "https://services.odata.org/V4/Northwind/Northwind.svc/$metadata#Customer_and_Suppliers_by_Cities",
  "value": [
    {
      "City": "Berlin",
      "CompanyName": "Alfreds Futterkiste",
      "ContactName": "Maria Anders",
      "Relationship": "Customers"
    },
    {
      "City": "México D.F.",
      "CompanyName": "Ana Trujillo Emparedados y helados",
      "ContactName": "Ana Trujillo",
      "Relationship": "Customers"
    },
    {
      "City": "México D.F.",
      "CompanyName": "Antonio Moreno Taquería",
      "ContactName": "Antonio Moreno",
      "Relationship": "Customers"
    }
  ]
}

```

2. 

```
jq '.value' entities.json
```

Response:

```json

[
  {
    "City": "Berlin",
    "CompanyName": "Alfreds Futterkiste",
    "ContactName": "Maria Anders",
    "Relationship": "Customers"
  },
  {
    "City": "México D.F.",
    "CompanyName": "Ana Trujillo Emparedados y helados",
    "ContactName": "Ana Trujillo",
    "Relationship": "Customers"
  },
  {
    "City": "México D.F.",
    "CompanyName": "Antonio Moreno Taquería",
    "ContactName": "Antonio Moreno",
    "Relationship": "Customers"
  }
]

```

3. add the [array value iterator](https://stedolan.github.io/jq/manual/#Array/ObjectValueIterator:.%5B%5D) ([]) 

```
jq '.value[]' entities.json
```

Response:

```json

{
  "City": "Berlin",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "Relationship": "Customers"
}
{
  "City": "México D.F.",
  "CompanyName": "Ana Trujillo Emparedados y helados",
  "ContactName": "Ana Trujillo",
  "Relationship": "Customers"
}
{
  "City": "México D.F.",
  "CompanyName": "Antonio Moreno Taquería",
  "ContactName": "Antonio Moreno",
  "Relationship": "Customers"
}

```

The first invocation (`.value`) emitted a single JSON value (an array), the second (`.value[]`) caused three JSON values (three objects) to be emitted, effectively one at a time.

4. 

**length filter**: Given an array will return the number of elements, and given an object will return the number of key-value pairs (and, for that matter, given a string, will return the length of that string).

```
$ jq '.value | length ' entities.json

```

Response:

```
3

```

5. 

**length filter**: Given an array will return the number of elements, and given an object will return the number of key-value pairs (and, for that matter, given a string, will return the length of that string).

```
$ jq '.value[] | length ' entities.json
```

Response:

```json

4
4
4

```

6. 

```
jq '.value[] | [ .City, .CompanyName, .ContactName, .Relationship ]  ' entities.json
```

Response:

```json

[
  "Berlin",
  "Alfreds Futterkiste",
  "Maria Anders",
  "Customers"
]
[
  "México D.F.",
  "Ana Trujillo Emparedados y helados",
  "Ana Trujillo",
  "Customers"
]
[
  "México D.F.",
  "Antonio Moreno Taquería",
  "Antonio Moreno",
  "Customers"
]

```

7. 

```
jq -r '.value[] | [ .City, .CompanyName, .ContactName, .Relationship ] | @csv ' entities.json
```

Response:

```

"Berlin","Alfreds Futterkiste","Maria Anders","Customers"
"México D.F.","Ana Trujillo Emparedados y helados","Ana Trujillo","Customers"
"México D.F.","Antonio Moreno Taquería","Antonio Moreno","Customers"

```

8. 

```
jq '.value[] | keys ' entities.json
```

Response:

```json

[
  "City",
  "CompanyName",
  "ContactName",
  "Relationship"
]
[
  "City",
  "CompanyName",
  "ContactName",
  "Relationship"
]
[
  "City",
  "CompanyName",
  "ContactName",
  "Relationship"
]

```

9. 

```
jq -r '.value[] | keys | @csv' entities.json
```

Response:

```

"City","CompanyName","ContactName","Relationship"
"City","CompanyName","ContactName","Relationship"
"City","CompanyName","ContactName","Relationship"

```

10. 

```
jq  '.value[0] | keys ' entities.json
```

Response:

```json

[
  "City",
  "CompanyName",
  "ContactName",
  "Relationship"
]

```

11. 

```
jq  '.value[0] | keys as $k | $k' entities.json
```

Response:

```json

[
  "City",
  "CompanyName",
  "ContactName",
  "Relationship"
]

```

12. 

```
jq  '.value[0] | keys[] as $k | $k' entities.json 
```

Response:

```

"City"
"CompanyName"
"ContactName"
"Relationship"

```
We may be focusing deeper and deeper on the keys here, but don't forget we always have the identity filter (.) to give us access to the input, to whatever came through the pipe to where we are now, as it were.

13. Variable assignment with 'as' as a foreach loop

```
jq  '.value[0] | keys[] as $k | .' entities.json
```

Response:

```json

{
  "City": "Berlin",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "Relationship": "Customers"
}
{
  "City": "Berlin",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "Relationship": "Customers"
}
{
  "City": "Berlin",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "Relationship": "Customers"
}
{
  "City": "Berlin",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "Relationship": "Customers"
}

```

Odd, the same object, four times. But when we stare at that for a second, we realise that it's exactly what we asked for. With keys[] we're iterating through the keys of the object (City, CompanyName, ContactName and Relationship). Four of them. So whatever is beyond the pipe after that, which is simply the identity filter (.), is being called four times. And the identity filter (which simply outputs whatever it receives as input) receives as input the original object.

14. 

What we might expect . to output is one key, each time. That would be the case if we didn't assign keys[] to the variable $k with keys[] as $k. Let's remove the as $k bit to see:

```
jq  '.value[0] | keys[]  | .' entities.json 
```

Response:

```

"City"
"CompanyName"
"ContactName"
"Relationship"
```

So in this case, .'s input are (each time) the keys of the object. The important thing to realise here is that the variable assignment as $k means that the input that came into that expression (the object) passes straight through unconsumed to the next filter. This part of the manual for the section on variables helps to explain:

> The expression exp as $x | ... means: for each value of expression exp, run the rest of </br>
> the pipeline with the entire original input, and with $x set to that value. Thus as </br>
> functions as something of a foreach loop. </br>

15. 

While that's an odd thing to produce, it helps a lot here. Having the input at this stage in the pipeline (.) set to the object, combined with the "foreach loop" (as the manual described it) iterating over the values in $k, is very useful!

Let's look at that in a basic form; how about emitting an array with two elements, the first being the value of $k and the second being the input, each time:

```
jq  '.value[0] | keys[] as $k | [$k, .]' entities.json
```

Response:

```json

[
  "City",
  {
    "City": "Berlin",
    "CompanyName": "Alfreds Futterkiste",
    "ContactName": "Maria Anders",
    "Relationship": "Customers"
  }
]
[
  "CompanyName",
  {
    "City": "Berlin",
    "CompanyName": "Alfreds Futterkiste",
    "ContactName": "Maria Anders",
    "Relationship": "Customers"
  }
]
[
  "ContactName",
  {
    "City": "Berlin",
    "CompanyName": "Alfreds Futterkiste",
    "ContactName": "Maria Anders",
    "Relationship": "Customers"
  }
]
[
  "Relationship",
  {
    "City": "Berlin",
    "CompanyName": "Alfreds Futterkiste",
    "ContactName": "Maria Anders",
    "Relationship": "Customers"
  }
]

```

16. 

And of course, look what we can do with that combination of data in . and $k, using the generic object index like this .[$k] to look up the value of each of the keys:

```
 jq  '.value[0] | keys[] as $k | .[$k]' entities.json 
```

Response:

```

"Berlin"
"Alfreds Futterkiste"
"Maria Anders"
"Customers"

```

17. 

Great! And if we wrap this entire expression in an array construction ([...]), we then have the right shape (an array) to give to the @csv format string (and as we're emitting CSV again we'll use the --raw-output option again here):

```
jq -r '.value[0] | [keys[] as $k | .[$k]] | @csv' entities.json
```

Response:

```

"Berlin","Alfreds Futterkiste","Maria Anders","Customers"

```

18. 

```
jq -r '.value[] | [keys[] as $k | .[$k]] | @csv' entities.json

```

Response:

```

"Berlin","Alfreds Futterkiste","Maria Anders","Customers"
"México D.F.","Ana Trujillo Emparedados y helados","Ana Trujillo","Customers"
"México D.F.","Antonio Moreno Taquería","Antonio Moreno","Customers"

```

19. 

I'm likely to want to use this approach again some time, so I'll store the core construct here as a function in my local ~/.jq file (see the modules section of the manual for more detail):

> def onlyvalues: [ keys[] as $k | .[$k] ];


```
jq -r '.value[] | onlyvalues | @csv' entities.json
```

Response:

```

"Berlin","Alfreds Futterkiste","Maria Anders","Customers"
"México D.F.","Ana Trujillo Emparedados y helados","Ana Trujillo","Customers"
"México D.F.","Antonio Moreno Taquería","Antonio Moreno","Customers"

```

