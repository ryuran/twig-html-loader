# twig-html-loader

A loader for webpack that allows compiling [twig](https://twig.symfony.com/) templates into html string.

## Getting Started

To begin, you'll need to install `twig-html-loader`:

```console
npm install twig-html-loader --save-dev
```

Add twig template to your sources:

```js
var html = require('./index.twig');
```

Then add the loader to your `webpack` config. For example:

```js
module.exports = {
  // ...
  rules: [
    // ...
    {
      test: /\.twig$/,
      use: [
        'raw-loader',
        {
          loader: 'twig-html-loader',
          options: {
            data: {}
          }
        }
      ]
    }
    // ...
  ]
};
```


## Using with html-webpack-plugin

Install plugin:

```console
npm install html-webpack-plugin --save-dev
```

Improve your config:

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
// ...
module.exports = {
  // ...
  rules: [
    // ...
    {
      test: /\.twig$/,
      use: [
        'raw-loader',
        'twig-html-loader'
      ]
    }
    // ...
  ],
  plugins: [
    // ...
    new HtmlWebpackPlugin({
      template: 'index.twig'
    })
  ]
};
```

## Options

|Name|Type|Default|Description|
|--|--|-----|----------|
|cache|boolean|false|Enables the Twigjs cache|
|data|object|{}|The data that is exposed in the templates|
|debug|boolean|false|Enables debug info logging|
|trace|boolean|false|Enables tracing info logging|
|functions|object|undefined|Extends Twig with custom functions
|filters|object|undefined|Extends Twig with custom filters
|tests|object|undefined|Extends Twig with custom tests
|extend|function(Twig)|undefined|Extends Twig with custom tags and more

## Custom functions, filters, tests and tags

You can use `functions`, `filters`, `tests` and `extend` options to extend Twig. See [here for adding custom functions, filters and tests](https://github.com/twigjs/twig.js/wiki/Extending-twig.js), and [here for adding custom tags](https://github.com/twigjs/twig.js/wiki/Extending-twig.js-With-Custom-Tags).

```js
module.exports = {
  // ...
  rules: [
    // ...
    {
      test: /\.twig$/,
      use: [
        'raw-loader',
        {
          loader: 'twig-html-loader',
          options: {
            functions: {
              repeat(value, times) {
                return new Array(times + 1).join(value);
              }
            },
            filters: {
              backwords(value) {
                return value.split(' ').reverse().join(' ');
              }
            },
            tests: {
              theAnswer(value) {
                return value === 42;
              }
            },
            extend(Twig) {
              Twig.exports.extendTag({
                type: 'echo',
                regex: /^echo\s+(.+)$/,
                next: [],
                open: true,
                compile: function (token) {
                  var expression = token.match[1];

                  token.stack = Twig.expression.compile.apply(this, [{
                    type: Twig.expression.type.expression,
                    value: expression
                  }]).stack;

                  delete token.match;
                  return token;
                },
                parse: function (token, context, chain) {
                  return {
                    chain: false,
                    output: Twig.expression.parse.apply(this, [token.stack, context])
                  };
                }
              });
            }
          }
        }
      ]
    }
    // ...
  ]
};
```

```
{{ repeat('_.', 10) }}
{# output: _._._._._._._._._._. #}

{{ 'a quick brown fox'|backwords }}
{# output: fox brown quick a #}

{% if 42 is theAnswer %}42{% endif %}
{# output: 42 #}

{% echo 'hello world' %}
{# output: hello world #}
```

## Alternatives

[twig-loader](https://github.com/zimmo-be/twig-loader)

## License

MIT (http://www.opensource.org/licenses/mit-license.php)
