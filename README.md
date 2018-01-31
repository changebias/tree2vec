# ast2vec

### Description

Transformation AST of code in some programming language to a vector.
The vector is constructed using feature extraction from AST.

Program consist the following feature extractors:
- **DepthExtractor** - min, max or mean depth extraction from AST;
- **CharsLengthExtractor** - min, max or mean chars length (for some node) from AST;
- **NGramsExtractor** - calculating number of specified n-grams.
- **AllNGramsExtractor** - calculating number of all n-grams by specified configuration (n, max_distance, etc). See [ast-ngram-extractor](https://github.com/PetukhovVictor/ast-ngram-extractor) (for only n-grams extraction)

**This program is used as part of [ast-set2matrix](https://github.com/PetukhovVictor/ast-set2matrix)**


### Example of use

```
python3 main.py -i ./ast/my_code_as_ast.json -o ./ast_vectors/my_code_as_vector.json --no_normalize
```

### Program arguments

- **--input (-i)**: file with AST
- **--output (-o)**: Output file, which will contain features and feature values as JSON
- **--is_normalize (-d)**: normalization necessary of vectors on the maximum value

### AST format

The program is required on input the AST of the following format (example input):
```
[
   {
      "type":"FUN",
      "chars":"override fun onCreateView(inflater: LayoutInflater?, container: ViewGroup?, savedInstanceState: Bundle?): View? {\n        dialog.window.requestFeature(Window.FEATURE_NO_TITLE)\n\n        DaggerAppComponent.builder()\n                .appModule(AppModule(context))\n                .mainModule((activity.application as MyApplication).mainModule)\n                .build().inject(this)\n\n        var view = inflater?.inflate(R.layout.dialog_signup, container, false)\n\n        ButterKnife.bind(this, view!!)\n\n        return view\n    }",
      "children":[
         {
            "type":"MODIFIER_LIST",
            "chars":"override",
            "children":[
               {
                  "type":"override",
                  "chars":"override"
               }
            ]
         },
         {
            "type":"IDENTIFIER",
            "chars":"onCreateView"
         },
         {
            "type":"VALUE_PARAMETER_LIST",
            "chars":"(inflater: LayoutInflater?, container: ViewGroup?, savedInstanceState: Bundle?)",
            "children":[
               {
                  "type":"LPAR",
                  "chars":"("
               },
               {
                  "type":"VALUE_PARAMETER",
                  "chars":"inflater: LayoutInflater?",
                  "children":[
                     {
                        "type":"IDENTIFIER",
                        "chars":"inflater"
                     }
                  ]
               }
            ]
         }
      ]
   }
]
```
It is Kotlin AST, generated by [**Kotlin custom compiler**](https://github.com/PetukhovVictor/kotlin-academic/tree/vp/ast_printing_text)

Also reqired AST transformer, which is a part of [**github-kotlin-code-collector**](https://github.com/PetukhovVictor/github-kotlin-code-collector) (see `src/lib/helper/AstHelper.py`)

File with JSON representation of AST must be passed as an argument of program.

For example: `python main.py ast/my_program.json`

### Vector format

Program output is map with name and value features.

Feature values is vector components.

For example:
```
{
  'chars_length_avg': 47.297029702970299,
  'chars_length_max': 2047,
  'depth': 16,
  'depth_avg': 6.3469387755102042,
  'CALL_EXPRESSION': 0.06373937677053824,
  'DOT_QUALIFIED_EXPRESSION:REFERENCE_EXPRESSION:IDENTIFIER': 0.028368794326241134,
  'DOT_QUALIFIED_EXPRESSION:DOT_QUALIFIED_EXPRESSION': 0.005689900426742532
}
```

### Feature configuration

Features are specified in main.py (keys array for simple features; and objects array for n-grams and other (in the future)).

Example with specified n-grams:
```
simple_features = [
    'depth',
    'depth_avg',
    'chars_length_avg',
    'chars_length_max'
]

features = [
    {
        'type': 'ngram',
        'params': {
            'name': 'CALL_EXPRESSION',
            'node_types': ['CALL_EXPRESSION']
        }
    },
    {
        'type': 'ngram',
        'params': {
            'name': 'DOT_QUALIFIED_EXPRESSION:REFERENCE_EXPRESSION:IDENTIFIER',
            'node_types': ['DOT_QUALIFIED_EXPRESSION', 'REFERENCE_EXPRESSION', 'IDENTIFIER'],
            'max_distance': 3
        }
    },
    {
        'type': 'ngram',
        'params': {
            'name': 'DOT_QUALIFIED_EXPRESSION:DOT_QUALIFIED_EXPRESSION',
            'node_types': ['DOT_QUALIFIED_EXPRESSION', 'DOT_QUALIFIED_EXPRESSION'],
            'max_distance': 1
        }
    }
]
```
`node_types` - type of nodes, which should be on the one path in AST (according to specified distance).

`name` - name of feature, it used in output (feature names).

Example with all n-grams with specified configuration:

```
simple_features = [
    'depth',
    'depth_avg',
    'chars_length_avg',
    'chars_length_max'
]

features = [
    {
       'type': 'all_ngrams',
       'params': {
           'n': 3,
           'max_distance': 3,
           'no_normalize': True,
           'include': [['CALL_EXPRESSION', 'LPAR'], ['VALUE_ARGUMENT_LIST'], ['SAFE_ACCESS_EXPRESSION']],
           'exclude': [['FUN']]
       }
   }
]
```

#### All n-grams extractor arguments

* **n**: max n in n-gram;
* **max_distance**: max distance between neighboring nodes (window);
* **no_normalize**: flag to normalize values (n-grams number);
* **include**: array of arrays with sub-n-gram witch should be **contained** in the found n-grams;
* **include_strict**: required n-grams (the remaining n-grams found will be removed);
* **exclude**: array of arrays with sub-n-gram witch should be **not contained** in the found n-grams;
* **exclude_strict**: n-grams, which should be excluded.
