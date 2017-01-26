The `lanes` policy in the [[circulation manager configuration file|Configuration]] is your chance to customize the lanes presented to patrons. You can have as many or as few lanes as you want. 

The value of `lanes` is a list.

```
{
 "lanes": [ ... ]
}
```

Each item in the list is a JSON object that describes one lane.

```
{
 "lanes": [ 
  { 
    "full_name" : "YA Fiction",
    "audiences" : "Young Adult",
    "fiction" : True
  }
 ]
}
```

Each lane may give values for the following keys:

## `full_name`

A name that uniquely identifies this lane. Required.

## `display_name`

The name shown to patrons. This does not have to be unique. If this value is not set, the `full_name` will be used instead.

## `audiences`

A single audience name, or a list of audience names. Only books classified for one of those audiences will be shown in this lane.

Audience names are: "Adults Only", "Adult", "Young Adult", and "Children". If this is not specified, the parent lane's value is inherited.

## `languages`

A single 3-character language code, or a list of language codes. Only books written in one of these languages will be shown in this lane.

Example language codes can be found below. If this is not specified, the parent lane's value is inherited.

| language | code |
| --- | --- |
| Arabic | ara |
| Bengali | ben |
| Chinese | chi |
| English | eng |
| French | fre |
| Hindi | hin |
| German | ger |
| Italian | ita |
| Japanese | jpn |
| Korean | kor |
| Portuguese | por |
| Punjabi | pan |
| Russian | rus |
| Spanish | spa |

More 3-character language codes can be found in the first column of [this document](http://www.loc.gov/standards/iso639-2/ISO-639-2_utf-8.txt).

## `age_range`

For lanes containing only children's books. A list of ages. Only books classified for one of those ages will be shown.

```
{
 "age_range": [0,1,2,3]
}
```

## `fiction`

If this is set to True, only fiction books will appear in this lane. If set to False, only nonfiction books will appear. If set to the string "both", both fiction and nonfiction will appear in this lane. If not set, or set to the string "default", this lane will inherit the behavior of its parent.

## `genres`

The lane will be restricted to books from these genres. If this is not provided, genre is not considered when selecting books.

## `subgenre_behavior`

If this is set to "separate" (or if it is not set), then each of the subgenres of `genres` will get its own sublane. If this is set to "collapse", then all the books that would have been in sublanes will be dumped into the primary lane.

This example defines a lane called "Science Fiction" which includes sublanes like "Dystopian SF", "Space Opera", and so on:

```
{
 { "name" : "Science Fiction", "genres" : ["Science Fiction"], "subgenre_behavior": "separate"}
}
```

This example defines a single "Science Fiction" lane in which books classified under "Dystopian SF", "Space Opera", and "Science Fiction" are all lumped together:

```
{
 { "name" : "Science Fiction", "genres" : ["Science Fiction"], "subgenre_behavior": "collapse"}
}
```

## `exclude_genres`

A list of genres to explicitly exclude from this lane. For instance, if you want "Dystopian SF" to be on the same level as "Science Fiction" instead of in a sublane, you could do this: 

```
{
 "lanes": [
  { "name" : "Dystopian", "genres" : ["Dystopian SF"] }
  { "name" : "Science Fiction", "genres" : ["Science Fiction"], "exclude_genres": ["Dystopian SF"] }
 ]
}
```

## `sublanes`

Instead of letting sublanes being implicitly defined by genre structure (with "Dystopian SF" being a sublane of "Science Fiction") you can explicitly define a lane's sublanes. The value of `sublanes` is the same as the value of `lanes`: A list of JSON objects, each defining a lane. Those lanes can themselves have `sublanes`, and so on, recursively.
